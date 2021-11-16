# Archlinux-Installation-guide
Archlinux Installation guide UEFI LVM Swapfile (NO Encryption)

## Set the keyboard layout

The default console keymap is US. Set the azerty layout with:

```
# loadkeys fr-latin1
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
