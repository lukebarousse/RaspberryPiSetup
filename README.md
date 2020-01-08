# Raspberry Pi Initial Setup
------
### Sources:  
[Gregology - Raspberry Pi Time Machine](https://gregology.net/2018/09/raspberry-pi-time-machine/)  
[Raspberry Pi - Installing Images](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)  
[Pi my lifeup - Apple File Protocol](https://pimylifeup.com/raspberry-pi-afp/)  
[How to geek - Using Pi as Time Machine](https://www.howtogeek.com/276468/how-to-use-a-raspberry-pi-as-a-networked-time-machine-drive-for-your-mac/)  
------
### Legend
1) Download OS for Pi
2) Unmount SD Card
3) Copy image to SD card (MacOS-Only)
4) Boot up Raspberry Pi
5) Give static IP address (Optional)
6) Set up SSH
7) Shell into Pi
8) Update and Upgrade Pi
9) Set up Remote Desktop (Optional)
10) Install External Storage (Optional)
11) Set up File Sharing (Optional)
12) Autoconnect to Network (MacOS-Only)  
13) EXTRA: Troubleshooting
------
### 1) Download OS for Pi
[Raspberry Pi - Raspian Download](https://www.raspberrypi.org/downloads/raspbian/)  
- Chose Download Zip Raspian Buster with desktop and recommended software (Takes 15-20 minutes)
- Unzip when done

### 2) Unmount SD Card
- On your local computer, determine what disks are installed
```
$ diskutil list
```
- Insert Smart Card into mac, determine disk number 
```
$ diskutil list
```
- Determine disk number from command above (i.e. /dev/diskN) and unmount
```
$ diskutil unmountDisk /dev/diskX
```

### 3) Copy image to SD Card (MacOS-Only)
[Raspberry Pi Installing Images Source(Windows Directions)](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)  
- On your local computer, flash the image to the card (Windows use Etcher)
```
$ sudo dd bs=1m if=/Users/‘Luke Barousse’/Desktop/2019-09-26-raspbian-buster-full.img of=/dev/rdiskN conv=sync (15-20 minutes)
#check progress with (Ctrl + t)
```
- When done, eject the disk
```
$ sudo diskutil eject /dev/rdiskN
```
### 4) Boot up Raspberry Pi
- Connect all peripherals boot up system (mouse, key, ethernet, power, HDD)
- Choose the operating system (Raspian) and install 

### 5) (Optional) Give your Pi a local static IP Address (Note: Easier to consistently access via SSH at same address)
Note: Steps below are for an ATT router
- In your web browser enter the IP address of the router (ex. http://192.168.1.254)
- Select Settings > LAN > Lan IP address Allocation
- If prompted enter device access code (ex. 10 digit code)
- Go to Device and for address assignment select
- Private Fixed: 192.168.1.100 (choose any number of your liking)
- On Pi, after Pi shutdown and/or restart verify static IP address reset by
```
$ ifconfig

#via Wifi (look for wlan0)
#via Ethernet (look for eth0)
#Verify the local ip address: 192.168.1.100 (i.e. matches above)
```

### 6) Set up to connect via shell (SSH)
- On Pi, go to configuriation settings
```
$ sudo raspi-config
```
- Select option #5 then #2, and enable SSH
- Shutdown, if moving and removing peripherals: 
```
$ sudo shutdown -h now
```
- At minimum, restart the Pi (ethernet, power, HDD)
```
$ sudo reboot
```

### 7) Shell into Pi (Windows users require 'Putty')
- On your local computer, shell into the Pi
```
$ ssh pi@192.168.1.100
# username: pi
# password: raspberry
```

### 8) Update and Upgrade Pi
- On Pi, update and upgrade Pi
```
$ sudo apt-get update
$ sudo apt-get upgrade
# Takes 15-20 mins
```

### 9) (Optional) Install Remote Desktop
- On pi, install the vncserver app
```
$ sudo apt install realvnc-vnc-server realvnc-vnc-viewer
```
- Enable VNC on pi
```
$ sudo raspi-config
# Interface Options > VNC > Yes
```
- Start VNC
```
$ vncserver
# To change default screen size (recommended)
$ vncserver :1 -geometry 1920x1080 -depth 24
```
- On your local computuer, install RealVNC application on computer: [Real VNC Download](https://www.realvnc.com/en/connect/download/viewer/macos/)
- Launch app, and enter IP address screen number: 192.168.1.100:1 (or local Pi IP address chosen)
- (Optional) File Manager would not open properly for me, so I had to reinstall this:
```
$ sudo apt-get install --reinstall pcmanfm
```
- When no longer needed to VNC stop the VNC server
```
$ vncserver -kill :1
```

### 10) (Optional) Install External Storage
NOTE: Prior to beginning I partitioned the hard drive with 1TB for regular files (ExFat) and 1 TB for Mac OS extended -Journaled (hfsplus - i.e., time machine)
- On Pi, list all the disk partitions
```
$ sudo lsblk -o UUID,NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL,MODEL
```
- Determine FSTYPE

```
# For NTFS-3g
$ sudo apt update
$ sudo apt install ntfs-3g

# FOR EXFAT
$ sudo apt update
$ sudo apt install exfat-fuse
```
- Get the location of disk partition (e.g. /dev/sda1)
```
$ sudo blkid 
```
- Install hfsutils & hfsprogs (this app formats harddrive for Time Machine)
```
$ sudo apt-get install hfsutils hfsprogs
```
- Format the selected time machine drive
```
$ sudo mkfs.hfsplus /dev/sda2 -v timemachine
```
- Create location for mount point
```
$ sudo mkdir /mnt/tm
```
- Mount the drive
```
$ sudo mount /dev/sda3 /mnt/tm
```
- Verify mounted correctly
```
$ ls /mnt/tm
```
- Give read/write/edit permissions
```
$ sudo chmod 777 /mnt/tm
```
- Change permissions of the directory (may need to run again at end for troubleshooting)
```
$ sudo chown pi:pi /mnt/tm
```
- To access after reboot edit the fstab
```
$ sudo nano /etc/fstab

# Replace FSTTYPE with Type (exfat or hfsplus) (noauto specified to not load on startup, load on startup is not consistent)
UUID=5C24-1453 /mnt/2tbhd FSTYPE force,rw,user,noauto 0 0
```
- To auto mount after reboot edit crontab (commands to execute on startup after a set amount of time to make sure drives loaded properly):
```
$ sudo crontab -e

# append the following
@reboot sleep 30 && sudo mount /media/tm >> /home/pi/TimeMachineMountStartUp.log 2>&1
@reboot sleep 40 && sudo chmod -R 777 /media/tm >> /home/pi/TimeMachineCHMOD.log 2>&1
@reboot sleep 50 && sudo service avahi-daemon start >> /home/pi/Avahi-DaemonServiceStartup.log 2>&1
@reboot sleep 60 && sudo service netatalk start >> /home/pi/NetatalkServiceStartUp.log 2>&1
```
- If necessary, to unmount
```
$ sudo umount /mnt/tm
```

### 11a) (Optional) Set up Netatalk for file sharing (MacOS)
[Apple File Protocol](https://pimylifeup.com/raspberry-pi-afp/)  
[Using Pi as Time Machine](https://www.howtogeek.com/276468/how-to-use-a-raspberry-pi-as-a-networked-time-machine-drive-for-your-mac/)  
- On Pi, ensure everything up to date
```
$ sudo apt update && sudo apt upgrade
```
- Install Netatalk
```
$ sudo apt install netatalk
```
- Configure Netatalk
```
$ sudo nano /etc/netatalk/afp.conf

#Edit the file as shown below
[Global]
mimic model = TimeCapsule6, 106
[Homes]
  basedir regex = /home
[Files]
  path = /mnt/Files
[Time Machine]
path = /mnt/tm
  time machine = yes
```
- Edit the nsswitch config file
```
$ sudo nano /etc/nsswitch.conf

#Add mdns4 mdns  on ‘host:’ line
```
- Start the service 
```
$ sudo service avahi-daemon start
$ sudo service netatalk start
```
- Connect to Raspberry Pi by typing CMD + k (Mac), type 'afp://192.168.1.100'
- Connect via default username (pi) and passoword (raspberry)
- Select volumes to mount (i.e., Shared Pi2TBHDD)

### 11b) (Optional) Set up Samba for file sharing (Windows Computer)
- On Pi, install Samba
```
$ sudo apt-get install samba samba-common-bin
```
- Create a shared directory, putting it in ‘shared’ in the hard drive
```
$ sudo mkdir -m 1777 /mnt/2tbhd/shared
```
- Edit the Samba config file
```
$ sudo nano /etc/samba/smb.conf

#Edit the file as shown below
[Shared Pi2TBHDD]
Comment = Shared Folder
Path = /mnt/2tbhd/shared
Browseable = yes
Writeable = Yes
only guest = no
create mask = 0777
directory mask = 0777
Public = yes
Guest ok = yes
```
- Restart Samba
```
$ sudo /etc/init.d/samba restart
```

### 12) (Optional) Setup Auto-connect to Network Drive (MacOS-Only)
- On Mac open System Preferences > Users & Groups
- Select UserName (Unlock, if necessary) & select 'Login Items'
- Click + and navigate to folders from drive to add
- Check 'Hide' box to keep windows from opening on login/boot

### 13) EXTRA: Troubleshooting
1. If saying Time Machine is read-only can change the permissions of directory
```
# determine location of mount for Time machine
$ sudo lsblk -o UUID,NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL,MODEL

# then insert correct location from MOUNTPOINT
$ sudo chown pi:pi /mnt/TimeMachine
```
OR forcing fsck.hfsplus to check and repair journaled HFS+ file systems
```
#determine location of mount for Time machine or using NAME column above
$ sudo blkid 

#repairing time machine
$ sudo fsck.hfsplus -f /dev/sda2
```
2. If errors with connecting or backing up (Noted during initial download)
- reboot the pi
- restart/shutdown local computer
- reconnect to server
