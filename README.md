# How to Mine on Linux (Debian 11) with Nvidia GPUs. A Step-by-step Guide.

Here is a step-by-step guide on setting up and overclocking GPUs on Debian Linux with useful links and tips. I spent around a week searching for a stable solution, and below is what works for me.


## Pre-requisits

1. First, you have to install Debian with the graphical desktop manager (I prefer GNOME). The reason for that is simple: to overclock Nvidia GPUs, you need an X-Server running.

2. If you have newly installed Debian 11, you need to add the user to the sudo group to run commands with admin privileges. To do that, run
```
su
/usr/bin/adduser <username> sudo
```

3. If you want to mine with multiple GPUs, it is recommended to switch PCIE[slot-number] speed from "Auto" to "Gen2" in your BIOS to detect all the GPUs correctly. For instance, in ASRock motherboard, it is done in the "Advanced" -> "Chipset config" section. 

I also set the primary monitor to PCIE[slot-number], where the physical monitor is attached to GPU on the rig.

**Tip.** By default Debian with graphical desktop goes to suspend mode after several minutes of inactivity. To prevent it, go to GNOME settings, and choose 
- Keep display switched on in "Power Management" settings. 
- Enable autologin for the user under the "Users" setting to avoid suspending the login screen and running startup scripts after GNOME is loaded. 


## Nvidia Drivers Installation

Here is a [beatiful article](https://www.linuxcapable.com/how-to-install-or-upgrade-nvidia-drivers-on-debian-11-bullseye/) describing the installation process. Please avoid installing the default driver included in apt package manager. Instead, choose the manual installation and with the latest driver available. 

I have tested both and noticed that the latest driver gives a better hash rate for LHR cards. 

P.S. If you are not familiar with **LHR** concept, please have a look at [this article](https://gamingdope.com/how-to-unlock-and-mine-on-nvidia-lhr-v2-rtx-30-series-gpus/)

**Tip.** For the Nvidia drivers to start and overclock correctly, you need a real monitor to be switched on and connected to one of the GPUs on the rig. You need it at least for the first time while you are tuning your GPUs.

## The Nvidia X-Server configuration

**Concept**. X Server is a daemon that allows rendering graphics. Nvidia X Server is a set of additional configurations to X Server which allow Nvidia GPU to render graphics/video.

By default, Nvidia protects its GPUs from accidental overlocking. For instance, if you call "nvidia-settings" GUI utility, you will notice no options to change clock offsets.

Fortunately, you may add a few lines to the X-Server config file to fix that. An even better solution is to use "nvidia-xsettings" command, which adds necessary lines to the config file for you.

Run:
```
sudo nvidia-xconfig --enable-all-gpus --cool-bits=12
```

It will update/create /etc/X11/xorg.conf file. You may notice that the new lines with the text "Option" "Coolbits 12" were added to the config file. This directive says "nvidia-settings" utility to allow overclocking options.

Now, if you run `nvidia-settings` in your rig's terminal, you will see the Nvidia Setting GUI window, which has a new feature: Editable performance levels (in PowerMizer section). Ok, now you can overclock.

## Miner installation

Before proceeding with overclocking, it is a good idea to install the miner. I prefer to test my overlock setting in the actual mining process to estimate the effectiveness of my tuning.

My personal preference is [NBMiner](https://github.com/NebuTech/NBMiner)

When writing this article, NBMiner allows unlocking LHRv2 Nvidia cards up to 74% of their typical performance. Again, if you are not familiar with **LHR** concept, please read [this article](https://gamingdope.com/how-to-unlock-and-mine-on-nvidia-lhr-v2-rtx-30-series-gpus/)

With the NBMiner installed, you can run the default script to test how your GPUs perform. Later on, you will adjust the miner script once you are happy with your overclock settings. 

For instance, to test your performance on the epherium network, go to the miner's directory and run `./start_eth.sh`. You will see the current hash rate, power consumption, and GPU's temperature. These are the most important KPIs to control.

```
[18:39:26] INFO - ===================== [nbminer v40.1] Summary 2021-12-01 18:39:26 =====================
[18:39:26] INFO - |ID|Device|Hashrate| LHR|Accept|Reject|Inv|Powr|CTmp|MTmp|Fan|CClk|GMClk|MUtl|Eff/Watt|
[18:39:26] INFO - | 0|3060ti| 42.19 M|  74|     6|     0|  0| 112|  58|    | 61|1350| 7525| 100| 376.7 K|
[18:39:26] INFO - |------------------+----+------+------+---+----+--------------------------------------|
[18:39:26] INFO - |    Total: 42.19 M|    |     6|     0|  0| 112| Uptime:  0D 00:14:40        CPU:  0% |
[18:39:26] INFO - =======================================================================================

```

Here is the miner's output:
- ID0 means GPU 0
- LHR 74 means the miner unlocked LHR GPU up to 74% of its performance. 
- CTmp 58 - GPU temperature 58 degrees
- Fan 61 - GPU fan gets 61% of its maximum speed
- GMClk 7525 - Memory Clock is 7525 MHz (factory's default is 7100 according to `nvidia-smi -q -d CLOCK | less`. That is an easy way to check if your GPU is overclocked)
- GClk 1351 - GPU Clock is is 1351 MHz
- MUtl 100 - GPU Memory gets 100% of its performance.




## Nvidia-setting and overclocking

You can play with GPU's memory frequencies within factory limits with `nvidia-smi` utility. But if you want to go beyond the limits, use `nvidia-settings` GUI utility.

When nvidia-settings starts, it reads the current settings from its configuration file (typically ~/.nvidia-settings-rc) and sends those settings to the X server.  Then, it displays a graphical user interface  (GUI) for configuring the current settings.  When nvidia-settings exits, it queries the current settings from the X server and saves them to the configuration file.

## Nvidia-settings over ssh

As you see, nvidia-setting utility runs as a GUI application, even if you run it as a command-line directive with some parameters.

It means that you need to run it locally in front of the display attached to the rig. 

If you prefer to use ssh connection and run graphical utility remotely, you need:

1. Install X-Server on your local machine if you are on Mac/Windows. It allows you to run GUI applications remotely. To check if it works, ssh to your rig (`ssh -X <your_rig_username>192.168.xx.xx`), type `firefox`, and watch X window opening. If you are on Linux, you have X server installed by default.
2. Check your current display in ssh session by typing `echo $DISPLAY` in ssh terminal. Typically display has number 10, which means you are using 6010 port to run X Window apps via ssh (Do not forget to use -X parameter `ssh -X ...` to enable X-session over ssh).
3. Now, instead of running `nvidia-settings` locally on the rig, run `nvidia-settings --display=:10 --ctrl-display=:0` over ssh. Here 
    - "10" means the monitor where the X Window is shown (your local computer which initiates ssh connection to the rig). 
    - "0" means the physical monitor attached to the rig. In my case, I have the rig's monitor connected to GPU0, so I use 0.

## Overclocking

Now the fun begins. 

### From the command line

To overclock your memory frequency, send a command to nvidia utility:
```
nvidia-settings --display=:10 --ctrl-display=:0 -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=1200'
```
The result applies to Nvidia X Server imidiately. Here
* `--display=:10 --ctrl-display=:0` says nvidia utility to run via ssh (see section above)
* `-a` - apply immidiately
* `'[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=1200'` - shift base memory frequency by 1200 MHz up.

You can see the result of this command by querying Nvidia X Server back:
```
nvidia-settings --display=:10 --ctrl-display=:0 -q '[gpu:0]/GPUMemoryTransferRateOffset[3]'
``` 

**Interesting fact** The command line directives are not well documented and may change with time. It is a good idea to ask for the right/actual command on [Nvidia Developers Forum](https://forums.developer.nvidia.com/t/solved-nvidia-settings-a-gpu-1-gpumemorytransferrateoffset-3-2000-doesnt-take-effect/178021/2)

### From GUI

It is much easier to call Nvidia-utility `nvidia-settings --display=:10 --ctrl-display=:0` and to change memory clock offset from there (GPU#XX -> PowerMizer). But in my case, the GUI utility didn't perform as well as the command line. So I have opened an [issue](https://github.com/NVIDIA/nvidia-settings/issues/77) after discussion with Nvidia technicians.

### Caution

Please do apply offset by small steps. For instance, for Nvidia GeForce 3060 Ti LHR v2, the maximum possible offset is 6000 MHz (see GPU#XX -> PowerMizer in nvidia-settings GUI). Hence the reasonable incremental step is around 5-10% of 6000 MHz, which is 300-600 MHz per test. 

After you change the memory clock offset a little, run the miner for a short time to see the current hash rate and GPU's temperature.

You'll see that GPU fails to run at some point in time - the miner halts or gives an error. In my case, it happens at a 1900 MHz offset. I want to be sure my GPU runs stable for the long term, so I roll back to 20% of this threshold, which gives me comfortable 1400-1500 MHz offset. It increases my LHR GPU's hash rate from 37,5 to 42,2 Mhs (+12%). Not so much, but this is stable for long-term use.

### Undervoltage by GPU frequency

While memory clock offset directly affects hash rate performance, the GPU offset mainly affects power consumption, I far as I noticed.

The `sudo nvidia-smi` command may lock the GPU frequencies in the narrow corridor. For instance, the commands below set the persistent mode and lock GPU#0 frequency in 1350 to 1350 MHz.
```
nvidia-smi -pm 1
nvidia-smi --id=0 --lock-gpu-clocks=1350,1350
```
If I run the miner again, I notice that the power consumption decreased from 200W to 115 +/- 10 W. The GPU temperature dropped to 58-60 degrees Celsium.

This 1350 MHz is less than base 1950 MHz by 600Mhz. You can see the current and base clocks by running `nvidia-smi -q -d CLOCK` command.

As far as I notice, when you decrease GPU clock, you decrease power consumption. So the aim is to guess what GPU locked frequency:

1. Best increases power consumption, and at the same time
2. Less affects your current hash rate.

Of course, this is a subject of experimental tuning. In my case (Asus GeForce 3060 Ti Mini LHR), a locked GPU frequency of 1350 MHz and 1450 memory overclock works well.

## Undervoltage by Power Limit

Alternatively, you may decrease consumption by settings a maximum power limit for your GPU with the same "nvidia-smi command". For instance, the commands below set persistent mode and limit power consumption for GPU#0 by 120W.

```
nvidia-smi -pm 1
sudo nvidia-smi --id=0 -pl 120
```

## Loading your configuration on boot

Once you are happy with the results, it is time to apply your configuration on load.

### Power management config on load

The power consumption config is set up with nvidia-smi utility, which requires root privileges and doesn't require X Server running. Hence the simplest way to apply this config is to write these commands in root's crontab by typing `sudo crontob -e` and adding your config like this:

```
@reboot nvidia-smi -pm 1
@reboot nvidia-smi --id=0 --lock-gpu-clocks=1350,1350
```

### Overclock settings on load

The overclock settings are a bit trickier since they need to be auto-load after Nvidia X server starts, i.e., after GNOME (or another desktop environment) is loaded.

Not elegant, but the working solution is to create a startup application that GNOME runs on user login. Remember, we set a user autologin option in GNOME so that we can use this method.

To make a startup application in GNOME, you need to create a file in ~/.config/autostart/ with .desktop extension with following content:
```
[Desktop Entry]
Name=Nvidia Setup
Encodinf=UTF-8
Type=Application
Exec=sh -c "/home/aborealis/start_miner_overclock.sh"
Hidden=false
X-GNOME-Autostart-enabled=true
Terminal=false
Comment=NVIDIA GPU Settings
```

Change "/home/aborealis/start_miner_overclock.sh" to your actial autoload script. 

In your script, write the command you want to send to X Server on load. In my case, this is:
```
#!/bin/bash
nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=1450'
screen -d -m -S miner
sleep 5
screen -S miner -X -p 0 stuff "/home/aborealis/miners/NBMiner_Linux/start_eth.sh^M"
```

Explanation:

1. First, I run the command to set up a 0th GPU memory offset to +1450 MHz,
2. Then, I run the miner in a new screen session. It allows me to log in to the rig via ssh and switch to the running miner to see how it goes by typing `screen -r`. More about screen utility is [here](https://linuxize.com/post/how-to-use-linux-screen/)
- `screen -d -m -S miner` starts a new screen named "miner" in detached mode
- `sleep 5` I deliberately wait for 5 seconds to allow all Nvidia settings to apply (not an elegant solution, I know)
- `screen -S miner -X -p 0 stuff "/home/aborealis/miners/NBMiner_Linux/start_eth.sh^M"` - 5 seconds later I send a command to new screen to start a miner. Here is "/home/aborealis/miners/NBMiner_Linux" is where NBMiner lives.

Finally, do not forget to make both
- the startup script and 
- your desktop app 
executable (type `sudo chmod +x <filename>`)

P.S. If you are luckier than me and your nvidia-setting utility reads/writes the ~/.nvidia-confog-rc configuration file as expected, 
1. you may save your overclock configuration setting right into the config file (by adjusting settings in nvidia-config GUI app and then exiting the window).
2. load the settings on startup with one command: `nvidia-settings --load-config-only` instead of sending sets of commands in the script above. 

## Configuring the miner

Now it is time to configure your miner to use your desired pool and wallet. In my case I use NBMiner and it is loaded by "/home/aborealis/miners/NBMiner_Linux/start_eth.sh" script.

```
/home/aborealis/miners/NBMiner_Linux/nbminer --log-cycle 20 -a ethash -o stratum+ssl://eu1.ethermine.org:5555 -u <...my wallet..>.<my rig name> --api 127.0.0.1:22333 -log
```

**Note.** I have added --api 127.0.0.1:22333 option to access the miner statistics via http (I'll explain this later)

In your case, the script will be different, depending on your mining preferences.

## Remote access to the rig behind a router

If your rig is behind a home router/NAT server, you may access it via remote ssh tunneling. [Here is an article](https://linuxize.com/post/how-to-setup-ssh-tunneling/), which explains the principle.

As you see from the article, you need a separate server (www server) with a public I.P. address which you will use as a connect-point to access your rig from the outer world. You may rent such a virtual web host at Digital Ocean, Hetzner, or any other hosting provider. It is really cheap.

The basic steps are following
1. Rent a web host with public IP.
2. Create ssh keys to log in remotely to the web host from the rig without the need to enter a password:
```
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub username@webserver
```

- The first command creates the pair of the public (id_rsa.pub) and private (id_rsa) keys and puts them into ~/.ssh directory
- The second command copies the content of your id_rsa.pub file and writes it to the web host into ~/.ssh/authorized_keys file
3. Create a ~/.ssh/config file on your rig:
```
host my_web_host
  HostName <webhost_public_ip>
  port 22
  IdentityFile ~/.ssh/id_rsa
  User <username>
```
now you can ssh from rig to remote web host by simply typing `ssh my_web_host`

4. Do the same procedure to connect from your local laptop/desktop/mobile terminal app to the webserver.

5. Now, create a reversed ssh tunnel between the rig and web host. From your rig type:
```
ssh -R -N -f 10022:localhost:22 my_web_host
```
This command asks to redirect all connections coming to the web host's localhost:10022 to your rig's 22nd port.

You can easily check how it works. Ssh to your web server and type there:
```
ssh -p 10022 <your_rig_username>@localhost
```
Option -p 10022 means that you connect to the 10022nd port to the localhost on the webserver. It will immediately redirect you back to the rig's terminal. 

To avoid asking for a password each time, create the ssh key on a webserver to connect to the rig as described in steps 2 & 3.
6. You may now connect to the rig remotely from your desktop/laptop/mobile app terminal by typing:
```
ssh -t my_web_host 'ssh -p 10022 <your_rig_username>@localhost'
```

## Load Remote SSH Access on Load

To make this connection resistant to failure and to load it automatically, use autossh utility (`sudo apt install autossh`) and write the following lines in your user's crontab:
```
@reboot /usr/bin/autossh -v -N -o "CheckHostIP=no" -o "ExitOnForwardFailure=yes" -o "ServerAliveInterval=10" -o "PubkeyAuthentication=yes" -o "PasswordAuthentication=no" -i /root/.ssh/id_rsa -R my_web_host:10022:localhost:22 -f my_web_host
```

## LAN Remote web monitoring

As I mentioned before, I have added `--api 127.0.0.1:22333` option to the mining script. So from now on, one can open firefox on the rig and go to http://127.0.0.1:22333 to see real-time web monitoring of NBMiner's work. But at this point, you can only access this web monitoring page from the rig.

To make it visible in the local network, 
1. [Install and enable Nginx server](https://idroot.us/install-nginx-debian-11/)
2. Write a simple config file instead of the default one in /etc/nginx/nginx.conf

```
user www-data;

events {
        worker_connections 768;
}

http {

        server {
          listen 80;
          server_name  localhost;

          location / {
            proxy_pass http://127.0.0.1:22333;
          }
        }
}
```
3. Restart Nginx `sudo systemctl restart nginx`

Now you can access the web-monitoring page from anywhere in your local network. For example, if you go to the `http://192.168.xx.xx` (input your rig's IP address), you will see a web monitoring page.


## WWW Remote web monitoring

Now, if you run the command below, you will create another reversed ssh tunnel.
```
ssh -R _N -f 8486:127.0.0.1:80 my_web_host
```
From now on, all requests to localhost:8486 on the webserver will be redirected to the rig's 80th port. As you remember, Nginx listens to this post and returns an HTML page in response.

Let's check it. Ssh to your web host and type:

```
wget -O- 127.0.0.1:8486
```

You will see the HTML content of the web monitoring page. To load your ssh tunnel on startup, add:

```
@reboot /usr/bin/autossh -o "CheckHostIP=no" -o "ExitOnForwardFailure=yes" -o "ServerAliveInterval=10" -o "PubkeyAuthentication=yes" -o "PasswordAuthentication=no" -i /root/.ssh/id_rsa -R 8486:127.0.0.1:80 -N -f my_web_host
```

to your crontab.

Now you can: 
1. Assign the domain to the webserver (not covered in this guide)
2. Configure the Nginx server on the remote host to redirect all requests to 443 port (https://<yourdomain>) to the localhost:8486. (not covered in this guide)

Once you have done it, you can access web monitoring statistics via the domain you created.
