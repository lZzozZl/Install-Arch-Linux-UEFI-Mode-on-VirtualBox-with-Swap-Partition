# Install Arch Linux UEFI Mode on VirtualBox with SWAP Partition
How to install UEFI Arch Linux on VirtualBox VM









In this tutorial im using VBox 6.0.10 with Arch linux 2019.10.01 iso


---
Button new  
Name the machine  
Choose folder where you want your vm to be  
On type choose Linux  
On verison choose Arch 32 or 64 it depends on your processor  
Next

Choose amount of ram. *Personaly i add 2048MB*  
Next  

Whell we create a new instalatiion and we do not have other VMs so we choose  
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

###### Making VM Start at UEFI Mode 
Before startint virtual machine press right mouse button on the VM you just created and choose ###### Settings... from the menu that pop up, or select the machine with left mouse button and at the top of the VBox Manager choose Orange gear Settings  

At the left side choose System 

On the ###### Motherboard tab check the Enable EFI (special OSes only)  
OK  

Start  

Choose virtual optical disc file  
Choose your downloaded iso  
Start  

---  
Choose Arch Linux archiso x86_64 UEFI USB

Wait  

Check are you installing in UEFI mode  
`ls /sys/firmware/efi/efivars`  
*If you do not see any output or your folder is empty, something is wrong.*  

Check if you have internet  
`ping archlinux.org`  

Check time and date  
`date`  
`timedatectl status`  

If tame and date are right continue  
`timedatectl set-ntp true`  

If they are not:  
`timedatectl show`  

*( mine are not. I found that timezone is different )  
I opened https://jlk.fjfi.cvut.cz/arch/manpages/man/timedatectl.1  
Found command to change timezone  
`timedatectl set-timezone [Region]/[Your Capital city]`*

`date` -To see did you change the region

Lets see our HDD information  
`lsblk`  
`fdisk -l`  


To start creating partitions we will use frisk tool.  
*They are many tools that can do the same job allredy preunstalled
Note: If you are thinking to do this on your main machine, not on your virtual one read more about what /dev/sda, /dev/sdb, /dev/sdc .., .. means*  
`fridk /dev/sda`  

###### Tool started    
*press m, than hit enter/return     - Help*  

###### Making boot drive:
* n
* enter or p
* enter or 1
* enter start from first bits
* +1G
* t - changing type of partition
* 1-st partition that we just make
* l to see codes
* ef EFI (FAT-12/16/  - *NOTE: may be you will not see but there is 32 also. At the end it will look like EFI (FAT-12/16/32)*
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




`p` - to see is everything cool  
if everything is cool press `w` to write changes  
you can check with `lsblk` and `fdisk -l` to be complettely shure  






###### Creatinf file systems/Formating partitions  
**boot drive:**
* mkfs.fat -F32 /dev/sda1


**swap partition:**  
* mkswap /dev/sdX2  
* swapon /dev/sdX2  


**root partition:**  
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
`pacstrap /mnt base base-devel linux linux-firmware vim`

wait...  


###### Setting up linux enviroment  
`genfstab -U /mnt >> /mnt/etc/fstab` - generating file with mounting points  

*Check:  
`genfstab -U /mnt`  
`cat /mnt/etc/fstab`  

* Chroot  
`arch-chroot /mnt`  

* Setting local time  
`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`  

`hwclock --systohc`  

* Localization  
`vi /etc/locale.gen` - Uncoment en_US.UTF-8 UTF-8  
`locale-gen`  
`echo "LANG=en_US.UTF-8" > /etc/locale.conf`  

* Network configuration  
`vim /etc/hostname` - Creating hostname ( Name of your computer ).  
It will be empty file. Write some name and than `:wq`  


* Setuping host file  
`vim /etc/hosts`  
`127.0.0.1	localhost`  
`::1		    localhost`  
`127.0.0.1	*hostname*.localdomain	*hostname`  

* Network Manager  
`pacman -S networkmanager`  
`systemctl enable NetworkManager`  


<details><summary>* Or dhcpcd enable</summary>
<p>  
  
`systemctl enable dhcpcd@eth0.service`  
`systemctl start dhcpcd@eth0.service`  
The `eth0` is the network interface. If you are using different get tha name of it and change it.  

</p>
</details>

(intramfs ?)

Create password for root account  
`passwd`  
write some password  
retype password  

creating login user  
`useradd -g users -G wheel,storage,power -m username`  
(for user to be root we need to edit some confs)

###### Boot loader
`pacman -S grub efibootmgr`  
`grub-install --target=x86_64-efi --efi-directory=/boot`  

`pacman -S os-prober` - if you have multiple OS's on the pc this is good to have  
`grub-mkconfig -o /boot/grub/grub.cfg`  

###### Virtualbox fix  
We need to do this if we install on virtual box as we are doing right now.
`mkdir /boot/EFI/boot`  
`cp /boot/EFI/arch/grubx64.efi /boot/EFI/boot/bootx64.efi`  

</br>
`exit`  
`umount -a` - if it work at all. For me it always says that drives are busy  
`reboot`  


---


login with:  
`root`  
`your password`

if you manage to boot and login you are almost done.  
Now `shutdown -r 0`  

---

There is a chance your bootable iso to start again. If this happen:  
* Select VM from VBox Manager
* Settings
* Storage
* On controller IDE you will se the name of your .iso image.
* Click right button on that .iso file
* Remove attachment
* Remove
* OK

Start the machine

### Congrats you have installed Arch linux in UEFI mode with SWAP drive


















