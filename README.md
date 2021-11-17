# Archlinux-Installation-guide
Archlinux Installation guide UEFI LVM Swapfile (NO Encryption)

## Set the keyboard layout
The default console keymap is US. Set the azerty layout with:
```
# loadkeys fr-latin1
```

## Connect to the internet
Ensure your network interface is listed and enabled, for example with ip-link: 
```
# ip link
```

### Wireless Connection
If you are on a laptop, you can connect to a wireless access point using `iwctl` command from `iwd`. Note that it's already enabled by default. Also make sure the wireless card is not blocked with `rfkill`.

Scan for network.
```
# iwctl station wlan0 scan
```
Get the list of scanned networks by:
```
# iwctl station wlan0 get-networks
```
Connect to your network.
```
# station wlan0 connect "NETWORKNAME"
```

Test internet with Google Public DNS to make sure we are online:
```
# ping -c 5 8.8.8.8
``` 

## Update the system clock
Use `timedatectl` to ensure the system clock is accurate:
```
# timedatectl set-ntp true
```
To check the service status, use `timedatectl status`.

## Verify the boot mode
To verify the boot mode, list the efivars directory: 
```
# ls /sys/firmware/efi/efivars
```
If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in BIOS (or CSM) mode.

## Partition the disks
When recognized by the live system, disks are assigned to a block device such as /dev/sda, /dev/nvme0n1 or /dev/mmcblk0. To identify these devices, use lsblk or fdisk.
```
# fdisk -l
```
Use fdisk or parted to modify partition tables. For example: 
```
# fdisk /dev/sda
```

|   Mount point   |   Partition   |   Partition type         |   Suggested size   |
|   ------------- | ------------- | ---------------------    | ------------------ |
|   /boot/efi  or /mnt/efi     |   /dev/sda1   |   EFI system partition   |      +500M         |
|                 |   /dev/sda2   |   Linux LVM              |       100%         |


## LVM building blocks
> Volume operations

## Physical volumes
To create a PV on /dev/sda2, run: 
```
pvcreate --dataalignment 1m /dev/sda2
```
 --dataalignment Size[k|UNIT]
    Align the start of a PV data area with a multiple of this number. To see the location of the first Physical Extent (PE) of an existing PV, use pvs -o +pe_start. In addition, it may be shifted by an alignment offset, see --dataalignmentoffset. Also specify an appropriate PE size when creating a VG.
    
## Volume groups
### Creating a volume group
To create a VG volgroup0 with an associated PV /dev/sda2, run: 
```
# vgcreate volgroup0 /dev/sda2
``` 

## Logical volumes
### Creating a logical volume
To create a LV rootvol in a VG volgroup0 with 30 GiB of capacity, run: 
```
# lvcreate -L 30GB volgroup0 -n rootvol
```
and, to create a LV homevol in a VG volgroup0 with the rest of capacity, run: 
```
# lvcreate -l 100%FREE volgroup0 -n homevol
```
Activate the lvm
```
# modprobe dm-mod
```
You can check the VG MyVolGroup is created using the following command: 
```
# vgs
```
(you can use `vgscan` as well)

### Activating a volume group
```
# vgchange -a y volgroup0
```
or
```
# vgchange -ay
```
both give the same result.

## Format the partitions
format the EFI system partition to FAT32 using mkfs.fat. 
```
# mkfs.fat -F 32 /dev/sda1
```

to create an Ext4 file system on /dev/volgroup0/rootvol, run: 
```
# mkfs.ext4 /dev/volgroup0/rootvol
```
to create an Ext4 file system on /dev/volgroup0/homevol, run: 
```
# mkfs.ext4 /dev/volgroup0/homevol
```
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
Activate the swap file: 
```
# mkswap /dev/swap_partition
```
Finally, edit the fstab configuration to add an entry for the swap file: 
```
/etc/fstab

/swapfile none swap defaults 0 0

```


## Mount the file systems
Mount the root volume to /mnt. For example, if the root volume is /dev/rootvol: 
```
# mount /dev/volgroup0/rootvol /mnt
```
Mount the home volume to /mnt/home. For example, if the home volume is /dev/homevol:
BUT fisr we have to create the home directory:
```
# mkdir /mnt/home
```
then
```
# mount /dev/volgroup0/homevol /mnt/home
```
Create any remaining mount points (such as /mnt/efi) using mkdir and mount their corresponding volumes. 
```
# mkdir /boot/EFI
```
```
# mount /dev/sda1 /boot/EFI
```

## swap
```
# swapon /dev/swap_partition
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


## Installing Arch Linux
### Install essential packages


Use the pacstrap script to install the base package, Linux kernel and firmware for common hardware: 
```
# pacstrap /mnt base linux linux-firmware
```
> you can install linux-lts instead if linux or both
## Chroot
Change root into the new system: 
```
# arch-chroot /mnt
```
other essential packages
```
# pacman -S linux-headers
```
> you can install linux-lts-headers instead or both

a text editor
```
# pacman -S vim
```
```
pacman -S base-devel
```




others SOURCES :
https://youtu.be/DPLnBPM4DhI <br/>
https://wiki.archlinux.org/title/installation_guide#Pre-installation <br/>
https://wiki.archlinux.org/title/LVM <br/>
https://www.systutorials.com/docs/linux/man/8-pvcreate/
