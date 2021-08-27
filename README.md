Arch Linux OS installation and configuration procedures for laptop PC. 
This repo includes scripts, configuration files and step-by-step guides for 
Arch Linux deployment on a laptop (mobile) computer.

This installation procedure is based on the official [Arch Linux Installation guide](https://wiki.archlinux.org/title/Installation_guide).

# Requirements #

1. Data at rest encryption (disk partitions and swap).
2. Battery saving features (hibernation to disk on lid close).
3. Wireless connectivity (wifi, bluetooth, wan), if available.
4. Snapshot system (to easily restore to previous state in case of failure).
5. Periodic full backup to NAS when connected to LAN.

# Installation steps #

1. Acquire installation image
2. Prepare installation media
3. Create disk partitions
4. Format disk partitions

# Partitioning #

## UEFI ##
partition | mount point	| size | file system
--------- | ----------- | ---- | ----------- 
/boot | ef00 | +550M | mkfs.fat -F32
/ | 8300 | +30G | mkfs.btrfs

# Packages #

## Base ##

base linux linux-firmware
dosfstools btrfs-progs
intel-ucode
dhcpcd wpa_supplicant
vim
man-db man-pages texinfo

## Optional ##

networkmanager (replaces: dhcpcd wpa_supplicant)
