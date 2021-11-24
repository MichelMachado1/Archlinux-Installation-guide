# Archlinux-Installation-guide
Archlinux Installation guide UEFI with swap partition

<br/> https://wiki.archlinux.org/title/installation_guide 




    
        Set the console keyboard layout
        Verify the boot mode
        Connect to the internet
        Update the system clock
        Partition the disks
        Example layouts
        Format the partitions
        Mount the file systems
        Installation
        Select the mirrors
        Install essential packages
   Configure the system
        Fstab
        Chroot
        Time zone
        Localization
        Network configuration
        Initramfs
        Root password
        Boot loader
     Reboot
     Post-installation












## Set the keyboard layout
The default console keymap is US. Set the azerty layout with:
```
# loadkeys fr-latin1
```
## Verify the boot mode
To verify the boot mode, list the efivars directory: 
```
# ls /sys/firmware/efi/efivars
```
If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in BIOS (or CSM) mode.

## Connect to the internet
Ensure your network interface is listed and enabled, for example with ip-link: 
```
# ip link
```

### Wireless Connection
If you are on a laptop, you can connect to a wireless access point using `iwctl` command from `iwd`. Note that it's already enabled by default. Also make sure the wireless card is not blocked with `rfkill`.

Scan for network.
```
# iwd station wlan0 scan
```
Get the list of scanned networks by:
```
# iwd station wlan0 get-networks
```
Connect to your network.
```
# iwd station wlan0 connect "NETWORKNAME"
```
exit the iwd with the exit command
```
# iwd exit
```
The connection may be verified with ping: 
```
# ping archlinux.org
``` 
ctrl + c to exit.

## Update the system clock
Use `timedatectl` to ensure the system clock is accurate:
```
# timedatectl set-ntp true
```
To check the service status, use `timedatectl status`.


## Partition the disks
When recognized by the live system, disks are assigned to a block device such as /dev/sda, /dev/nvme0n1 or /dev/mmcblk0. To identify these devices, use lsblk or fdisk.
```
# lsblk
```
Use fdisk or parted to modify partition tables. For example: 
```
# fdisk /dev/sda
```
https://wiki.archlinux.org/title/Partitioning#UEFI/GPT_layout_example

|   Mount point   |   Partition   |   Partition type         |   Suggested size   |
|   ------------- | ------------- | ---------------------    | ------------------ |
|   /boot or /efi     |   /dev/sda1   |   EFI system partition   |      +500M         |
|          [SWAP] |         /dev/sda2      |           Linux swap                |      2048 MiB              |
|          /mnt       |   /dev/sda3   |   Linux x86-64 root (/)               |       Remainder of the device       |


## Format the partitions
an EFI system partition must contain a FAT32 file system
```
# mkfs.fat -F 32 /dev/sda1
```
Initialize swap partition with mkswap:
```
# mkswap /dev/sda2
```
https://wiki.archlinux.org/title/EFI_system_partition

to create an Ext4 file system on /dev/root_partition, run: 
```
# mkfs.ext4 /dev/sda3
```


https://wiki.archlinux.org/title/File_systems#Create_a_file_system







## Mount the file systems
Mount the root volume to /mnt
```
# mount /dev/sda3 /mnt
```
mount the EFI system partition:

```
# mkdir /mnt/boot

# mount /dev/sda1 /mnt/boot
```
Enabable the swap partition
```
# swapon /dev/sda2
```





## Installation
## Select the mirrors
The mirrors are automatically set
On the live system, after connecting to the internet, reflector updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate. 

### Install essential packages
Use the pacstrap script to install the base package, Linux kernel and firmware for common hardware: 
```
# pacstrap /mnt base linux linux-firmware
```
> you can install linux-lts instead if linux or both

other essential packages
```
# pacman -S linux-headers
```
> you can install linux-lts-headers instead or both

a text editor
```
# pacman -S vim
```
almost essential packages include sudo cmd
```
# pacman -S base-devel
```
wifi pakage support
```
# pacman  -S networkmanager wpa_supplicant wireless_tools dialog
```
auto start NetworkManager
```
# systemctl enable NetworkManager
```
lvm support REQUIRED for hard drive
```
pacman -S lvm2
```
Xorg provides an interface between your hardware and the graphical software you want to run.
```
pacman -S xorg-server
```
video driver
```
pacman -S mesa
```
<details><summary># vim /etc/mkinitcpio.conf</summary>
<p>
 
insert **lvm2** inside HOOKS between block and filesystems
 
#### HOOKS=(base udev autodetect modconf block _lvm2_ filesystems keyboard fsck)

</p>
</details>

make the options take effect
```
# mkinitcpio -p linux
```
## Configure the system
### Fstab

Generate an fstab file: 
```
mkdir /mnt/etc
```
```
# genfstab -U -p /mnt >> /mnt/etc/fstab
```

Check the resulting /mnt/etc/fstab file.
```
cat /mnt/etc/fstab
```
should fetch:

|# /dev/mapper/volgroup0-rootvol ||||                                            | 
| ---                            | ---               | ---   | ---         | --- |
|UUID=6ecea935-5efa-43f3-b93b-f525887a1b34|    /     | ext4  | rw,relatime | 0 1 |

|# /dev/mapper/volgroup0-homevol ||||                                            | 
| ---                            | ---               | ---   | ---         | --- |
|UUID=aa72e511-f2cf-46bd-abc7-9867cc92eb5c|    /home     | ext4  | rw,relatime | 0 2 |
> note: you will have different UUID's.
# Chroot
Change root into the new system: 
```
# arch-chroot /mnt
```
## Time zone

Set the time zone:
```
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
for France is Europe/Paris
if you don't know your time zone you can set it after rebooting with `timedatectl` </br>
https://wiki.archlinux.org/title/System_time#Time_zone

Run hwclock to generate /etc/adjtime:
```
# hwclock --systohc
```




## Localization
Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed locales. Generate the locales by running: 
```
# locale-gen
```
Create the locale.conf file, and set the LANG variable accordingly: 
|  /etc/locale.conf |
|---|
|  LANG=en_US.UTF-8 |

Make the keyboard layout changes persistent in vconsole.conf:
|  /etc/vconsole.conf |
|---|
|  KEYMAP=fr-latin1 |


## Network configuration

Create the hostname file:
```
vim /etc/hostname

myhostname
```
Create the hosts file
```
vim /etc/hosts

127.0.0.1 localhost
127.0.1.1 myhostname
```

## Root password

Set the root password:
```
# passwd
```
recommanded: add a standard user for yourself
```
useradd -m -g USERS -G wheel michel
```
michel is my name put your name instead.
add password for yourself
```
passwd michel
```

<details><summary># visudo</summary>
<p>
 
 Uncomment to allow members of group wheel to execute any command
#### %wheel ALL=(ALL) ALL

</p>
</details>



## Boot loader

Choose and install a Linux-capable boot loader. If you have an Intel or AMD CPU, enable microcode updates in addition. 
```
# pacman -S intel-ucode
```
amd-ucode for AMD processors
source: https://wiki.archlinux.org/title/Microcode#Early_loading
```
# pacman -S grub efibootmgr dosfstools os-proger mtools
```
```
# grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```
source: https://wiki.archlinux.org/title/GRUB#Installation_2
check for the locale directory
```
# ls -l /boot/grub
```
set the grub/boot screen language
```
# cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
```
generate the grub configuration file
```
grub-mkconfig -o /boot/grub/grub.cfg
```

## Reboot

Exit the chroot environment by typing exit or pressing Ctrl+d.

Optionally manually unmount all the partitions with umount -R /mnt: this allows noticing any "busy" partitions, and finding the cause with fuser.

Finally, restart the machine by typing reboot: any partitions still mounted will be automatically unmounted by systemd. Remember to remove the installation medium and then login into the new system with the root account. 

# AFTER REBOOT 

### Swap file creation
> source: https://wiki.archlinux.org/title/Swap
```
# dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
```
Set the right permissions (a world-readable swap file is a huge local vulnerability): 
```
# chmod 600 /swapfile
```
After creating the correctly sized file, format it to swap: 
```
# mkswap /swapfile
```
optional:
Making a backup copy of the fstabfile in case of mistake
```
# cp /etc/fstab /etc/fstab.bak
```
Finally, edit the fstab configuration to add an entry for the swap file: 
```
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
```
activate
```
# swapon -a
```
check with free -m

other sources : <br/> LearLinuxTV
https://youtu.be/DPLnBPM4DhI 






