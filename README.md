DISCLAIMER: No liability for the contents of this documents can be accepted. Use the concepts, examples and other content at your own risk. There may be errors and inaccuracies, that may of course be damaging to your system. Although this is highly unlikely, you should proceed with caution. The author does not accept any responsibility for any damage incurred.

I am Linux enthisiast with no system administration background. This guide was put together by myself following a number of documents, tutorials and recommendations. I would appreciate any kind of constructive feedback about the below procedure. You can use Issues section for that purpose.

This repository is an attempt to create a complete guide for Arch Linux operating system deployment on a mobile personal computer (laptop). 

This installation procedure heavily borrows from the following sources:
* [Arch Linux Installation guide (archlinux wiki)](https://wiki.archlinux.org/title/Installation_guide)
* [Archlinux on encrypted btrfs with systemd-boot and KDE (blog)](https://rich.grundy.io/blog/archlinux-on-encrypted-btrfs-with-systemd-boot-and-kde/)
* [Nice Micro (youtube)](https://www.youtube.com/channel/UC2bkdAPR47c7FkvwSXzzMzQ)
* [EF - Linux Made Simple (youtube)](https://www.youtube.com/c/EFLinuxMadeSimple)
* [Installing Arch on an encrypted BRTFS with EFI (blog)](https://gareth.com/index.php/2020/07/15/installing-arch-on-an-encrypted-btrfs-with-efi/)

# I. Requirements #

1. Data at rest encryption (data partitions and swap).
2. Battery saving features (hibernation to disk on lid close or button press).
3. Wireless connectivity (wifi, bluetooth, wan), if available.
4. Snapshot system (to easily restore to previous state in case of failure).
5. Periodic full backup to NAS when connected to LAN.
6. Graphical environment.
7. Should support both MBR and UEFI systems.
8. Should be a minimal install, include only essential packages required for each functionality.

# II. Assumptions #

1. To create an installation medium we will use Linux distribution with the following tools: GnuPG, wget, dd. For instructions to create installation medium from Windows or MacOS, please refer to Arch Linux installation guide.
2. USB flash drive (pendrive) is used as installation medium.
3. We are using the following Arch Linux image file: _**archlinux-2021.08.01-x86_64.iso**_ and corresponding _**archlinux-2021.08.01-x86_64.iso.sig**_

# III. Installation steps #

1. Acquire an installation image.
2. Verify an installation image signature.
3. Prepare an installation medium.
4. Boot the live environment.
5. Set keyboard layout and console font.
6. Set up the network.
7. Set up SSH access to the live environment.
8. Verify the boot mode.
9. Update the system clock.
10. Partition the disks.
11. Setup an encryption on the main partition.
12. Format disk partitions.
13. Create btrfs subvolumes.
14. Mount the subvolumes.
15. Select the mirrors.
16. Install essential packages with pacstrap.
17. Generate fstab.
18. Chroot into the new system.
19. Set a timezone.
20. Set locale.
21. Set up the network.
22. Set root password.
23. Set initramfs.
24. Install bootloader.
25. Boot into a newly installed system.

## 1. Acquire an installation image: ##
The updated list of mirrors can be found on [Arch Linux download page](https://archlinux.org/download). Download Arch Linux image (.iso) from preferred mirror, and the corresponding PGP signature file (.iso.sig) from Arch Linux download page directly.\
**`$ wget https://ftp.icm.edu.pl/pub/Linux/dist/archlinux/iso/2021.08.01/archlinux-2021.08.01-x86_64.iso`**\
**`$ wget https://archlinux.org/iso/2021.08.01/archlinux-2021.08.01-x86_64.iso.sig`**

## 2. Verify an installation image signature: ##
Execute the gpg command to verify .iso file agianst .iso.sig. Both files must be in the same directory. Make sure RSA key from the output matches PGP fingerprint provided on Arch Linux download website.\
**`$ gpg --keyserver-options auto-key-retrieve --verify archlinux-2021.08.01-x86_64.iso.sig`**\
`gpg: assuming signed data in 'archlinux-2021.08.01-x86_64.iso'`\
`gpg: Signature made Sun 01 Aug 2021 10:32:22 AM CEST`\
`gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC`\
`gpg: Can't check signature: No public key`

## 3. Prepare an installation medium: ##
Find out the name of the USB drive and make sure it is not mounted:\
**`$ lsblk`**\
`NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT`\
`sda           8:0    1 14.5G  0 disk`\
`├─sda1        8:1    1  711M  0 part /media/pi/ARCH_202107`\
`└─sda2        8:2    1   68M  0 part`\
`sdb         179:0    0 59.5G  0 disk`\
`├─sdb1      179:6    0   66M  0 part /boot`\
`└─sdb2      179:7    0 57.8G  0 part /`

In the above example, the USB drive is the _**sda**_ device. Make sure the device is not mounted.\
**`$ umount /dev/sda1`**

In case the device is not empty, wipe out the device prior to .iso copy.\
**`# wipefs --all /dev/sda`**

Copy Arch Linux install image to USB drive.\
**`# dd if=archlinux-2021.08.01-x86_64.iso of=/dev/sda bs=4M conv=fsync oflag=direct status=progress`**

## 4. Boot the live environment: ##
Boot laptop with the USB drive prepared in the previous step. Arch Linux installation images do not support Secure Boot. Consult Arch Linux installation guide for more details.

## 5. Set keyboard layout and console font: ##
By default console keymap is US. Available layouts can be listed with:\
**`# ls /usr/share/kbd/keymaps/**/*.map.gz`**

To modify the layout, append a corresponding file name to loadkeys. For example, for Polish programmers layout:\
**`# loadkeys pl`**

Set console font to support Polish special characters.\
**`# setfont lat2-16.psfu.gz`**

## 6. Set up the network access: ##
### Wired connection with DHCP ###
Connect network cable to the laptop. If DHCP server is available in the network, ethernet network interface will be configured automatically. Check the network interface configuration.\
**`# ip addr`**\
`2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel UP mode DEFAULT group default qlen 1000`\
`link/ether 08:00:27:8f:69:ff brd ff:ff:ff:ff:ff:ff`\
`inet 192.168.0.2/24 metric 100 brd 192.168.0.255 scope global dynamic enp0s3`\
`valid_lft 83738sec preferred_lft 83738sec`\
`inet6 fe80::a12:abcd:a12b:1234/64 scope link`\
`valid_lft forever preferred_lft forever`

Check for Internet access:\
**`# ping archlinux.org`**\
`PING archlinux.org (95.217.163.246) 56(84) bytes of data.`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=1 ttl=49 time=44.3 ms`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=49 time=44.3 ms`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=3 ttl=49 time=44.3 ms`

### Wired connection without DHCP ###
Connect network cable to the laptop. Check for available network interfaces:\
**`# ip link`**\
`1: lo <LOOPBACK,UP,LOWER_UP> mtu 65535 qdisc noqueue state UNKNOWN mode DEFAULT group qlen 1000`\
`link/loopacbk 00:00:00:00:00:00 brd 00:00:00:00:00:00`\
`2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel UP mode DEFAULT group default qlen 1000`\
`link/ether 08:00:27:8f:69:ff brd ff:ff:ff:ff:ff:ff`\

Add an IP address to an interface:\
**`# ip address add 192.168.0.2/24 broadcast + dev enp0s3`**

Add default route to access Internet:\
**`# ip route add default via 192.168.0.1/24 dev enp0s3`**

Check for Internet access:\
**`# ping archlinux.org`**\
`PING archlinux.org (95.217.163.246) 56(84) bytes of data.`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=1 ttl=49 time=44.3 ms`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=49 time=44.3 ms`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=3 ttl=49 time=44.3 ms`

## 7. Set up SSH access to the live environment: ##

## 8. Verify the boot mode: ##
We need verify that we are actually booted in UEFI mode. If the following command is executed without error that means we run UEFI, similarly to the below output.\
**`# mount | grep efi`**\
`efivarfs on /sys/firmware/efi/efivars type efivarfs (rw,nosuid,nodev,noexec,realtime)`

# Partitioning #

## UEFI ##
partition | type | size | file system
--------- | ---- | ---- | ----------- 
boot | ef00 | +550M | mkfs.fat -F32
OS | 8300 | ~ | mkfs.btrfs

For System integrity create subvolumes instead of directories for:
* /tmp
* /opt
* /srv
* /var/spool
* /var/log
* /var/run
* /var/tmp

# Packages #

categoty | official | AUR 
-------- | -------- | --------
Base System | base base-devel linux linux-firmware |
File System tools | dosfstools btrfs-progs |
CPU microcode | intel-ucode |
Network | dhcpcd wpa_supplicant | 
Utils | vim git |
Documentation | man-db man-pages texinfo |
Graphical environment | xf86-video-intel xorg-server xorg-xinit xorg-xsetroot |
Window Manager | | dwm
