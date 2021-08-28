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

# Installation steps #

1. Acquire installation image
2. Prepare installation media
3. Create disk partitions
4. Format disk partitions

# Partitioning #

## UEFI ##
partition | type | size | file system
--------- | ----------- | ---- | ----------- 
/boot | ef00 | +550M | mkfs.fat -F32
/ | 8300 | +30G | mkfs.btrfs

For System integrity create subvolumes instead of directories for:
* /tmp
* /opt
* /srv
* /var/spool
* /var/log
* /var/run
* /var/tmp

# Packages #

categoty | default | optional
-------- | -------- | --------
Base System | base base-devel linux linux-firmware |
File System tools | dosfstools btrfs-progs |
CPU microcode | intel-ucode |
Network | dhcpcd wpa_supplicant | networkmanager (replaces: dhcpcd wpa_supplicant)
Utils | vim git |
Documentation | man-db man-pages texinfo |
Graphical environment | xf86-video-intel xorg-server xorg-xinit xorg-xsetroot |
