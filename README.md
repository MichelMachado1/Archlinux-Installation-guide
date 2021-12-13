
Arch Linux Base Install on UEFI 
========

   * [Keymap](#keymap)
   * [Internet](#Internet)</br>
   * [Mirrors](#Mirrors)</br>
   * [Partitioning](#Partitioning)</br>
   * [Formatting](#Formatting)</br>
   * [Mounting](#Mounting)
   * [Base Install](#Base_Installation)
   * [FSTAB](#FSTAB)
   * [Chroot](#Chroot)
   * [Locales](#Locales)
   * [Hostname](#Hostname)
   * [Password](#Password)
   * [GRUB](#GRUB)
   * [Enable Services](#Enable Services)
   * [New User](#New_User)
   * [Reboot](#Reboot)
   * [Internet](#Internet)
   * [Graphics Driver](#Graphics Driver)
   * [YAY](#YAY)






*by [MichelMachado1](https://github.com/MichelMachado1)

<!---
BEGIN
-->

===============================================================================================

## Keymap
fetch your keymap layout name
```
localectl list-keymaps | grep fr
```
The default console keymap is US. Set the azerty layout with:
```
# loadkeys fr-latin1
```

## Internet
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

## Mirrors
reflector -c France -a 6 --sort rate --save /etc/pacman.d/mirrorlist

## Partitioning

## Verify the boot mode
To verify the boot mode, list the efivars directory: 
```
# ls /sys/firmware/efi/efivars
```
If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in BIOS (or CSM) mode.

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
|   /boot or /efi     |   /dev/sda1   |   EFI system partition   |      +300M         |
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







## Mounting
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





## Base_Installation


### Install essential packages
Use the pacstrap script to install the base package, Linux kernel and firmware for common hardware: 
```
# pacstrap /mnt base linux linux-firmware intel-ucode vim
```



## FSTAB
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




## Locales
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


## Hostname

Create the hostname file:
```
vim /etc/hostname

myhostname
```


## Password

Set the root password:
```
# passwd
```

## GRUB

Choose and install a Linux-capable boot loader. If you have an Intel or AMD CPU, enable microcode updates in addition. 

amd-ucode for AMD processors

```
# pacman -S grub efibootmgr networkmanager network-manager-applet wireless_tools wpa_supplicant dialog os_prober mtools dosfstools base-devel linux-headers bluez bluez-utils cups git pulseaudio pulseaudio-bluetooth pulseaudio-alsa pulseaudio-equalizer pulseaudio-jack xdg-utils xdg-user-dirs
```
```
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
```

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
## Enable Services
```
systemctl enable NetworkManager
```
```
systemctl enable bluetooth
```
```
systemctl enable cups.service
```

## New_User
```
useradd -mG wheel michel
```

## Reboot

Exit the chroot environment by typing exit or pressing Ctrl+d.

Optionally manually unmount all the partitions with umount -R /mnt: this allows noticing any "busy" partitions, and finding the cause with fuser.

Finally, restart the machine by typing reboot: any partitions still mounted will be automatically unmounted by systemd. Remember to remove the installation medium and then login into the new system with the root account. 

# AFTER REBOOT 








