DISCLAIMER: No liability for the contents of this documents can be accepted. Use the concepts, examples and other content at your own risk. There may be errors and inaccuracies, that may of course be damaging to your system. Although this is highly unlikely, you should proceed with caution. The author does not accept any responsibility for any damage incurred.

This repository is an attempt to create a complete guide for Arch Linux operating system deployment on a mobile personal computer (laptop). The installation will fulfill all the requirements listed below.

This installation procedure heavily borrows from the following sources:
* [Arch Linux Installation guide](https://wiki.archlinux.org/title/Installation_guide)
* [Rich Grundy - Archlinux on encrypted btrfs with systemd-boot and KDE](https://rich.grundy.io/blog/archlinux-on-encrypted-btrfs-with-systemd-boot-and-kde/)
* [Nice Micro channel](https://odysee.com/@nicemicro:6?)
* [EF - Linux Made Simple](https://www.youtube.com/c/EFLinuxMadeSimple)

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

1. To create an installation media we will use Linux distribution with the following tools: GnuPG, wget, dd.
2. USB flash drive (pendrive) is used as installation media.
3. We are using the following Arch Linux image file: _**archlinux-2021.08.01-x86_64.iso**_ and corresponding _**archlinux-2021.08.01-x86_64.iso.sig**_

# III. Installation steps #

1. Acquire an installation image.
2. Verify an installation image signature.
3. Prepare an installation media.
4. Boot the live environment.
5. Set keyboard layout.
6. Verify the boot mode.
7. Connect to the Internet.
8. Update the system clock.
9. Partition the disks.
10. Setup an encryption on the main partition.
11. Format disk partitions.
12. Create btrfs subvolumes.
13. Mount the subvolumes.
14. Select the mirrors.
15. Install essential packages with pacstrap.
16. Generate fstab.
17. Chroot into the new system.
18. Set a timezone.
19. Set locale.
20. Set up the network.
21. Set root password.
22. Set initramfs.
23. Install bootloader (systemd-boot).
24. Boot into a newly installed system.

## 1. Acquire an installation image: ##
The updated list of mirrors can be found on [Arch Linux download page](https://archlinux.org/download). Download Arch Linux image (.iso) from preferred mirror, and the corresponding PGP signature file (.iso.sig) from Arch Linux download page directly.\
`$ wget https://ftp.icm.edu.pl/pub/Linux/dist/archlinux/iso/2021.08.01/archlinux-2021.08.01-x86_64.iso`\
`$ wget https://archlinux.org/iso/2021.08.01/archlinux-2021.08.01-x86_64.iso.sig`

## 2. Verify an installation image signature: ##
Execute the gpg command to verify .iso file agianst .iso.sig. Both files must be in the same directory. Make sure RSA key from the output matches PGP fingerprint provided on Arch Linux download website.\
`$ gpg --keyserver-options auto-key-retrieve --verify archlinux-2021.08.01-x86_64.iso.sig`
`gpg: assuming signed data in 'archlinux-2021.08.01-x86_64.iso'`\
`gpg: Signature made Sun 01 Aug 2021 10:32:22 AM CEST`\
`gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC`\
`gpg: Can't check signature: No public key`

## 3. Prepare an installation media: ##
Find out the name of the USB drive and make sure it is not mounted:\
`$ lsblk`\
`NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT`\
`sda           8:0    1 14.5G  0 disk`\
`├─sda1        8:1    1  711M  0 part /media/pi/ARCH_202107`\
`└─sda2        8:2    1   68M  0 part`\
`sdb         179:0    0 59.5G  0 disk`\
`├─sdb1      179:6    0   66M  0 part /boot`\
`└─sdb2      179:7    0 57.8G  0 part /`

In the above example, the USB drive is the _**sda**_ device. Make sure the device is not mounted.\
`$ umount /dev/sda1`

In case the device is not empty, wipe out the device prior to .iso copy.\
`# wipefs --all /dev/sda`

Copy Arch Linux install image to USB drive.\
`# dd if=archlinux-2021.08.01-x86_64.iso of=/dev/sda bs=4M conv=fsync oflag=direct status=progress`

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
