IMPORTANT: This **Guide** is still under development.

# 0. Introduction #

This repository is an attempt to create a complete **Guide** for [Arch Linux](https://archlinux.org/) operating system deployment on a mobile personal computer a.k.a laptop. The solution should satisfy the list of requirements collected in section [I.Requirements](#i-requirements), following a number of assumptions listed in section [II.Assumptions](#ii-assumptions).

I am Linux enthusiast with no system administration background. This **Guide** was put together by myself following a number of documents, guides and tutorials available online which I list below in the section [A.Sources](#a-sources). Hopefully this is a living document, updated frequently, to catch up with the latest developments.

I would appreciate all kind of constructive feedback about the respository, described procedure steps, applied solutions, and used software. You can use [Issues](../../issues) section for that purpose. Feel free to share this **Guide** with larger audience and open for discussion. All kind of best practices and recommendations are more than welcome.

**DISCLAIMER**: No liability for the contents of this documents can be accepted. Use the concepts, examples and other content at your own risk. There may be errors and inaccuracies, that may of course be damaging to your system. Although this is highly unlikely, you should proceed with caution. The author does not accept any responsibility for any damage incurred.

**IMPORTANT**: If applied incorrectly, some of the steps described in the **Guide** may irreversibly erase data from, and damage your computer. Proceed with caution.

# A. Sources #
This installation procedure heavily borrows from the following sources:
* [Arch Linux Wiki](https://wiki.archlinux.org/)
* [Arch Linux Installation guide (archlinux wiki)](https://wiki.archlinux.org/title/Installation_guide)
* [Archlinux on encrypted btrfs with systemd-boot and KDE (blog)](https://rich.grundy.io/blog/archlinux-on-encrypted-btrfs-with-systemd-boot-and-kde/)
* [Nice Micro (youtube channel)](https://www.youtube.com/channel/UC2bkdAPR47c7FkvwSXzzMzQ)
* [EF - Linux Made Simple (youtube channel)](https://www.youtube.com/c/EFLinuxMadeSimple)
* [Luke Smith (youtube channel)](https://www.youtube.com/c/LukeSmithxyz)
* [Installing Arch on an encrypted BRTFS with EFI (blog)](https://gareth.com/index.php/2020/07/15/installing-arch-on-an-encrypted-btrfs-with-efi/)
* [Jordan Williams (blog)](https://www.jwillikers.com/btrfs-layout)
* [OpenSUSE Wiki](https://en.opensuse.org/SDB:BTRFS#Default_Subvolumes)
* [LinuxFoundation - Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html)

# I. Requirements #

id | Requirement | Rationale | Solution
-- | ----------- | --------- | --------
1 | Data-at-rest encryption | If the device is lost, the data and swap area can not be easily accessible. | LUKS encryption for data partition and swapfile on the encrypted data partition
2 | Battery saving | Mobile device should work without grid connection for longer periods of time. | TLP and hibernation to disk on lid close or button press
3 | Wireless connectivity | Connection to LAN or WAN should be supported without wired connectivity in case only wireless access points are available in the environment. | Network Manager
4 | Snapshot system | To reduce the risk of failure the snapshot system must be available. Atomic snapshots should be taken prior to system upgrades or on-demand. | ~~BTRFS file system and Snapper~~
5 | Periodic backup to NAS | When connected to home LAN full system/data backup to NAS should be available | rsync
6 | Graphical environment | Graphical environment must be available to support graphical content (ex. X window applications, Internet browsing) | Suckless DWM
7 | Minimal installation | The solution should provide only the bare minimum number of packages to support required functionalites. | Arch Linux OS
8 | Rolling release Linux distro | The latest available software versions should be applied as soon as possible to keep up with the latest developments and security patches. | Arch Linux OS
9 | Hybrid boot | In case the hard drive containing the OS must be connected to a different hardware. Must be able to boot in UEFI and legacy BIOS modes. | GRUB bootloader

# II. Assumptions #

1. To create an installation medium we will use Linux distribution with the following tools: GnuPG, wget, dd. For instructions to create installation medium from Windows or MacOS, please refer to Arch Linux installation guide.
2. USB flash drive a.k.a pendrive is used as installation medium.
3. We are using the following Arch Linux image file: _**archlinux-2021.10.01-x86_64.iso**_ and corresponding _**archlinux-2021.10.01-x86_64.iso.sig**_
4. Arch Linux installation process requires Internet access. There is a LAN network with Internet access available.
5. Arch Linux live environment will be accessed via SSH to perform the installation procedure.
6. All storage devices are SSD SATA.

# III. Installation steps index #

1. [Acquire and verify an installation image.](#1-acquire-and-verify-an-installation-image)
2. [Prepare an installation medium.](#2-prepare-an-installation-medium)
3. [Boot the live environment, and configure console keymap and font.](#3-boot-the-live-environment-and-configure-console-keymap-and-font)
4. [Live environment network access setup.](#4-live-environment-network-access-setup)
5. [Live environment SSH access setup.](#5-live-environment-ssh-access-setup)
6. [Live environment system clock update.](#6-live-environment-system-clock-update)
7. [Live environment boot mode verification.](#7-live-environment-boot-mode-verification)
8. [Wipe out the disk.](#8-wipe-out-the-disk)
9. [Partition the disks.](#9-partition-the-disks)
10. [Setup an encryption and format the partitions.](#10-setup-an-encryption-and-format-the-partitions)
11. [Mount partitions for the System and Boot.](#11-mount-partitions-for-the-system-and-boot)
12. [Create a swap area: swapfile.](#12-create-a-swap-area-swapfile)
13. [Select the mirrors.](#13-update-the-mirror-list)
14. [Install system packages with pacstrap.](#14-install-system-packages-with-pacstrap)
15. [Generate fstab.](#15-generate-fstab)
16. [Chroot into the new system and perform basic configuration.](#16-chroot-into-the-new-system-and-perform-basic-configuration)
17. [Update mkinitcpio configuration and generate initramfs.](#17-update-mkinitcpio-configuration-and-generate-initramfs)
18. [Install and configure the boot loader.](#18-install-and-configure-the-boot-loader)
19. [Boot into a newly installed system.](#19-boot-into-a-newly-installed-system)
20. [Install Graphical Environment.](#20-install-graphical-environment)
21. [Configure backup to NAS and perform initial full backup.](#21-configure-backup-to-nas-and-perform-initial-full-backup)

## 1. Acquire and verify an installation image ##
The updated list of mirrors can be found on [Arch Linux download page](https://archlinux.org/download). Download Arch Linux image (.iso) from preferred mirror, and the corresponding PGP signature file (.iso.sig) from Arch Linux download page directly.\
**`$ wget http://ftp.icm.edu.pl/pub/Linux/dist/archlinux/iso/2021.10.01/archlinux-2021.10.01-x86_64.iso`**\
**`$ wget https://archlinux.org/iso/2021.10.01/archlinux-2021.10.01-x86_64.iso.sig`**

Execute the gpg command to verify .iso file agianst .iso.sig. Both files must be in the same directory. Make sure RSA key from the output matches PGP fingerprint provided on Arch Linux download website.\
**`$ gpg --verify archlinux-2021.10.01-x86_64.iso.sig`**
>`gpg: assuming signed data in 'archlinux-2021.10.01-x86_64.iso'`\
>`gpg: Signature made Fri 01 Oct 2021 10:32:22 AM CEST`\
>`gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC`\
>`gpg: Can't check signature: No public key`

## 2. Prepare an installation medium ##
**IMPORTANT**: Make sure you are applying the changes to the USB drive. After this step the data will be permanently erased from the device on which you apply the commands.

Find out the name of the USB drive device. In this example, the USB drive is the _**sda**_ device. \
**`$ lsblk -o +VENDOR,MODEL`**
>`NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT            VENDOR   MODEL`\
>`sda           8:0    1 14.5G  0 disk                       SanDisk  Ultra`\
>`├─sda1        8:1    1  711M  0 part /media/pi/ARCH_202107`\
>`└─sda2        8:2    1   68M  0 part`\
>`sdb         179:0    0 59.5G  0 disk`\
>`├─sdb1      179:6    0   66M  0 part /boot`\
>`└─sdb2      179:7    0 57.8G  0 part /`

Make sure the device is not mounted. Make sure to replace _**sdXY**_ with the corresponding partition name specific to your deployment.\
**`$ umount /dev/sdXY`**

In case the device is not empty, wipe out the device prior to .iso copy. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# wipefs --all /dev/sdX`**

Copy Arch Linux install image to USB drive. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# dd if=archlinux-2021.10.01-x86_64.iso of=/dev/sdX bs=4M conv=fsync oflag=direct status=progress`**

## 3. Boot the live environment, and configure console keymap and font ##
Boot laptop with the USB drive prepared in the previous step. Arch Linux installation images do not support Secure Boot. If you are having trouble booting from the USB, please, consult [Arch Linux installation guide](https://wiki.archlinux.org/title/Installation_guide) for more details.

By default console keymap is US. List the directory with available keyboard layouts.\
**`# ls /usr/share/kbd/keymaps/**/*.map.gz`**

To modify the layout, append a corresponding file name without extension (.map.gz) to loadkeys. Make sure to replace _**layout_name**_ with your preferred layout.\
**`# loadkeys layout_name`**

List the directory with available console fonts.\
**`# ls /usr/share/kbd/consolefonts/**/*.psfu.gz`**

To modify the console font to support special characters, append a font name without extension (.psfu.gz) to setfont. Make sure to replace _**font_name**_ with your preferred font.\
**`# setfont font_name`**

## 4. Live environment network access setup ##
Since network connectivity is a critical component for successful Arch Linux installation we will describe four different network access scenarios:
1. Wired connection with DHCP
2. Wired connection without DHCP
3. Wireless connection with DHCP
4. Wireless connection without DHCP

(TODO: Include WWAN configuration procedure)

Make sure the wireless network adapter is not blocked.\
**`# rfkill list`**
>`0: phy0: Wireless LAN`\
>`Soft blocked: no`\
>`Hard blocked: no`

If the wireless network adapter is hard-blocked then use the hardware button (switch) to unblock it. If it is not hard-blocked but soft-blocked then unblock it with rfkill.\
**`# rfkill unblock wifi`**

Check for available network interfaces, and apply one of the above scenarios to the preferred interface.\
**`# ip link`**
>`1: lo <LOOPBACK,UP,LOWER_UP> mtu 65535 qdisc noqueue state UNKNOWN mode DEFAULT group qlen 1000`\
>`link/loopacbk 00:00:00:00:00:00 brd 00:00:00:00:00:00`\
>`2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel UP mode DEFAULT group default qlen 1000`\
>`link/ether 08:00:27:8f:69:ff brd ff:ff:ff:ff:ff:ff`\
>`3: wlan0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DORMANT group default qlen 1000`\
>`link/ether 08:00:27:aa:bb:cc brd ff:ff:ff:ff:ff:ff` 

Once applied the procedure for any of the DHCP scenarios, you should verify local IP address.\
**`# ip addr`**
>`2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel UP mode DEFAULT group default qlen 1000`\
>`link/ether 08:00:27:8f:69:ff brd ff:ff:ff:ff:ff:ff`\
>`inet 10.0.0.2/24 metric 100 brd 10.0.0.255 scope global dynamic enp0s3`\
>`valid_lft 83738sec preferred_lft 83738sec`\
>`inet6 fe80::a12:abcd:a12b:1234/64 scope link`\
>`valid_lft forever preferred_lft forever`

Once applied the procedure from one of the above scenarios, you should verify Internet access.\
**`# ping archlinux.org`**
>`PING archlinux.org (95.217.163.246) 56(84) bytes of data.`\
>`64 bytes from archlinux.org (95.217.163.246): icmp_seq=1 ttl=49 time=44.3 ms`\
>`64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=49 time=44.3 ms`\
>`64 bytes from archlinux.org (95.217.163.246): icmp_seq=3 ttl=49 time=44.3 ms`

### 4-1. Wired connection with DHCP ###
Connect network cable to the laptop. If DHCP server is available in the network, ethernet network interface will be configured automatically.

### 4-2. Wired connection without DHCP ###
Add an IP address to the interface. Make sure to replace _**ip_address/net_mask**_ and _**interface_name**_ with the respective values specific to your deployment.\
**`# ip address add ip_address/net_mask broadcast + dev interface_name`**

Add default route to access Internet. Make sure to replace _**gateway_address/net_mask**_ and _**interface_name**_ with the respective values specific to your deployment.\
**`# ip route add default via gateway_address/net_mask dev interface_name`**

### 4-3. Wireless connection with DHCP ###
Connect to wireless LAN. Make sure to replace _**your_wifi_password**_, _**wireless_interface**_, and _**your_ssid**_ with the respective values specific to your deployment.\
**`# iwctl --passphrase your_wifi_password station wireless_interface connect your_ssid`**

### 4-4. Wireless connection without DHCP ###
Connect to wireless LAN. Make sure to replace _**your_wifi_password**_, _**wireless_interface**_, and _**your_ssid**_ with the respective values specific to your deployment.\
**`# iwctl --passphrase your_wifi_password station wireless_interface connect your_ssid`**

Add an IP address to the interface. Make sure to replace _**ip_address/net_mask**_ and _**interface_name**_ with the respective values specific to your deployment.\
**`# ip address add ip_address/net_mask broadcast + dev interface_name`**

Add default route to access Internet. Make sure to replace _**gateway_address/net_mask**_ and _**interface_name**_ with the respective values specific to your deployment.\
**`# ip route add default via gateway_address/net_mask dev interface_name`**

## 5. Live environment SSH access setup ##
To be able to login via ssh we need to set a root password.\
**`# passwd root`**

If ssh service is already enabled/running you can try to connect via ssh. Check if ssh service is already running.\
**`# systemctl list-unit-files -t service | grep ssh`**
>`sshd.service enabled disabled`

If ssh is not running start the service.\
**`# systemctl start sshd`**

You can now connect via ssh to the live environment from another machine.

## 6. Live environment system clock update ##
Use NTP server to syncronize time.\
**`# timedatectl set-ntp true`**

Set your timezone. Make sure to use your respective _**Region**_ and _**City**_ instead.\
**`# timedatectl set-timezone Region/City`**

Verify the clock is correctly configured.\
**`# timedatectl`**

## 7. Live environment boot mode verification ##
We need verify that we are actually booted in UEFI mode. If the following command is executed without error that means we run UEFI, similarly to the below output.\
**`# mount | grep efi`**
>`efivarfs on /sys/firmware/efi/efivars type efivarfs (rw,nosuid,nodev,noexec,realtime)`

## 8. Wipe out the disk ##
**IMPORTANT**: If you follow the instructions included in this step, the data on the disk will be erased. If you apply incorrectly the following steps you may erase the data from your computer irreversibly. Proceed with caution.

This step is optional but recommended. It ensures that random data is written to the disk prior to the encryption making it almost impossible to distinguish the free space from the occupied one.

Verify the disk device name on which we will install Arch Linux. In this example, we will use _**sda**_ device.\
**`# lsblk -o +VENDOR,MODEL`**
>`NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS      VENDOR   MODEL`\
>`loop0 7:0 0 662.1M 1 loop /run/archiso/airootfs`\
>`sda 8:0 0 20G 0 disk                       	ATA      VBOX HARDDISK`\
>`sr0 11:0 1 831.3M 0 rom /run/archiso/bootmnt  VBOX     VBOX CD-ROM`
 
Check if the drive security is frozen. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`hdparm -I /dev/sdX | grep frozen`**
>`frozen`

If it says _frozen_ it is not possible to proceed with the next step. To unfreeze the drive you may try to suspend the system. Upon waking up, it is likely that the freeze will be lifted. If this trick does not work for you, please, follow up on Arch Linux wiki [SSD/Memory cell cleaning](https://wiki.archlinux.org/title/Solid_state_drive/Memory_cell_clearing).\
**`# systemctl suspend`**

If you are performing the installation via ssh, you have now lost your connection. Resume the device from suspension and reconnect via ssh to proceed.

Check if the drive security is still frozen. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# hdparm -I /dev/sdX | grep frozen`**
>`not frozen`

**IMPORTANT**: Do not reboot after applying this step.\
Enable security by setting a user password. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# hdparm --user-master u --security-set-pass PasSWorD /dev/sdX`**

As a sanity check, let's verify that security is enabled. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# hdparm -I /dev/sdX`**
>`Security:`\
>`        Master password revision code = 65534`\
>`                supported`\
>`                enabled`\
>`        not     locked`\
>`        not     frozen`\
>`        not     expired: security count`\
>`                supported: enhanced erase`\
>`        Security level high`\
>`        2min for SECURITY ERASE UNIT. 2min for ENHANCED SECURITY ERASE UNIT.`

**IMPORTANT**: After this command the SSD will be wiped and data will be lost.\
**IMPORTANT**: Ensure that the drive is not mounted.\
Issue the ATA Secure Erase command. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# hdparm --user-master u --security-erase PasSWorD /dev/sdX`**

The drive is now erased, let's verify that security is disabled. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# hdparm -I /dev/sdX`**
>`Security:`\
>`        Master password revision code = 65534`\
>`                supported`\
>`        not     enabled`\
>`        not     locked`\
>`        not     frozen`\
>`        not     expired: security count`\
>`                supported: enhanced erase`\
>`        Security level high`\
>`        2min for SECURITY ERASE UNIT. 2min for ENHANCED SECURITY ERASE UNIT.`

Fill the disk with random bytes stream. Depending on the drive size, this step will take an extended period of time. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# shred -v -n 1 /dev/sdX`**

## 9. Partition the disks ##
**IMPORTANT**: If you follow the instructions included in this step, the data on the disk will be erased. If you apply incorrectly the following steps you may erase the data from your computer irreversibly. Proceed with caution.

We will create three partitions. One for the legacy BIOS, second one for the UEFI boot, and the last one for the System. We will not use swap partition, instead we will use swapfile on subvolume.

partition | parted type | size | file system
--------- | ---- | ---- | ----------- 
BIOS | boot_grub | 1MiB | N/A
ESP | esp | 550MiB | fat32
SYSTEM | linux | ~ | ext4

In case you skipped the previous step, where we wiped out the disk, you need to verify the disk device name on which we will install Arch Linux. In this example, we will use _**sda**_ device.\
**`# lsblk -o +VENDOR,MODEL`**
>`NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS      VENDOR   MODEL`\
>`loop0 7:0 0 662.1M 1 loop /run/archiso/airootfs`\
>`sda 8:0 0 20G 0 disk                       	ATA      VBOX HARDDISK`\
>`sr0 11:0 1 831.3M 0 rom /run/archiso/bootmnt  VBOX     VBOX CD-ROM`

We will use GNU parted to partition the disk.\
First create GUID Partition Table. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# parted -s /dev/sdX mklabel gpt`**

Make partition for BIOS boot. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# parted -s -a minimal /dev/sdX mkpart BIOS 0G 1MiB set 1 bios_grub on`**

Make partition for UEFI boot. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# parted -s /dev/sdX mkpart ESP fat32 1MiB 551MiB set 2 esp on`**

Make partition for the system. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# parted -s /dev/sdX mkpart SYSTEM ext4 551MiB 100%`**

Verify the partitions are correcrtly created. In this example drive device is _**sda**_.\
**`# lsblk -o +PARTLABEL`**
>`NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS           PARTLABEL`\
>`loop0    7:0    0 673.1M  1 loop /run/archiso/airootfs`\
>`sda      8:0    0    20G  0 disk`\
>`├─sda1   8:1    0  1007K  0 part                       BIOS`\
>`├─sda2   8:2    0   550M  0 part                       ESP`\
>`└─sda3   8:3    0  19.5G  0 part                       SYSTEM`\
>`sr0     11:0    1 846.3M  0 rom  /run/archiso/bootmnt`

## 10. Setup an encryption and format the partitions ##
We will encrypt SYSTEM partition with LUKS.

Setup encryption on the SYSTEM partition and open the encrypted partition to work with. Make sure to replace _**sdXY**_ with the corresponding SYSTEM partition name specific to your deployment.\
**`# cryptsetup luksFormat /dev/sdXY`**\
**`# cryptsetup luksOpen /dev/sdXY cryptroot`**

Format SYSTEM and EFI partition with its respective file systems.

ESP partition requires FAT32 file system. Make sure to replace _**sdXY**_ with the corresponding ESP partition name specific to your deployment.\
**`# mkfs.fat -F32 /dev/sdXY`**

SYSTEM partition will use btrfs file system.\
**`# mkfs.ext4 /dev/mapper/cryptroot`**

## 11. Mount partitions for the System and Boot. ##
Mount SYSTEM partition to /mnt.\
**`# mount -o noatime,commit=60 /dev/mapper/cryptroot /mnt`**

Create the directory where ESP partition will be mounted.\
**`# mkdir /mnt/boot`**

Mount ESP partition. Make sure to replace _**sdXY**_ with the corresponding partition name specific to your deployment.\
**`# mount /dev/sdXY /mnt/boot`**

## 12. Create a swap area: swapfile ##
To be able to hibarnate the system on button press or lid close we will need to create a swap area. In the previous steps we have created @swap volume to make sure it is not part of the snapshots, and mounted it to the /swapspace directory.

Create a directory for a swap file.\
**`# mkdir /mnt/swapspace`**

Create a zero length file.\
**`# truncate -s 0 /mnt/swapspace/swapfile`**

Set the swap file size. Swap file size should be two times the size of available RAM.\
**`# fallocate -l 8G /mnt/swapspace/swapfile`**

Format the swap file to Linux swap.\
**`# mkswap /mnt/swapspace/swapfile`**

Set the right permissions.\
**`# chmod 600 /mnt/swapspace/swapfile`**

Activate swap file.\
**`# swapon /mnt/swapspace/swapfile`**

Verify swap area is on.\
**`# free -m`**
>`               total        used        free      shared  buff/cache   available`\
>`Mem:            3913         133        3241         165         538        3403`\
>`Swap:           8191           0        8191`

## 13. Update the mirror list ##
Arch Linux installation is performed via network. The packages are downloaded from the mirrors.

We will use _reflector_ - python script to update pacman mirror list. Sort the 20 most recently synchronized mirrors accessible via https.\
**`# reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist`**

We can verify the selected mirrors viewing /etc/pacmand.d/mirrorlist file.\
**`# vim /etc/pacman.d/mirrorlist`**

## 14. Install system packages with pacstrap ##
_pacstrap_ script is used to install the selected system packages and copy the mirror list established in the previous step into a hard drive.

NOTE: Make sure to chose video driver for your particular hardware. More details can be found on Arch Linux Wiki [Xorg: Driver Installation](https://wiki.archlinux.org/title/Xorg#Driver_installation)

Identify the video card.\
**`# lspci -v | grep -A1 -e VGA -e 3D`**
>`00:02.0 VGA compatible controller: VMware SVGA II Adapter (prog-if 00 [VGA controller])`\
>`		  Subsystem: VMware SVGA II Adapter`\
>`		  Flags: bus master, fast devsel, latency 64, IRQ 18`

Depending on your system hardware include the video drivers packages in the following _pacstrap_ command. In this example we will use _xf86-video-intel_ package.

This is a complete list of packages that will be installed to satisfy the system requirements from section [I.Requirements](#i-requirements).
 
categoty | official | AUR 
-------- | -------- | --------
Base System | base base-devel linux linux-firmware |
File System tools | dosfstools ef2sprogs |
CPU microcode | intel-ucode |
Network | dhcpcd wpa_supplicant networkmanager | 
Bootloader | grub efibootmgr | 
Utils | neovim git openssh parted wget rsync |
Documentation | man-db man-pages texinfo |
Graphical environment | xorg-server xorg-xinit xorg-xsetroot ttf-dejavu |
Video drivers | xf86-video-intel |
Window Manager | | dwm

Use _pacstrap_ to install Arch Linux on the hard drive.\
**`# pacstrap /mnt base base-devel linux linux-firmware dosfstools e2fsprogs intel-ucode dhcpcd wpa_supplicant networkmanager grub efibootmgr neovim git openssh parted wget rsync man-db man-pages texinfo xorg-server xorg-xinit xorg-xsetroot ttf-dejavu xf86-video-intel`**

## 15. Generate fstab ##
Generate fstab file using UUIDs.\
**`# genfstab -p -U /mnt >> /mnt/etc/fstab`**

Verify the correct entries in fstab file. Since we have specified mount points in step 11 they should now be included in the fstab file. However, the mount options need to be revisited since btrfs is not applying mount options on subvolume level. Example of fstab file is in the below output.\
**`# vim /mnt/etc/fstab`**
>`# Static information about the filesystems.`\
>`# See fstab(5) for details.`\
>`# <file system> <dir> <type> <options> <dump> <pass>`\
>`UUID=d72f6385-bb67-4ce2-810c-8eb8935402a2 / ext4 rw,noatime,commit=60 0 1`\
>`# /dev/sda2`\
>`UUID=8D9C-13EE /boot vfat rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro 0 2`\
>`/swapspace/swapfile none swap defaults 0 0`

Make note for /dev/mapper/cryptroot UUID we will need it later in the process. In this case it is _**d72f6385-bb67-4ce2-810c-8eb8935402a2**_

## 16. Chroot into the new system and perform basic configuration ##
Change root into the new system using Arch Linux provided tool.\
**`# arch-chroot /mnt`**

Setup root password.\
**`# passwd`**

Setup your timezone. Make sure to replace Region/City with your respective values.\
**`# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`**

Generate /etc/adjtime\
**`# hwclock --systohc`**

Enable NTP time syncronization.\
**`# timedatectl set-ntp true`**

Edit locale.gen and uncomment the locales required for the system.\
**`# nvim /etc/locale.gen`**

Generate the locales.\
**`# locale-gen`**

Create locale.conf and set the LANG variable accordingly. Instead of _**en_US.UTF-8**_ make sure to use a correct entry from /etc/locale.gen file.\
**`# echo "LANG=en_US.UTF-8" > /etc/locale.conf`**

Create vconsole.conf and set KEYMAP and FONT values.\
**`# nvim /etc/vconsole.conf`**
>`KEYMAP=pl`\
>`FONT=Lat2-Terminus16`

Setup the hostname. Make sure to replace _**myHostname**_ with the name of your choice.\
**`# echo "myHostname" > /etc/hostname`**

Add default entries to hosts file. Make sure to replace _**myHostname**_ with the name of your choice.\
**`# nvim /etc/hosts`**
>`127.0.0.1 localhost`\
>`::1 localhost`\
>`127.0.1.1 myHostname.localdomain myHostname`

Add user and include it to _wheel_ group for sudo access. Make sure th change _**UserName**_ to a correct user name.\
**`# useradd -mG wheel UserName`**

Set the user password. Make sure th change _**UserName**_ to a correct user name.\
**`# passwd UserName`**

Grant your user admin privilages via sudo. Uncomment a respective line in sudoers file.\
**`# EDITOR=nvim visudo`**
>`## Same thing without a password`\
>`%wheel ALL=(ALL) NOPASSWD: ALL`

Enable network manager.\
**`# systemctl enable NetworkManager.service`**

Enable sshd service.\
**`# systemctl enable sshd.service`**

## 17. Update mkinitcpio configuration and generate initramfs ##
Since we have added to our system some features that are not default, we have to include additional hook and binary in our initramfs.

Update mkinitcpio.conf\
**`# nvim /etc/mkinitcpio.conf`**

Update HOOKS.
>`HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt filesystems resume fsck)`

Generate initramfs.\
**`# mkinitcpio -P`**

## 18. Install and configure the boot loader ##
We will use GRUB as boot loader for our installation. Mainly because it supports both legacy BIOS and UEFI boot modes.

Install GRUB for legacy BIOS. Make sure to replace _**sdX**_ with the corresponding device name specific to your deployment.\
**`# grub-install --target=i386-pc /dev/sdX`**

Install GRUB for UEFI.\
**`# grub-install --target=x86_64-efi --efi-directory=/boot --boot-directory=/boot --bootloader-id=GRUB`**

Add resume entries to GRUB configuration file.

To be able to resume from hibernation we need to calculate the resume_offset number. As recommended in Arch Linux Wiki we will use [tool btrfs_map_physical.c](https://github.com/osandov/osandov-linux/blob/master/scripts/btrfs_map_physical.c) to compute resume_offset.\

Download and compile the tool.\
**`# mkdir -p ~/bin/btrfs_map_physical`**\
**`# cd ~/bin/btrfs_map_physical`**\
**`# wget https://raw.githubusercontent.com/osandov/osandov-linux/master/scripts/btrfs_map_physical.c`**\
**`# gcc -O2 -o btrfs_map_physical btrfs_map_physical.c`**

Run the tool and make a note of the first physical offset.\
**`# ./btrfs_map_physical /swapspace/swapfile`**
>`FILE OFFSET     FILE SIZE       EXTENT OFFSET   EXTENT TYPE     LOGICAL SIZE    LOGICAL OFFSET  PHYSICAL SIZE   DEVID   PHYSICAL OFFSET`\
>`0       4096    0       regular 268435456       298844160       268435456       1       575668224`\
>`4096    268431360       4096    prealloc        268435456       298844160       268435456       1       575668224`\
>`268435456       268435456       0       prealloc        268435456       567279616       268435456       1       844103680`

In the above example, the first physical offset returned is _**575668224**_.

Find out the PAGESIZE.\
**`# getconf PAGESIZE`**
>`4096`

To compute the resume_offset value, divide the physical offset by the pagesize.\
In this example, 575668224 / 4096 = _**140544**_

We will not need the tool anymore so we remove it.\
**`# rm -r ~/bin`**\
**`# cd`**

Previously we have made a note of /dev/mapper/cryptroot UUID which is d72f6385-bb67-4ce2-810c-8eb8935402a2.

Update GRUB configuration to make sure we have access to encrypted SYSTEM partition and the resume from hibernation details.\
**`# vim /etc/default/grub`**

Update GRUB_CMDLINE_LINUX_DEFAULT. Make sure to replace _**sdXY**_ with the corresponding SYSTEM partition name specific to your deployment.\
>`GRUB_CMDLINE_LINUX_DEFAULT="cryptdevice=/dev/sdXY:cryptroot root=/dev/mapper/cryptroot rootflags=subvol=@ resume=UUID=d72f6385-bb67-4ce2-810c-8eb8935402a2 resume_offset=140544 loglevel=3 quiet"`

Configure GRUB.\
**`# grub-mkconfig -o /boot/grub/grub.cfg`**

## 19. Boot into a newly installed system ##
Finalize the installation and boot into a newly deployed Arch Linux system.

Exit chroot environment.\
**`# exit`**

Turn off swap.\
**`# swapoff /mnt/swapspace/swapfile`**

Unmount all the partitions from /mnt\
**`# umount -R /mnt`**

Shutdown to safely remove the installation media (USD flash memory), and start the system again.\
**`# shutdown now`**

Remove the USB device with installation image and turn on the laptop. You should be greeted with grub menu. Boot the Arch Linux installation. Enter encryption passphrase to open the encrypted SYSTEM partition. And finally login as regular _**UserName**_ created in step 16.

You will probably get the below error message about missing /home/UserName directory. We will fix the issue in the next step.
>` -- UserName: /home/UserName: change directory failed: No such file or directory`\
>`Logging in with home = "/"`

Wireless network connectivity needs to be configured on a first boot into new system. Make sure to replace _**your_wifi_password**_, and _**your_ssid**_ with the respective values specific to your deployment.\
**`$ nmcli device wifi connect your_ssid password your_wifi_password`**

Once network connectivity is established you may log in to the system via SSH to perform the remaining steps.

## 20. Install Graphical Environment ##
The following steps are coming from the procedure published by [Nice Micro](https://www.youtube.com/c/NiceMicroLinux) on his [Youtube channel](https://www.youtube.com/c/NiceMicroLinux) as part of ["Understanding the Arch Linux installation procedure"](https://www.youtube.com/watch?v=wZr9WTfFed0&list=PL2t9VWDusOo-0jF18YvEVhwpxTXlXPunG) series.
As per the author, this method tries to preserve both, suckless philosophy and Arch Linux best practices. To fully understand the process, I encourage you to watch both videos published by Nice Micro:\
[Installing DWM on Arch Linux the proper Arch Way](https://youtu.be/-Hw9WLztuqM)\
[Arch Linux: Customizing and "patching" DWM through PKGBUILD](https://youtu.be/WOACiOXEuaI)

Perform initial git configuration. Make sure to use a correct _**User Name**_, _**username**_ and _**domain.com**_.\
**`$ git config --global user.email "username@domain.com"`**\
**`$ git config --global user.name "User Name"`**

We will use suckless [dinamic window manager](https://dwm.suckless.org/) and [simple terminal](https://st.suckless.org/) to build Graphical Environmnet. We will use AUR to prepare the build.\
**`$ cd ~/bin`**

**`$ git clone https://aur.archlinux.org/st`**\
**`$ git clone https://aur.archlinux.org/dwm`**

Create our own branch for ST.\
**`$ cd st`**\
**`$ git branch my-config`**\
**`$ git switch my-config`**

Make sure git keeps track of config.h and config.def.h files. Remove those files from .gitignore. Also change the _/pkg.tar.xz_ to _/pkg.tar.zst_, since this is a new format.\
**`$ vim .gitignore`**
>`/*pkg.tar.zst`\
>`/pkg/`\
>`/src/`\
>`/st-*.tar.gz`

Commit the changes to .gitignore file.\
**`$ git add .`**\
**`$ git commit -m "modified .gitignore file"`**

Create a new branch for DWM.\
**`$ cd ../dwm`**\
**`$ git branch my-config`**\
**`$ git switch my-config`**

Remove config.h file.\
**`$ rm config.h`**

Update PKGBUILD file from DWM directory.\
**`$ vim PKGBUILD`**

Remove reference config.h in source section.
>`source=(dwm.desktop`\
>`        https://dl.suckless.org/dwm/dwm-$pkgver.tar.gz)`\
>`sha256sums=('bc36426772e1471d6dd8c8aed91f288e16949e3463a9933fee6390ee0ccd3f81'`\
>`            '97902e2e007aaeaa3c6e3bed1f81785b817b7413947f1db1d3b62b8da4cd110e')`

Add _sourcedir line.
>`_sourcedir=$pkgname-$pkgver`

Change the content of prepare() function.
>`prepare() {`\
>`  # This package provides a mechanism to provide a custom config.h. Multiple`\
>`  # configuration states are determined by the presence of two files in`\
>`  # $BUILDDIR:`\
>`  #`\
>`  # config.h  config.def.h  state`\
>`  # ========  ============  =====`\
>`  # absent    absent        Initial state. The user receives a message on how`\
>`  #                         to configure this package.`\
>`  # absent    present       The user was previously made aware of the`\
>`  #                         configuration options and has not made any`\
>`  #                         configuration changes. The package is built using`\
>`  #                         default values.`\
>`  # present                 The user has supplied his or her configuration. The`\
>`  #                         file will be copied to $srcdir and used during`\
>`  #                         build.`\
>`  #`\
>`  # After this test, config.def.h is copied from $srcdir to $BUILDDIR to`\
>`  # provide an up to date template for the user.`\
>`  if [ -e "$BUILDDIR/config.h" ]`\
>`  then`\
>`    cp "$BUILDDIR/config.h" "$_sourcedir"`\
>`  elif [ ! -e "$BUILDDIR/config.def.h" ]`\
>`  then`\
>`    msg='This package can be configured in config.h. Copy the config.def.h '`\
>`    msg+='that was just placed into the package directory to config.h and '`\
>`    msg+='modify it to change the configuration. Or just leave it alone to '`\
>`    msg+='continue to use default values.'`\
>`    echo "$msg"`\
>`  fi`\
>`  cp "$_sourcedir/config.def.h" "$BUILDDIR"`\
>`}`

Copy .gitignore file rom ST to DWM directory.\
**`$ cp ../st/.gitignore .`**

Update the last line to match DWM requirements.\
**`$ vim .gitignore`**
>`/dwm-*.tar.gz`

Commit the changes to dwm.\
**`$ git add .`**\
**`$ git commit -m "modify PKGBUILD (delete config.h, add _sourcedir and update prepare() function), add .gitignore file, and remove config.h file"`**

Install ST and the missing dependencies.\
**`$ cd ../st`**\
**`$ makepkg -sifc`**

Verify the ST package is installed on the system.\
**`$ pacman -Qi st`**

Configure ST.\
**`$ cp config.def.h config.h`**\
**`$ vim config.h`**

Once the changes in config.h are done, rebuild the package.\
**`$ makepkg -sifc`**

If the changes are correct and approved, git commit. Make sure to replace _**changes_description**_ with the description of actual changes.\
**`$ git add .`**\
**`$ git commit -m "changes description"`**

Install DWM and the missing dependencies.\
**`$ cd ../dwm`**\
**`$ makepkg -sifc`**

Verify the DWM package is installed on the system.\
**`$ pacman -Qi dwm`**

Configure DWM.\
**`$ cp config.def.h config.h`**\
**`$ vim config.h`**

Once the changes in config.h are done, rebuild the package.\
**`$ makepkg -sifc`**

If the changes are correct and approved, git commit. Make sure to replace _**changes_description**_ with the description of actual changes.\
**`$ git add .`**\
**`$ git commit -m "changes description"`**

Go to user home directory and create .xinitrc file.\
**`$ cd`**\
**`$ cp /etc/X11/xinit/xinitrc ./.xinitrc`**

Update .xinitrc to make it start DWM.\
**`$ vim .xinitrc`**

Remove the last 5 lines and add the following lines instead.
>`setxkbmap pl`\
>`exec dwm`

You can now start Graphical Environment.\
**`$ startx`**

However, we will use [Ly](https://github.com/nullgemm/ly) display manager to start graphical environment.

Clone Ly AUR git repository.\
**`$ cd ~/bin`**\
**`$ git clone https://aur.archlinux.org/ly`**\
**`$ cd ly`**

Install the package, and enable the service to start on boot.\
**`$ makepkg -sifc`**\
**`# systemctl enable ly`**

After reboot you will be greeted by Ly login screen.

## 21. Configure backup to NAS and perform initial full backup ##

\
\
\
\
\
TODO: ( Test on hardware, for now the process is only tested on virtual machine )\
TODO: ( Add step 23 )
