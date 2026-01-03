# Proxmox
I'll be using Proxmox v9.1.2 to develop my home server with installations such as JellyFin, Media automation, File Storage, and many more!! Proxmox is mainly used as my virtual environment creation to host multiple lightweight LXC &amp; VMs for my  testing purposes. Documenting all of this for my jorney.

# Hardware
Notes: I just bought an Optiplex and used leftover HDD that I had laying around. I won't be using a backup in this set up but if I would, I'd likely add a 2T HDD in the later future. Though it's entirely dependent on the power supply of the Optiplex that I purchased, it's still a really good chance that this could be supported. If the PSU doesn't support it, I may have to find an alternative to either 1. Replace the current PSU or 2. Utilize another PSU and provide the power there or 3. Find a reason to rebuild a computer because its cool but with $$$ as a downside... You could always spend more ;)

- Intel i5 8500 (6 Core) @3.00GHz
- 16 GB 2666mhz Memory
- 2x1T HDD
- 1x256 NVMe SSD

What you need:
- 8 GB or more empty USB
- A working computer???

# Initial Proxmox Setup
I downloaded Proxmox (https://www.proxmox.com/en/downloads) and retrieved an ISO file, where I would use an application called Rufus (https://rufus.ie/en/#download) to format the ISO file and create a bootable USB flash drive. I just found an 8 GB USB that I had laying around and flashed it from there.

> [!WARNING]
> ** Make sure that your USB flash drive is not mounted and does not contain any important data, or else it will be wiped!**

To re-format the USB along with the Proxmox ISO file, under "Device", select the USB drive you want to set as a bootable device. Under "Boot Selection", we want to select "Disk or ISO image", and then select on the downloaded ISO file. I've left the rest of the forms as default as it will propogate themselves and started the process, which took no more than 3 mins. Once completed, I can finally use this USB to install Proxmox!

<img width="478" height="585" alt="image" src="https://github.com/user-attachments/assets/e3e3922b-dfb1-4e42-ae2e-be3db977f8af" />

> [!NOTE]
> For the install you'd essentially want to select the option to boot up the USB flash drive over the current bootable set up, which im not going to go over.

Once I got Proxmox booted up, I followed the instructions on each page. One thing I kept in mind was where I wanted the OS to be installed (which I obviously selected the NVMe storage I had for faster booting). As you follow the instructions, be aware of your current interal networking set up. You want to note what your current private IP you are using. For me, I configured my private network using a 10.0.0.0/24 subnet. At the end of this, you should be able to install Proxmox on the selected storage space and remove the USB flash drive. You should be able to log in with the link via your web browser from any device in your internal private network.

When I got into my local Proxmox web GUI, the first thing I did was remove any subscription based and disabled and enterprise or payment features that it supported. I simply went to "Repositories" section and disabled any of the following repositories I didnt need.

From there, I was able to start off and create my very own LXC! I learned an LXC is a lightweight linux container that can isoloate apps, which can improve efficeincy compared to an actual VM. A VM (virtual machine) is a literal virtual computer that acts as a separate computer, where there can be multiple choices of OSes to choose from. In this case, I created an LXC purely for fileshare management and a VM to host all my media - which would include docker instances that will run in the background for automation.

# LXC Set up

Firstly, we create an LXC... This LXC will be using the Ubuntu 24.04 CT template. I've made it extremely light weight with 1 core and 1Gb of memory since this is only meant for fileshare purposes and used my NVMe SSD as the boot disk (it was only 32Gb). For the network portion, I chose dynamic for now so my router can automatically assign a random private IP to the LXC. I also mounted the HDD that I want to use as storage to a specific file (in this case it would be /data), where I would then install Samba (SMB - Server Message Block) to complete all the fileshare for me.

When creating a new instance, always do the following code:
```
sudo apt update && sudo apt upgrade -y
```
Once this was completed, I want to **set up a static IP** for the LXC to prevent the IP from releasing/renewing. You can search the current IP via this command and make the adjustments on the **"Network" tab** via the LXC we've created.
```
ip a
```
We can create our fileshare directory here and also add user (it will provide a prompt to input a password and etc):
```
sudo mkdir data
sudo mkdir personal
adduser brian
adduser brian sudo
su brian
```
> [!note]
> The personal directory is just for me to store and share personal data. It's completely irrelevant from the media server that I am creating.

I created the share directory above, so now we just need  to set up permissiosn
```
sudo chmod -R 0774 /data
sudo chmod -R 0774 /personal
sudo chown -R brian:brian /data
sudo chown -R brian:brian /personal
```
> [!NOTE]
> You can learn more about permissions on https://www.redhat.com/en/blog/linux-file-permissions-explained. It can help explain what the numbers represent and how it is used to set up types of permissions for user, group, others. More info is found via the link provided.

# Samba Set up
To install samba:
```
sudo apt install samba
```
Creating a backup of the default samba configuration:
```
cd /etc/samba
sudo mv smb.conf smb.conf.old
```
Editing the Samba configuration file. Here is the file I've ended up using within the configs.
```
sudo nano smb.conf
```
Samba Configuration
```
[global]
   server string = filestorage
   workgroup = WORKGROUP
   security = user
   map to guest = Bad User
   name resolve order = bcast host
   hosts allow = 10.0.0.0/24
   hosts deny = 0.0.0.0/0
   client min protocol = SMB3
   client max protocol = default
# the default is equal to Samba v3_11
#   local master = yes
#   os level = 255
#   preferred master = yes

[data]
   path = /data
   force user = brian
   force group = brian
   create mask = 0774
   force create mode = 0774
   directory mask = 0775
   force directory mode = 0775
   browseable = yes
   writable = yes
   read only = no
   guest ok = no

[personal]
   path = /personal
   force user = brian
   force group = brian
   create mask = 0774
   force create mode = 0774
   directory mask = 0775
   force directory mode = 0775
   browseable = yes
   writable = yes
   read only = no
   guest ok = no
```
Setting up my new samba password for log in purposes.

```
sudo smbpasswd -a brian
```
Setting up autostarting the services if a reboot ever occurs
```
sudo systemctl enable smbd nmbd
sudo systemctl restart smbd nmbd
```
Allowing firewall rules for Samba. (Please note that Samba requires port 137/udp, 138/udp, 139/tcp, 445/tcp) - You could also just type in the following command to enable them and check their status:
```
sudo ufw allow Samba
sudo ufw status
```
I also wanted to set this up on Windows 11 Fileshare directory. To do this, all I have to do is install a package with the following command:
```
sudo apt install wsdd
```
From there, I could easily type in \\10.0.0.x to locate my fileshare via my Windows 11 Home PC
<img width="1156" height="631" alt="image" src="https://github.com/user-attachments/assets/bd8c610c-d484-4109-8719-da40285a8e74" />
