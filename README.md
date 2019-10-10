# Install Arch Linux UEFI Mode on VirtualBox with Swap Partition
How to install UEFI Arch Linux on VirtualBox VM









In this tutorial im using VBox 6.0.10 with Arch linux 2019.10.01 iso

Button new

---
Name the machine
Choose folder where you want your vm to be
On type choose Linux
On verison choose Arch 32 or 64 it depends on your processor
Next

Choose amount of ram. Personaly i add 2048MB wich is more than enough
Next

Whell we create a new instalatiion and we do not have other vms so we choose
Create virtual hard disk now
Next

I do not know difference between the typed and i choose the default option
VDI (VirtualBox Disk Image)
Next

Again i do not know what options do and use default one
Dinamycally allocated
Next

On HDD space i place 20,00GB
Next

---

Making VM Start at UEFI Mode
Before start press right mouse button on the VM you just created and choose Settings... from the menu that pop up, or select the machine with left mouse button and at the top of the VBox Manager choose Orange gear Settings

At the left side choose System

On the Motherboard tab check the Enable EFI (special OSes only)
OK

Start
Choose virtual optical disc file
Choose your downloaded iso
Start


Choose Arch Linux archiso x86_64 UEFI USB

Wait

Check are you installing on UEFI mode
ls /sys/firmware/efi/efivars
If you do not see any output or your folder is empty, something is wron.

Check if you have internet
ping archlinux.org

Check time and date
```
date
timedatectl status
```

If tame and date are right continue
If they are not:
timedatectl show

( mine problem was timezone )
I opened https://jlk.fjfi.cvut.cz/arch/manpages/man/timedatectl.1
Found command to change timezone
timedatectl set-timezone Europe/[Your Capital city]

date    -To confirm



Lets see what we have
lsblk
fdisk -l


To start creating partitions we will use frisk tool. They are many toolsa llredy preunstalled
Note: If you are thinking to do this on your main machine not in virtual one read more about what /dev/sda, /dev/sdb, /dev/sdc .., .. means

fridk /dev/sda

Tool started
press m, than hit enter/return     - Help



###### Making boot drive:
* n
* enter or p
* enter or 1
* enter start from first bits
* +1G
* t - changing type of partition
* 1-st partition that we just make
* l to see codes
* ef EFI (FAT-12/16/  - NOTE: may be you will not see but there is 32 also. At the end it will look like EFI (FAT-12/16/32)
* a
* select boot drive that we just make




###### Making swap drive:
* n
* enter or p
* enter or 2
* enter
* +4G
* t - changing type of partiton
* 2-nd partition that we just make
* l to see codes
* 82 Linux swap / Solaris







###### Making root partition:
* n
* enter or p
* enter or 3
* enter
* enter - get rest of the space




p - to see is everything cool  
if everything is cool press w to write changes  
you can check with lsblk and fdisk -l to be complettely shure  






Creatinf file systems  
boot drive:
* mkfs.fat -F32 /dev/sda1


swap partition:  
* mkswap /dev/sdX2  
* swapon /dev/sdX2  


root partition:  
* mkfs.ext4 /dev/sda3

---

Ok. This was easy part. Now start confusing one.

Mounting file system to /root partition  
* mount /dev/sda3 /mnt
* mkdir /mnt/boot - creating boot directory inside mount point
* mount /dev/sda1 /mnt/boot- Mounting UEFI partition to /mtn/boot

Lets check if mouning points are ok  
`df` or `lsblk` or `findmnt` or `mount`  - Choose one or two commands to check :)

---

###### Mirrorlist:  
__*NOTE: Read how to work with vi or vim editor.*__   
`vi /etc/pacman.d/mirrorlist`  

In command mode go to line to your closest region server  
`/Country` - search ( most likley first letter should be capital or it will say that it did not find it )  
`yy` - copy line  
Go to the top of your document again and paste wit  
`p`
`:wq` - this will write changes to file and it will close the document

###### Installing the system:  
`pacstrap /mnt base base-devel`

wait...  















