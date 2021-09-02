In this repository I attempt to create a complete **Guide** for Arch Linux operating system deployment on a mobile personal computer (laptop). The solution should satisfy the list of requirements collected in section I. Requirements.

I am Linux enthusiast with no system administration background. This **Guide** was put together by myself following a number of documents, tutorials and recommendations available online. Hopefully this is a living document, updated frequently to catch up with the latest developments.

I would appreciate all kind of constructive feedback about the below procedure steps, applied solutions, and used software. You can use Issues section for that purpose.

**DISCLAIMER**: No liability for the contents of this documents can be accepted. Use the concepts, examples and other content at your own risk. There may be errors and inaccuracies, that may of course be damaging to your system. Although this is highly unlikely, you should proceed with caution. The author does not accept any responsibility for any damage incurred.

**IMPORTANT**: If applied incorrectly, some of the steps described in the **Guide** may irreversibly erase data from, and damage your computer.

This installation procedure heavily borrows from the following sources:
* [Arch Linux Wiki](https://wiki.archlinux.org/)
* [Arch Linux Installation guide (archlinux wiki)](https://wiki.archlinux.org/title/Installation_guide)
* [Archlinux on encrypted btrfs with systemd-boot and KDE (blog)](https://rich.grundy.io/blog/archlinux-on-encrypted-btrfs-with-systemd-boot-and-kde/)
* [Nice Micro (youtube channel)](https://www.youtube.com/channel/UC2bkdAPR47c7FkvwSXzzMzQ)
* [EF - Linux Made Simple (youtube channel)](https://www.youtube.com/c/EFLinuxMadeSimple)
* [Luke Smith (youtube channel)](https://www.youtube.com/c/LukeSmithxyz)
* [Installing Arch on an encrypted BRTFS with EFI (blog)](https://gareth.com/index.php/2020/07/15/installing-arch-on-an-encrypted-btrfs-with-efi/)

# I. Requirements #

1. Data at rest encryption (data partitions and swap).
2. Battery saving features (hibernation to disk on lid close or button press).
3. Wireless connectivity (wifi, bluetooth, wwan), if available.
4. Snapshot system (to easily restore to previous state in case of failure).
5. Periodic full backup to NAS when connected to LAN.
6. Graphical environment.
7. Should be a minimal install, include only essential packages required for each functionality.

# II. Assumptions #

1. To create an installation medium we will use Linux distribution with the following tools: GnuPG, wget, dd. For instructions to create installation medium from Windows or MacOS, please refer to Arch Linux installation guide.
2. USB flash drive (pendrive) is used as installation medium.
3. We are using the following Arch Linux image file: _**archlinux-2021.08.01-x86_64.iso**_ and corresponding _**archlinux-2021.08.01-x86_64.iso.sig**_
4. There is a LAN network with Internet access available.
5. Arch Linux live environment will be accessed via SSH to perform the installation procedure.
6. All storage devices are SSD SATA.
7. Boot mode is UEFI.

# III. Installation steps #

1. Acquire an installation image.
2. Verify an installation image signature.
3. Prepare an installation medium.
4. Boot the live environment.
5. Live environment keyboard layout and console font setup.
6. Live environment network access setup.
7. Live environment SSH access setup.
8. Live environment boot mode verification.
9. Live environment system clock update.
10. Wipe out the disk.
11. Partition the disks.
12. Setup an encryption on the main partition.
13. Format disk partitions.
14. Create btrfs subvolumes.
15. Mount the subvolumes.
16. Select the mirrors.
17. Install essential packages with pacstrap.
18. Generate fstab.
19. Chroot into the new system.
20. Set a timezone.
21. Set locale.
22. Set up the network.
23. Set root password.
24. Set initramfs.
25. Install bootloader.
26. Boot into a newly installed system.

## 1. Acquire an installation image ##
The updated list of mirrors can be found on [Arch Linux download page](https://archlinux.org/download). Download Arch Linux image (.iso) from preferred mirror, and the corresponding PGP signature file (.iso.sig) from Arch Linux download page directly.\
**`$ wget https://ftp.icm.edu.pl/pub/Linux/dist/archlinux/iso/2021.08.01/archlinux-2021.08.01-x86_64.iso`**\
**`$ wget https://archlinux.org/iso/2021.08.01/archlinux-2021.08.01-x86_64.iso.sig`**

## 2. Verify an installation image signature ##
Execute the gpg command to verify .iso file agianst .iso.sig. Both files must be in the same directory. Make sure RSA key from the output matches PGP fingerprint provided on Arch Linux download website.\
**`$ gpg --keyserver-options auto-key-retrieve --verify archlinux-2021.08.01-x86_64.iso.sig`**\
`gpg: assuming signed data in 'archlinux-2021.08.01-x86_64.iso'`\
`gpg: Signature made Sun 01 Aug 2021 10:32:22 AM CEST`\
`gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC`\
`gpg: Can't check signature: No public key`

## 3. Prepare an installation medium ##
**IMPORTANT**: Make sure you are applying the changes to the USB drive. After this step the data will be permanently erased from the device on which you apply the commands.

Find out the name of the USB drive device.\
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

## 4. Boot the live environment ##
Boot laptop with the USB drive prepared in the previous step. Arch Linux installation images does not support Secure Boot. If you are having trouble booting from the USB, please, consult [Arch Linux installation guide](https://wiki.archlinux.org/title/Installation_guide) for more details.

## 5. Live environment keyboard layout and console font setup ##
By default console keymap is US. List the directory with available keyboard layouts.\
**`# ls /usr/share/kbd/keymaps/**/*.map.gz`**

To modify the layout, append a corresponding file name without extension (.map.gz) to loadkeys. For example, for Polish programmers layout.\
**`# loadkeys pl`**

List the directory with available console fonts.\
**`# ls /usr/share/kbd/consolefonts/**/*.psfu.gz`**

To modify the console font to support Polish special characters, append a font name without extension (.psfu.gz) to setfont.\
**`# setfont lat2-16`**

## 6. Live environment network access setup ##
Since network connectivity is a critical setting for successful Arch Linux installation we will describe four different network access scenarios:
1. Wired connection with DHCP
2. Wired connection without DHCP
3. Wireless connection with DHCP
4. Wireless connection without DHCP

Make sure the card is not blocked.\
**`# rfkill list`**\
`0: phy0: Wireless LAN`\
`Soft blocked: no`\
`Hard blocked: no`

If the card is hard-blocked then use the hardware button (switch) to unblock it.\
If the card is not hard-blocked but soft-blocked then unblock it with rfkill.\
**`# rfkill unblock wifi`**

Wireless access point specification:
>SSID: MyTestLab\
>passphrase: myTestPass
>WPA/WPA2-PSK encryption mode\

Check for available network interfaces, and apply one of the above scenarios to the preferred interface.\
**`# ip link`**\
`1: lo <LOOPBACK,UP,LOWER_UP> mtu 65535 qdisc noqueue state UNKNOWN mode DEFAULT group qlen 1000`\
`link/loopacbk 00:00:00:00:00:00 brd 00:00:00:00:00:00`\
`2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel UP mode DEFAULT group default qlen 1000`\
`link/ether 08:00:27:8f:69:ff brd ff:ff:ff:ff:ff:ff`\
`3: wlan0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DORMANT group default qlen 1000`\
`link/ether 08:00:27:aa:bb:cc brd ff:ff:ff:ff:ff:ff` 

Once applied the procedure for any of the DHCP scenarios, you should verify local IP address.\
**`# ip addr`**\
`2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel UP mode DEFAULT group default qlen 1000`\
`link/ether 08:00:27:8f:69:ff brd ff:ff:ff:ff:ff:ff`\
`inet 10.0.0.2/24 metric 100 brd 10.0.0.255 scope global dynamic enp0s3`\
`valid_lft 83738sec preferred_lft 83738sec`\
`inet6 fe80::a12:abcd:a12b:1234/64 scope link`\
`valid_lft forever preferred_lft forever`

Once applied the procedure from one of the above scenarios, you should verify Internet access.\
**`# ping archlinux.org`**\
`PING archlinux.org (95.217.163.246) 56(84) bytes of data.`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=1 ttl=49 time=44.3 ms`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=49 time=44.3 ms`\
`64 bytes from archlinux.org (95.217.163.246): icmp_seq=3 ttl=49 time=44.3 ms`

### 6-1. Wired connection with DHCP ###
Connect network cable to the laptop. If DHCP server is available in the network, ethernet network interface will be configured automatically.

### 6-2. Wired connection without DHCP ###
Add an IP address to the interface.\
**`# ip address add 10.0.0.2/24 broadcast + dev enp0s3`**

Add default route to access Internet.\
**`# ip route add default via 10.0.0.1/24 dev enp0s3`**

### 6-3. Wireless connection with DHCP ###
Connect to wireless LAN.\
**`# iwctl --passphrase myTestPass station wlan0 connect MyTestLab`**

### 6-4. Wireless connection without DHCP ###
Connect to wireless LAN.\
**`# iwctl --passphrase myTestPass station wlan0 connect MyTestLab`**

Add an IP address to the interface.\
**`# ip address add 10.0.0.2/24 broadcast + dev enp0s3`**

Add default route to access Internet.\
**`# ip route add default via 10.0.0.1/24 dev enp0s3`**

## 7. Live environment SSH access setup ##
To be able to login via ssh we need to set a root password.\
**`# passwd root`**

If ssh service is already enabled/running you can try to connect via ssh. Check if ssh service is already running.\
**`# systemctl list-unit-files -t service | grep ssh`**\
`sshd.service enabled disabled`

If ssh is not running start the service.\
**`# systemctl start sshd`**

You can now connect via ssh to the live environment from another machine.

## 8. Live environment boot mode verification ##
We need verify that we are actually booted in UEFI mode. If the following command is executed without error that means we run UEFI, similarly to the below output.\
**`# mount | grep efi`**\
`efivarfs on /sys/firmware/efi/efivars type efivarfs (rw,nosuid,nodev,noexec,realtime)`

## 9. Live environment system clock update ##
Use NTP server to syncronize time.\
**`# timedatectl set-ntp true`**

Set CET timezone.\
**`# timedatectl set-timezone Europe/Warsaw`**

Verify the clock is correctly configured.\
**`# timedatectl show`**\
`Timezone=Europe/Warsaw`\
`LocalRTC=no`\
`CanNTP=yes`\
`NTP=yes`\
`NTPSynchronized=yes`\
`TimeUSec=Sun 2021-08-29 10:48:17 CEST`\
`RTCTimeUSec=Sun 2021-08-29 10:48:17 CEST`

## 10. Wipe out the disk ##
**IMPORTANT**: If you follow the instructions included in this step, the data on the disk will be erased. Proceed with caution.

This step is optional. It is only required if you want to sanitize the disk before it is used to host Arch Linux OS.

Verify the disk device name on which we will install Arch Linux.\
**`# lsblk`**\
`NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS`\
`loop0 7:0 0 662.1M 1 loop /run/archiso/airootfs`\
`sda 8:0 0 20G 0 disk`\
`sr0 11:0 1 831.3M 0 rom /run/archiso/bootmnt`

Make sure the drive security is not frozen.\
**`hdparm -I /dev/sda | grep frozen`**\
`frozen`

If it says _frozen_ it is not possible to proceed with the next step. To unfreeze the drive you may try to suspend the system. Upon waking up, it is likely that the freeze will be lifted. If this trick does not work for you, please, follow up on Arch Linux wiki [SSD/Memory cell cleaning](https://wiki.archlinux.org/title/Solid_state_drive/Memory_cell_clearing).\
**`# systemctl suspend`**\
**`# hdparm -I /dev/sda | grep frozen`**\
`not frozen`

**IMPORTANT**: Do not reboot after applying this step.\
Enable security by setting a user password.\
**`# hdparm --user-master u --security-set-pass PasSWorD /dev/sda`**

As a sanity check, let's verify that security is enabled.\
**`# hdparm -I /dev/sda`**

**IMPORTANT**: After this command the SSD will be wiped and data will be lost.\
**IMPORTANT**: Ensure that the drive is not mounted.\
Issue the ATA Secure Erase command.\
**`# hdparm --user-master u --security-erase PasSWorD /dev/sda`**

The drive is now erased, let's verify that security is not enabled.\
**`# hdparm -I /dev/sda`**

## 11. Partition the disks ##
**IMPORTANT**: If you follow the instructions included in this step, the data on the disk will be erased. Proceed with caution.

We will create two partitions. One for the boot and another one for the System. We will not use swap partition, instead we will use swapfile.

partition | type | size | file system
--------- | ---- | ---- | ----------- 
boot | esp | +550M | fat32
System | linux | ~ | btrfs

We will use GNU parted to partition the disk.\
First create GUID Partition Table.\
**`# parted -s /dev/sda mklabel gpt`**

Make partition for boot.\
**`# parted -s /dev/sda mkpart ESP fat32 1MiB 551MiB set 1 esp on name 1 efi`**

Make partition for the system.\
**`# parted -s /dev/sda mkpart System 551MiB 100% name 2 system`**

Verify the partitions are correcrtly created.\
**`# lsblk**`\
`loop0 7:0 0 662.1M 1 loop /run/archiso/airootfs`\
`sda 8:0 0 20G 0 disk`\
`└─sda1 8:1 0 550M 0 part`\
`└─sda2 8:2 0 19.5G 0 part`\
`sr0 11:0 1 831.3M 0 rom /run/archiso/bootmnt`

## 12. Setup an encryption on the main partition ##
We will encrypt the system partition with LUKS.


;\
;\
;\
;\
;\
;

# TODO #

ad 6. Live environment network setup:
	- add WWAN configuration

# TEMP #

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
