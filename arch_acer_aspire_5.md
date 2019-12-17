# Installation Notes for Arch Linux on Acer Aspire 5

## Pre-Installation

### BIOS Settings 

Enable booting off a live USB:
1. Enter the bios using `F2`.
1. Set a BIOS password and disable Safe Mode. Note that you will not be able to disable this option without setting a password.
1. Change the boot order under the Boot menu.

Make the NVMe SSD visible during installation:
1. In order for the NVMe drive to be correctly mounted on `/dev`, you will need to change the SATA Mode to AHCI. Otherwise this
won't show up when you run `fdisk -l`, `lsblk`, etc.

## Installation

### Network Configuration

I have not figured out how to correctly configure network settings with the NetworkManager service. I'm working on this since
it should be considerably easier than the following steps, but this will allow you to use the internet on just a wireless connection in the meantime:
1. Identify the wireless port:
```
ip link
```
This has always been __wlan0__ during my previous installs, but be careful to check this. In addition, the port should
be listed as DOWN before you start the `netctl` service or you will fail to connect. `netctl start` will bring up the port
itself. If this is not already listed as down, you can bring it down with:
```
ip link set <PORT> down
```
1. Enter configuration for the wireless network:
```
cp /etc/netctl/examples/wireless-wpa /etc/netctl/<SSID>
```
Where <SSID> is the appropriate SSID. Edit `/etc/netctl/<SSID>`, inputting access point info and the port you identified earlier.
1. Start the netctl service for this particular access point:
```
netctl start <SSID>
```
1. Obtain an IP for the wireless port you identified earlier:
```
dhcpcd <PORT>
```
1. Ping some arbitrary site to ensure connectivity:
```
ping archlinux.org
```

After booting, you'll need access to the same services you used above in the event that you need to reconfigure the network
settings (you almost certainly will). Make sure to run the following sometime after `arch-chroot`ing to the installation:
```
pacman -S netctl dhcpcd wpa\_supplicant
```

### UEFI Setup

UEFI setup should be almost identical for other machines, but here's a complete list of the things you'll need to do:

Create a UEFI partition of ~300MB. This should be the first partition `/dev/nvme0n1p1`. When running `fdisk`:
```
n
<ENTER>
<ENTER>
+300M
t
<ENTER>
1
```

Format the filesystem to FAT-32:
```
mkfs.fat -F32 /dev/nvme0n1p1
```

You don't need to mount this partition until after you `arch-chroot`:
```
mount /dev/nvme0n1p1 /boot/EFI
```

You will probably need to install the following packages to the new system:
```
pacman -S efibootmgr dosfstools os-prober mtools
```

When setting up the boot loader (using GRUB):
```
grub-install --target=x86_64-efi --bootloader-id=grub_uefi
```

## Post Installation

Both times I've installed Arch on this system, the wireless port identification changed from __wlan0__ to __wlp2s0__ when
accessing it during install and post-install, respectively. My guess is that this changes based on the install medium but
I have not verified this. Basically, just repeat the Network Configuration steps post-installation. You'll need to do this
regardless, since the files you modified during installation were located on the installation medium. Just be mindful of
which ports show up when running `ip link`.

After making the necessary changes, run the following to have wireless connect to this access point every time the machine
boots:
```
netctl reenable <SSID>
```

You will also have to rerun `dhcpcd <PORT>` but this can probably be automated.
