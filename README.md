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

##Partition the disks
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
|   /boot/efi     |   /dev/sda1   |   EFI system partition   |      +500M         |
|                 |   /dev/sda2   |   Linux LVM              |       100%         |
