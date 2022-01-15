# How-To-Crypto-Mine-for-Monero-XMR-on-Ubuntu-20.04-with-XMRig

## System Setup
For this setup we can use a desktop or laptop and install `Ubuntu 20.04 LTS`. We go over the full setup assuming a fresh install of the OS is ready for us.

If you have not installed the OS yet, feel free to choose the "minimal" install since that is all we need.  However, feel free to install whatever you like.

*Note: Optionally, consider using a dedicated `Rasperry Pi` (or similar) to run a hardware firewall such as the `openwrt` router.  We will use the `ubuntu` firewall known as `ufw`, but we also want a hardware firewall as well.  For a full guide on setting that up, see my write-up at `https://github.com/mikenizo808/How-To-Install-openwrt-on-Raspberry-Pi`*

## Credit / Motivation
Thanks to `Network Chuck` for the motivation and and literally the commands.  We essentially follow his video for setting up Raspberry Pi, except that we use Ubuntu, and (spoiler) the commands are exactly the same.

Check out Chuck's great video on the topic of `XMRig` on `Raspberry Pi` at:

    https://youtu.be/hHtGN_JzoP8

## List Video Card

    lspci | grep VGA

## Example Output

    #nvidia system
    mike@ubuntu02:~$ lspci | grep VGA
    01:00.0 VGA compatible controller: NVIDIA Corporation TU116 [GeForce GTX 1660 Ti] (rev a1)

    #amd system
    mike@ubuntu004:~$ lspci | grep VGA
    0a:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Device 73ff (rev c7)


## `Nouveau` Driver
By default, modern Nvidia cards should work okay with the native Ubuntu driver known as `Nouveau`. This runs automatically and requires no interaction. However, depending on the os release you are running and other factors, your results may vary (i.e. some crashes or lots of crashes).

## The `Nvidia` Driver
If your system uses an `AMD` card then no drivers are needed.  However, if using an `Nvidia` card, you can optionally switch from the default `nouveau` driver to the `nvidia` driver.

Simply launch the `Software & Updates` application and navigate to the `Additional Drivers` tab. There you will see the various versions available; Click the one at the top of the list for the latest.

## Optional Networking - Install `net-tools` with `apt`
For conveneince, we can install `net-tools` which will let us do `ifconfig` to see our network info if ever needed (or just use the default `ip addr` command, though the output is not great for that one.).  For these reasons and more, we install `net-tools`.

    sudo apt install net-tools

## About `ssh` on your Desktop and Miner Node
Let's discuss who needs software installed to be able to reach them from terminal using `ssh`.

First, let's review the software involved:

- `openssh-server` - This allows things to `ssh` to us.  This requires installation if desired.

- `openssh-client` - This let's you `ssh` to things and is installed by default.

For my desktop, I do not need things to `ssh` to me (though I could allow it for the convenince of being able to `scp` files to myself). So, nothing needs to reach me; However, I do want to reach other things (i.e. terminal using `ssh` to my Miner nodes).

As such, the other nodes will need to be running the "Server" component of ssh. Here, we install `openssh-server` and configure the Ubuntu `ufw` firewall.

    #install
    sudo apt install openssh-server

    #Optional - check service status
    sudo systemctl status sshd

    #Optional - Show apps that Ubuntu firewall (a.k.a `ufw`) has defined by name.
    sudo ufw app list

    #Choose the network to allow through firewall
    sudo ufw allow from 10.100.0.0/16 to any app OpenSSH

    #enable firewall
    #sudo ufw enable

    #restart ssh
    sudo systemctl restart sshd

*Note: There are many ways to allow `ssh` access, this is just one example using my network. For more details and general examples, see `https://ubuntu.com/server/docs/security-firewall`. If you want to reset the firewall to defaults you can run `sudo ufw reset`.*


## Installing `XMRig` Open Source Mining Software
Spoiler: every single command is exactly the same for `Ubuntu` or `Raspberry Pi`, so feel free to run `XMRig` on both. 

    ## install requirements
    sudo apt install -y git build-essential cmake libuv1-dev libssl-dev libhwloc-dev

    #change directory
    cd ~/Downloads

    #clone repo
    git clone https://github.com/xmrig/xmrig.git
    
    #change directory
    cd xmrig

    #make directory
    mkdir build

    #change directory
    cd build

    #cmake (don't forget the .. which says the bits are up one directory)
    cmake ..

    #make (choose how many procs to compile on)
    make -j4

*Note: Using the "-j" parameter with make speeds up the compile time with the integer being how many procs you have or want to compile on. On my desktop with 16 procs, I compile using -j14 so I keep two procs for the system. Remember that this only speeds up the compile time, and the resulting binary will be the same.*

## Optional - Copy `xmrig` to `/opt` 
You can run `xmrig` immediately by typing `./xmrig` from the build directory.  Or, we can copy the bits someplace official.

    #create directory
    sudo mkdir /opt/xmrig

    #copy bits
    sudo cp ./xmrig /opt/xmrig/xmrig

    #test
    /opt/xmrig/xmrig --help

## Choose a "pool"
You can mine on your own but most people join a mining pool such as `moneroocean`. To join a pool we just point to `gulf.moneroocean.stream:10128` (or other pool you may choose); There is no sign-up or login.

You can learn more (and also watch your hashrate in realtime) from a web browser to: 

    https://moneroocean.stream/


## Run `xmrig`
This is an example using a mining pool from `moneroocean`, and also my "Wallet ID".

*Note: If you do not have a "Wallet ID" yet not worry as we get to that later.*

    #syntax
    </path/to/xmrig> -o <mining pool address> -u <your monero wallet address> -p <optional label for worker> 

    #example
    /opt/xmrig/xmrig -o gulf.moneroocean.stream:10128 -u someWalletAddress -p someDescription

    #example with my wallet
    /opt/xmrig/xmrig -o \
    gulf.moneroocean.stream:10128 \
    -u 48VWWKR6kEzByCxSZSTtJYMUgZzM83eTfMAZKbkirmktDMn7XcXaQxsRxja1GACmSaVmHxi5GDg9HGDtRTtSjnm36GZ1CqU \
    -p ubuntu4

*Note: Where `-p` is a friendly name for your instance; This can be the same as the hostname of your mining device, or whatever helps you remember which nodes are which.*

## Keyboard Shortcuts for `xmrig`
When `xmrig` is running in terminal, there are a several command shortcuts such as the following:

 `h` - Hash rate
 `s` - Shares
 `c` - Connection (shows your connection quality etc.)

## Setup Monero Wallet
From your desktop or some other computer we will setup the Monero Wallet (an application).  This will make your unique id so you can get paid from XMR.

Follow the steps to download and verify your binaries at:

    https://www.getmonero.org/resources/user-guides/verification-allos-advanced.html


*Important: For either GUI or CLI downloads, be sure to use at least version `0.17.2.3`, since there were major bugfixes in this release.  Here, we use use the very latest `0.17.3.1`.*

## Installing `Monero Wallet`
You can just download and launch the app, but the recommendation is to perform the following technique to verify the bits.

    #Optional - show working directory
    pwd

    #Optional - change to desired directory such as home or downloads.
    cd ~/Downloads

    #get the gpg key
    wget -O binaryfate.asc https://raw.githubusercontent.com/monero-project/monero/master/utils/gpg_keys/binaryfate.asc

    #show fingerprint
    gpg --keyid-format long --with-fingerprint binaryfate.asc

    #If fingerprint above matches, proceed by importing the gpg key
    gpg --import binaryfate.asc

    #get hash file
    wget -O hashes.txt https://www.getmonero.org/downloads/hashes.txt

    #verify hash
    gpg --verify hashes.txt

    #download monero binary (unless you already have it downloaded already)
    wget -O monero-gui-linux-x64-v0.17.3.1.tar.bz2 https://downloads.getmonero.org/cli/linux64

    #verify the binary
    shasum -a 256 monero-gui-linux-x64-v0.17.3.1.tar.bz2

*Note: The checksum output from the last command above should return the same info as provided by the `hashes.txt` file.*

## Checking File Validity manually
The sha256 hash for each of the current releases is available on the downloads page. The following is the hash for the Linux 64 GUI version `v0.17.3.1`.

    02e8e32455383cf32030e33511656492a352788a619a0c9220ec360c2e863ef9

## Using `grep` to check the `hashes.txt`
We can use `grep` to search for our hash, so get that on the clipboard and paste it in, similar to the following.

    grep 02e8e32455383cf32030e33511656492a352788a619a0c9220ec360c2e863ef9 ./hashes.txt

## Example Output

    mike@ubuntu03:~/Downloads$ grep 02e8e32455383cf32030e33511656492a352788a619a0c9220ec360c2e863ef9 ./hashes.txt 
    02e8e32455383cf32030e33511656492a352788a619a0c9220ec360c2e863ef9  monero-gui-linux-x64-v0.17.3.1.tar.bz2

## View the Guide for Monero GUI

    https://github.com/monero-ecosystem/monero-GUI-guide/releases


## Optional - Install the `Monero CLI`
Not needed, but you can optionally install the `Monero CLI`. This can be skipped if using the `Monero GUI` version.

To download the `Monero CLI` (if desired) navigate to the same downloads page as the GUI version (just scroll to bottom to find it):

*Reminder: For GUI or CLI downloads, be sure to use `v0.17.2.3` or greater, since there were major bugfixes in this release. The latest release is `v0.17.3.1` at the time of this writing.* 

## Launch the Monero GUI
You can just launch it from your downloads or home drive wherever you placed it.  However, you might want to move the bits to a standard location such as `opt`.

    #make a directory
    sudo mkdir /opt/monero

    #move the bits (adjust this path for your setup!)
    sudo mv "/path/to/monero-gui-linux-x64-v0.17.3.1" /opt/monero/
    
    #change directory
    cd /opt/monero

    #open a file browser in current directory
    xdg-open .

    #To launch, double-click the file that says "AppImage"
    monero-wallet-gui.AppImage

## Start Mining
The following is an overview of the steps required to start mining now.. 

    ## Secure your network
        - Run a hardware firewall
        - Disable bluetooth and WIFI
        - Connect to network via ethernet 

    ## From your main system, if you have not already:

        - Participate in the secure network
        - This means turning off your bluetooth and WIFI
        - Consume network from your hardware router using ethernet
        - Launch Monero GUI
        - Use the guided setup / quick-start to generate a user key (this identifies your wallet).
        - Click "Account" in the left pane
        - In the right pane, click the Clipboard icon to "Copy address to clipboard".
    
    ## From an ssh terminal to your mining device

        #change directory to where you placed the bits
        cd /opt/xmrig/
        
        #start the XMRig miner (adjust as needed to point to the desired pool, etc)
        ./xmrig -o gulf.moneroocean.stream:10128 -u <your monero key> -p <friendly name>

*Note: Where `-u` is your Monero "key" you copied to your computer's clipboard earlier (using Monero GUI).*

*Note: The `-p` parameter allows you to name this particular runtime (useful if running more than one device). This information then shows up on the dashboard at `https://moneroocean.stream`*

## Summary
In this guide, we got you up and running `XMRig` open source mining software on `Ubuntu 20.04`. Credit and props to Network Chuck for the motivation and literally the commands!

Also worth mentioning again is that setting up `XMRig` on `Raspberry Pi` uses exactly all of the same commands as `Ubuntu` (so add some `Pi` nodes!).

As for hashrate, my `Raspberry Pi 4` nodes solve at about `900` hashes/sec (depending on memory) and my `AMD` desktop card gets about `4000/sec`.

Finally, be aware it may take a couple of days before you see any balance above complete zeros, since the mining pools usually have some limit before your first payout show up.

*PS - Remember that with the cost of electricity, mining should really only be done to break even and have fun!*
