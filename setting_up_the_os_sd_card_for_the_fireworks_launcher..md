##Setting up the OS SD card for the fireworks launcher.

###Create SD image
[Download](http://downloads.raspberrypi.org/raspbian_latest) the latest Raspbian

Insert your SD card, then in Terminal list the availabe disks. 

	$ diskutil list
	
It should look something like :

	/dev/disk0
	   #:                       TYPE NAME                    SIZE       IDENTIFIER
	   0:      GUID_partition_scheme                        *251.0 GB   disk0
	   1:                        EFI EFI                     209.7 MB   disk0s1
	   2:                  Apple_HFS d-air-inty              250.1 GB   disk0s2
	   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
	/dev/disk5
	   #:                       TYPE NAME                    SIZE       IDENTIFIER
	   0:     FDisk_partition_scheme                        *7.9 GB     disk5
	   1:                 DOS_FAT_32 BOOT                    7.9 GB     disk5s1

My SD card is `/dev/disk5` yours may have a different number. 

Unmount the disk, in my case 
	
	$ diskutil unmountDisk /dev/disk5
	
Format the card, again using the correct disk name.

	$ diskutil eraseDisk FAT32 RASPBIAN  MBRFormat /dev/disk5
	
Unmount the SD card again

	$ diskutil unmountDisk /dev/disk5
	
And do the clone. Make sure you put the correct path to the image and add the correct disk name. Note the `r` in  `/dev/rdisk5`
	
	$ sudo dd if=/path/to/2013-09-25-wheezy-raspbian.img of=/dev/rdisk5 bs=1m
	
This can take a while from 11.25
	
One more unmount

	$ diskutil unmountDisk /dev/disk5
	
and then eject

	$ diskutil eject /dev/disk1
	
Stick your SD card in your pi and power it up.

###Add a password to the root user 
Boot the Pi and follow the initial setup instructions. 

	$ sudo passwd root

Add your own password.

###Set up auto login

Edit /etc/inittab 

	$ sudo nano /etc/inittab
	
Scroll down to: 

	1:2345:respawn:/sbin/getty 115200 tty1
	
And comment it out.

	#1:2345:respawn:/sbin/getty 115200 tty1
	
Add this line

	1:2345:respawn:/bin/login -f root tty1 </dev/tty1 >/dev/tty1 2>&1
	
Save the file and exit, (`ctrl + o` to save and `ctrl + x` to exit) 

Reboot to test.

	$ reboot
	
You should now autologin as root.

###Install ruby

First ensure everything is up to date 


	$ sudo apt-get update
	
Then 

	$ curl -L https://get.rvm.io | bash -s stable --ruby
	
Be very very patient, it will take over an hour and sometimes up to two depending on the speed of your internet connection and your SD card.

Once the process is complete 

	$ ruby -v
	
Should give you something like.

	> ruby 2.0.0p353 (2013-11-22 revision 43784) [armv61-linux-eabihf]
	



