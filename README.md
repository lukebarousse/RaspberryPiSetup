# Raspberry Pi Initial Setup
------
### Sources:  
[Paul Mucur - Using a Raspberry Pi for a Time Machine](https://mudge.name/2019/11/12/using-a-raspberry-pi-for-time-machine/)  
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
12) EXTRA: Troubleshooting
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
- Determine disk number from command above (i.e. /dev/diskN) and unmount (to prevent 'resource busy' error)
```
$ diskutil unmountDisk /dev/diskX
```

### 3) Copy image to SD Card (MacOS-Only)
[Raspberry Pi Installing Images Source(Windows Directions)](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)  
- On your local computer, flash the image to the card (Windows use Etcher)
```
$ sudo dd bs=1m if=/Users/UserName/Downloads/2019-09-26-raspbian-buster-full.img of=/dev/rdiskN conv=sync (15-20 minutes)
#check progress with (Ctrl + t)
#use rdiskN vice diskN as this expediates the load process
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
- Allow read/write/edit permissions for pi (May not be necessary)
```
$ sudo chmod 777 /mnt/tm
```
- Create a separate user for storing backups to match the username on your local computer
```
sudo adduser username
```
- Change file ownership (recursively) of the mounted directory  
```
$ sudo chown -R username: /mnt/tm
```
- To access after reboot edit the fstab
```
$ sudo nano /etc/fstab

# Replace FSTTYPE with Type (exfat or hfsplus)
# noauto specified to not load on startup, load on startup is flaky
UUID=5C24-1453 /mnt/2tbhd FSTYPE force,rw,user,noauto 0 0
```
- To auto mount after reboot edit crontab (crontab is commands to execute on startup after a set amount of time to make sure drives loaded properly):
```
$ sudo crontab -e

# append the following
@reboot sleep 30 && sudo mount /media/tm >> /home/pi/TimeMachineMountStartUp.log 2>&1
@reboot sleep 40 && sudo chmod -R 777 /media/tm >> /home/pi/TimeMachineCHMOD.log 2>&1
```
- If using a spinning disk HDD (as Time Machine doesn’t run constantly), put the disk in standby after 10 minutes of inactivity. To do this, we’ll need to install hdparm to set a standby (spindown) timeout in 5 second increments, again identifying our disk by its UUID:
```
$ sudo apt install hdparm
$ sudo hdparm -S 120 /dev/disk/by-uuid/e613b4f3-7fb8-463a-a65d-42a14148ea65
```
- Make this permanent by adding the following stanza to /etc/hdparm.conf:
``` 
$ sudo nano /etc/hdparm.conf

#add to bottom of file
/dev/disk/by-uuid/e613b4f3-7fb8-463a-a65d-42a14148ea65 {
	spindown_time = 120
}
```

### 11a) (Optional) Set up Samba (SMB) for file sharing (recommended-option)  
[Using a Raspberry Pi for Time Machine](https://mudge.name/2019/11/12/using-a-raspberry-pi-for-time-machine/)
- On Pi, if not done recently, ensure everything is up to date
```
$ sudo apt update && sudo apt upgrade
```
- Install Samba
```
$ sudo apt-get install samba
```
- Edit the SMB config file
```
$ sudo nano /etc/samba/smb.conf

# The following already exists, just make minor changes, this only adds the directory for the username you created above
[homes]
	browseable = No 
	comment = Home Directories
	create mask = 0700
	directory mask = 0700
	read only = No              #Changed from Yes to No
	valid users = username      #Add the User Name

# The following doesn't exist and needs to be added
[TimeMachine]
    comment = Time Machine
    path = /mnt/TimeMachine
    valid users = username
    read only = no
    vfs objects = catia fruit streams_xattr
    fruit:time machine = yes

# The following doesn't exist and needs to be added to get your pi home directory
[PiDirectory]
	comment = Pi Directory
	path = /home/pi
	read only = No
	valid users = username

# The following doesn't exist and needs to be added if you partitioned your hard drive with a time machine and regular file system
[FilesDirectory]
	comment = FilesDirectory
	path = /mnt/Files
	read only = No
	valid users = pi
```
- Add username to Samba's password file and set password; This is the username and password to connect to pi from mac
```
$ sudo smbpasswd -a username
```
- Run testparm to check our configuration is free of errors
``` 
$ sudo testparm -s
```
- Reload Samba configuration
```
$ sudo service smbd reload
```
- Configure Avahi to load pi automatically in finder by editing samba.service file
``` 
$ sudo nano /etc/avahi/services/samba.service

# add the following to the file, this will adviertise the pi as AirPort Time Capsule
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">%h</name>
  <service>
    <type>_smb._tcp</type>
    <port>445</port>
  </service>
  <service>
    <type>_device-info._tcp</type>
    <port>9</port>
    <txt-record>model=TimeCapsule8,119</txt-record>
  </service>
  <service>
    <type>_adisk._tcp</type>
    <port>9</port>
    <txt-record>dk0=adVN=TimeMachine,adVF=0x82</txt-record>       #this name must match name in brackets from smb.conf
    <txt-record>sys=adVF=0x100</txt-record>
  </service>
</service-group>
```
- On local computer, open Finder and select raspberry pi in sidebar
- Click 'Connect As...', and login with username and password defined above
- Make sure to check box to save credentials for future reboots

### 11b) (Optional) Set up Netatalk (AFP) for file sharing (not recommended, I encountered many issues after install)
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
- Setup Auto-connect to Network Drive (MacOS-Only)
- On Mac open System Preferences > Users & Groups
- Select UserName (Unlock, if necessary) & select 'Login Items'
- Click + and navigate to folders from drive to add
- Check 'Hide' box to keep windows from opening on login/boot

### 12) EXTRA: Troubleshooting
1. If you have any trouble connecting (especially if you have changed your user credentials at all), it’s worth using the Keychain Access application in /Applications/Utilities to check for any cached network passwords and delete old entries.  
1. You may also need to forcibly relaunch the Finder by holding down the Option key and right-clicking on its icon in the Dock and choosing the Relaunch option at the bottom of the resulting menu.  
1. If saying Time Machine is read-only need to change the permission of directory
```
# determine location of mount for drive
$ sudo lsblk -o UUID,NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL,MODEL

# then insert correct location from MOUNTPOINT
$ sudo chown username: /mnt/TimeMachine
```
4. If Time Machine needs repair force fsck.hfsplus to check and repair journaled HFS+ file systems
```
#determine location of mount for Time machine or using NAME column above
$ sudo blkid 

#repairing time machine
$ sudo fsck.hfsplus -f /dev/sda2
```
5. If errors with connecting or backing up (Noted during initial download)
- reboot the pi
- restart/shutdown local computer
- reconnect to server
