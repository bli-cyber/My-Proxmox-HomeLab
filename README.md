# Proxmox
I'll be using Proxmox v9.1.2 to develop my home server with installations such as JellyFin, Media automation, File Storage, and many more!! Proxmox is mainly used as my virtual environment creation to host multiple lightweight LXC &amp; VMs for my  testing purposes. Documenting all of this for my jorney.

# Hardware
Notes: I just bought an Optiplex and used leftover HDD that I had laying around. I won't be using a backup in this set up but if I would, I'd likely add a 2T HDD in the later future. Though it's entirely dependent on the power supply of the Optiplex that I purchased, it's still a really good chance that this could be supported. If the PSU doesn't support it, I may have to find an alternative to either 1. Replace the current PSU or 2. Utilize another PSU and provide the power there or 3. Find a reason to rebuild a computer because its cool but with $$$ as a downside... You could always spend more ;)

- Intel i5 8500 (6 Core) @3.00GHz
- 16GB 2666mhz 
- 2x1T HDD
- 1x256 NVMe SSD

What you need:
- 16 GB or more empty USB
- A working computer???

# Initial Proxmox Setup
I downloaded Proxmox (https://www.proxmox.com/en/downloads) and retrieved an ISO file, where I would use an application called Rufus (https://rufus.ie/en/#download) to format the ISO file and create a bootable USB flash drive. I just found a 64 GB USB that I had laying around and flashed it from there. 

To re-format the USB along with the Proxmox ISO file, under "Device", select the USB drive you want to set as a bootable device. Under "Boot Selection", we want to select "Disk or ISO image", and then select on the downloaded ISO file. I've left the rest of the forms as default as it will propogate themselves and started the process, which took no more than 3 mins. Once completed, I can finally use this USB to install Proxmox!

<img width="478" height="585" alt="image" src="https://github.com/user-attachments/assets/e3e3922b-dfb1-4e42-ae2e-be3db977f8af" />

**For the install you'd essentially want to select the option to boot up the USB flash drive over the current bootable set up, which im not going to go over. **

Once I got Proxmox booted up, I followed the instructions on each page. One thing I kept in mind was where I wanted the OS to be installed (which I obviously selected the NVMe storage I had for faster booting). As you follow the instructions, be aware of your current interal networking set up. You want to note what your current private IP you are using. For me, I configured my private network using a 10.0.0.0/24 subnet. At the end of this, you should be able to install Proxmox on the selected storage space and remove the USB flash drive.
