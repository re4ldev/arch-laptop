This repository is an attempt to create a complete guide for Arch Linux operating system deployment on a mobile personal computer (laptop). The installation will fulfill all the requirements listed below.

This installation procedure heavily borrows from the following sources:
* [Arch Linux Installation guide](https://wiki.archlinux.org/title/Installation_guide)
* [Rich Grundy - Archlinux on encrypted btrfs with systemd-boot and KDE](https://rich.grundy.io/blog/archlinux-on-encrypted-btrfs-with-systemd-boot-and-kde/)
* [Nice Micro channel](https://odysee.com/@nicemicro:6?)
* [EF - Linux Made Simple](https://www.youtube.com/c/EFLinuxMadeSimple)

# Requirements #

1. Data at rest encryption (disk partitions and swap).
2. Battery saving features (hibernation to disk on lid close or button press).
3. Wireless connectivity (wifi, bluetooth, wan), if available.
4. Snapshot system (to easily restore to previous state in case of failure).
5. Periodic full backup to NAS when connected to LAN.
6. Graphical environment.
7. Should support both MBR and UEFI systems.
8. Should be a minimal install, include only essential packages required for each functionality.

# Assumptions #

# Installation steps #
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

# Partitioning #

## UEFI ##
partition | type | size | file system
--------- | ---- | ---- | ----------- 
boot | ef00 | +550M | mkfs.fat -F32
OS | 8300 | +30G | mkfs.btrfs

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
