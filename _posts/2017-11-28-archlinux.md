---
layout: post
title: archlinux
---

1. toc
{:toc}

# [Installation guide](https://wiki.archlinux.org/index.php/Installation_guide)

1. Arch Linux has dropped support for *i686* platforms. Only *x86_64* now!
2. Download the latest ISO image from [download link](https://www.archlinux.org/download/).
3. Install necessary packages. At least *wpa_supplicant* as the *base* group does not include that.

## USB Stick

*dd* command will be used to create the bootable stick as it is more reliable and also the recommended method on the Wiki page. *dd* erases the whole USB stick and treats it as a hard drive.

Without explict notice, we assume the USB drive to be */dev/sdb*. *dd* command is effective but dangerous as well. It take effects immediately without confirmation prompt. Therefore, please make sure you select the exact `of=` argument.

```bash
root@archiso / # fdisk -l; blkid; lsblk -f; findmnt
root@archiso / # dd bs=1M if=/path/to/archlinux.iso of=/dev/sdb status=progress oflag=sync
```

The optimal block size `bs` argument is determined by various factors (both hardware and software) on the system. Empirically, 1M or 4M is good enough.

To restore the USB stick as normal data storage, we should remove the ISO 9660 filesystem signature before re-partitioning and re-formating the USB drive by *wipefs* command. It only erase signatures of filesystem, raid  or partition-table while leave filesystem itself untouched.

```bash
root@archiso / # wipefs --all /dev/sdb
```

## USB Booting

Disable *secure boot* in BIOS setting as most Linux distributions do not buy digital certifits from Verisign.

If the USB stick refuses to load, then try the [dd command](#usb-stick) once more. If directory `/sys/firmware/efi/efivars` exists, then it's booted in UEFI mode, otherwise it is in legacy BIOS mode.

By default, Arch ISO uses Zsh shell. To log all commands, set in *~/.zshrc* the following to empty string:

```
HISTSIZE=-1
HISTFILESIZE=-1
```

And then run `source .zshrc`.

Within the Live system, we use Alt-*arrow* or Ctrl-Alt-Fx to switch virtual console and read installation guide there.

## Time and Network

Once booted into the live system, check network connection and rectify system clock:

```bash
root@archiso ~ # ping www.archlinux.org
root@archiso ~ # timedatectl set-ntp true; timedatectl set-timezone Asia/Shanghai; timedatectl status
```

For wired/wireless network that require captive login or user agreement, refer to [Using command line to connect to a wireless network with an http login](https://superuser.com/q/132392) and [Connecting to a Wireless network with a captive portal](https://bbs.archlinux.org/viewtopic.php?id=187511). Also, please check [network](#network).

## BIOS GPT scheme

1. Grub bootloader.
2. BIOS/GPT [partitioning](https://wiki.archlinux.org/index.php/Partitioning).
3. Single *root* partition plus extra *swap file*.

This booting schema is only used with VMs in [VirtualBox](https://www.zhstar.win/2015/08/21/virtualbox/).

### Partitioning

Firstly, to identify disk devices:

```bash
root@archiso ~ # fdisk -l
root@archiso ~ # lsblk
```

For "Grub, BIOS/GPT" scheme, we must create the [BIOS boot partition](https://wiki.archlinux.org/index.php/BIOS_boot_partition) to hold Grub *core.img*. Around 1 MiB (2048 sectors) is enough. Partition number can be in any position order but has to be on the first 2 TiB of the disk. This partition should be flagged as *bios_grub* for *parted*, *ef02* for *gdisk*, or select *BIOS boot* and partition type *4* for *fdisk*.

On the other hand, "Grub, BIOS/MBR" scheme uses post-MBR gap (after the 512B MBR and before the first partition) to store *core.img*. Usually this gap is 31 KiB. For complex modules in *core.img*, this space is limited. That can be resolved by pushing forward starting sector of the first parititon. For instance, partitioning tool can align at MiB boundaries, thus leaving enough post-MBR gap. So, it eliminates the bother to create BIOS boot partition.

Here is the partitioning steps:

```
root@archiso ~ # parted -a optimal /dev/sda
(parted) help
(parted) mktable gpt
(parted) unit s
(parted) print free
(parted) mkpart primary 0% 2047s        # 1MiB 2047s
(parted) set 1 bios_grub on
(parted) mkpart primary 2048s 100%
(parted) print free
(parted) quit
```

1. Tell *parted* to optimally align at 1MiB boundaries.
2. By default, GPT's free sector starts at 34s (equally *0%*). Make BIOS boot partition be the very first parition which starts at *34s* spanning to *2047s*. *parted* reminds:

   >Warning: The resulting partition is not properly aligned for best performance. Ignore/Cancel?

   Choose *Ignore* as this partition will not be regularly accessed. Performance issues can be disregarded though this is out of GPT alignment specifications.
3. The rest space is assigned to *root* starting at *2048s*.

#### Swap

There is not any other separate partitions except that a *swap file* will be created. Swap partition bears NO advantages over swap file but could be shared among different systems. On the other hand, we can easily resize and plug/unplug a swap file on-the-fly.

Leave the swap file part after booting into new system.

### Filesystem

There is no need to create file system for the partition `bios_grub`. Hence, just create an *ext4* on the single *root* parition:

```bash
root@archiso ~ # mkfs.ext4 /dev/sda2
```

### Mount

```bash
root@archiso ~ # mount /dev/sda2 /mnt
```

As there is only the *root* parition, we would not create any other mount points under */mnt*, neither */home* nor */boot*.

## UEFI GPT scheme

Almost all modern systems take UEFI booting schema. The old BIOS legacy mode discussed above is almost deprecated. To set the PC booting in only UEFI mode in BIOS setting (disable legacy BIOS). Also make sure the ISO is [booted in UEFI mode](#usb-booting).

### Disk Preparation

1. If LVM, system encrytion (LUKS), or RAID is desired, do it now.
2. ESP could be any of FAT12, FAT16, FAT32 and VFAT (preferred).

We will erase partition or the whole drive before partitioning and formating.

### Erase Disk

>This section only covers normal mechanical disk drives. For SSDs, it is a different story.

We want to securely [erase the](https://wiki.archlinux.org/index.php/Dm-crypt/Drive_preparation#Secure_erasure_of_the_hard_disk_drive) disk by overwritting the entire drive with random data. There are a host of [general methods](https://wiki.archlinux.org/index.php/Securely_wipe_disk) available. But I choose the *dm-crypt* specific method here. Optionally, you could either erase just a single partition (i.e. */dev/sdXY*) or the complete drive (i.e. */dev/sdX*).

We first create a temporary *dm-crypt* container:

```
# erase a partition
root@archiso ~ # cryptsetup open /dev/sda1 --type plain --key-file /dev/urandom to_be_wiped
# erase a disk
root@archiso ~ # cryptsetup open /dev/sda --type plain --key-file /dev/urandom to_be_wiped
```

`--key-file /dev/urandom` is used as the encryption key. The container device is located at */dev/mapper/to\_be\_wipded*. We can verify its existence by *lsblk*.

Next, we fill the container with zeros without the need of */dev/urandom*:

```bash
root@archiso ~ # dd bs=1M if=/dev/zero of=/dev/mapper/to_be_wiped status=progress
```

For a large (i.e. a multi-terabyte disk) drive or partition, this may take hours or even a day.

Finally, close the temporary container:

```bash
root@archiso ~ # cryptsetup close /dev/mapper/to_be_wipded
root@archiso ~ # lsblk
```

### GPT Table

Use *parted* to create GPT table:

```bash
root@archiso ~ # parted -a opt /dev/sda
(parted) mklabel gpt
```

### Partitioning

For future compatability and scability, create an extra BIOS boot partition (for Grub). Additionally, I want the */boot* partition also encrypted. This setup really makes the system complicated. Anyway, the practice and security improvement deserve it!

Here is the [partitioning steps](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Encrypted_boot_partition_(GRUB)):

```bash
root@archiso ~ # parted -a opt /dev/sda
(parted) unit s/MiB
(parted) p free
(parted) mkpart primary 30s/40s 2047s
(parted) toggle 1 bios_grub
(parted) mkpart primary fat32 2048s/1MiB 551MiB
(parted) toggle 2 esp/boot
(parted) mkpart primary 551MiB 751MiB
(parted) mkpart primary 751MiB 100%
(parted) print free
```

1. As the Grub BIOS boot partition is not regularly accessed, there is no need to follow rigid alignment rules. It can just start at 34s (no alignment) or 40s (minimal alignment).

   GRUB embeds its *core.img* into this partition
2. It is [recommended](https://www.rodsbooks.com/efi-bootloaders/principles.html). to create the ESP with 550MiB.
3. When the unit used if MiB, the end of part 2 and the start of part 3 are both 551MiB. If using *s* unit, their relation should be `+1`.

Up to now, the partitions layout looks like:

```
/dev/sda1: BIOS boot
/dev/sda2: ESP
/dev/sda3: boot LUKS
/dev/sda4: '/', '/home', and '/swap' LUKS
```

### ESP Filesystem

```bash
root@archiso ~ # mkfs.vfat -F32 /dev/sda2
root@archiso ~ # parted /dev/sda p free ; fdisk -l /dev/sda
```

1. BIOS boot partition shall not be formated or mounted. Just leave it alone.
2. Both *sda3* and *sda4* will be encrypted by LUKS, so there is no need to create filesystem at this stage.

### LUKS

As Grub does not support LUKS2, use LUSK1 (`--type luks1`) on */dev/sda3* for */boot*.

Firstly, we create a passphrase-protected file by *gpg* (namely *arch-luks.gpg*) that will be used to to encrypt the LUKS container.

```bash
root@archiso ~ # export GPG_TTY=$(tty)
root@archiso ~ # dd if=/dev/urandom bs=8388607 count=1 | gpg -v --symmetric --cipher-algo AES256 --armor --output ~/arch-luks.gpg
```

Why `bs=8388607 count=1`? Because the maximum key file size is 8192KiB. From the empirics of [Gentoo installation](https://www.zhstar.win/2015/09/10/lvm-luks-lvm/), we should decrease the maximum value by 1.

Before anything else, make a backup of the key file (i.e. to a USB stick), as otherwise the Live environment would lose it upon reboot.

Next, we create the LUKS *container*:

```bash
root@archiso ~ # gpg --decrypt ~/arch-luks.gpg | cryptsetup luksFormat -v --type luks1 --cipher aes-xts-plain64 --key-size 512 --hash sha512 --key-file - --use-random /dev/sda3
root@archiso ~ # gpg --decrypt ~/arch-luks.gpg | cryptsetup luksFormat -v --type luks2 --cipher aes-xts-plain64 --key-size 512 --hash sha512 --key-file - --use-random /dev/sda4
```

*luksFormat* does not format the device, but sets up the *header* part of the LUKS container. Specially it encrypts the *master key* and embeds it into the header.

>Master key is randomly choosen upon container creation and, encrypted and embedded in the header part.

It is the master key's job to encrypt the data on partition. Key file or passphrase is only used to [encrypt/decrypt the master key](https://gitlab.com/cryptsetup/cryptsetup/wikis/FrequentlyAskedQuestions).

For safety, add a fallback passphrase in addition to the key file. It always asks for an existing passphrase or key file before adding a new one.

```bash
root@archiso ~ # cryptsetup luksAddKey -v --key-file arch-luks --iter-time 5000 /dev/sda3 --key-slot 0
root@archiso ~ # cryptsetup luksAddKey -v --key-file arch-luks --iter-time 5000 /dev/sda4 --key-slot 0
```

Apart from *luksAddKey*, we can also *luksRemoveKey*, *luksKillSlot* etc. 

We can use *luksDump* command to check the LUKS header:

```bash
root@archiso ~ # cryptsetup luksDump /dev/sda3
root@archiso ~ # cryptsetup luksDump /dev/sda4
```

It's better to backup the LUKS header:

```bash
root@archiso ~ # cryptsetup luksHeaderBackup /dev/sda3 --header-backup-file ~/sda3-luks-header.img
root@archiso ~ # cryptsetup luksHeaderBackup /dev/sda4 --header-backup-file ~/sda4-luks-header.img
```

Of course, we have a counterpart command *luksHeaderRestore* to restore header file. Once backuped, the the LUKS header may be deleted from the container. Details, refer to Arch wiki.

Once created, it is time to *open* the containers. The first open command, uses the fallback passphrase while the second uses key file.

```bash
root@archiso ~ # cryptsetup open /dev/sda3 cryptboot
root@archiso ~ # cryptsetup --key-file ~/arch-luks open /dev/sda4 cryptlvm
root@archiso ~ # lsblk
root@archiso ~ # ls -al /dev/mapper/
```

The *open* command asks for key file or passphrase to decrypt the master key in the LUKS header. The decrypted master key is then used to decrypte the LUKS container. The decrypted containers are now available at */dev/mapper/*.

### LVM

LUKS containers are ready for LVM containers on which we then create Arch Linux filesystem.

At this stage, we only care about */dev/mapper/cryptlvm* as there is no LVM requirement on */dev/mapper/cryptboot*.

```bash
root@archiso ~ # pvcreate /dev/mapper/cryptlvm ; pvdisplay -v -m
root@archiso ~ # vgcreate volgrp /dev/mapper/cryptlvm ; vgdisplay -v
root@archiso ~ # lvcreate --size 16G volgrp --name swap
root@archiso ~ # lvcreate --size 60G volgrp --name root
root@archiso ~ # lvcreate --size 250G volgrp --name home ; lvdisplay -v -m
root@archiso ~ # ls -al /dev/{mapper,volgrp}
```

1. Create a physical volume */dev/mapper/cryptlvm* on top of the LUKS container.
2. Create a volume group *volgrp* and add the physical volume into it. We can add more, if multiple physical volumes exist.
3. Create logical volumes within *volgrp*.

   For futher compatibility, leave some *volgrp* space unallocated. Size of logical volumes can be ajusted on the fly.

Created logical volmes' symbolic links reside in two equivalent locations, namely */dev/mapper/* and */dev/volgrp/*.

### Filesystem

Format the logical volumes.

```bash
root@archiso ~ # mkswap /dev/volgrp/swap
root@archiso ~ # mkfs.ext4 /dev/volgrp/root
root@archiso ~ # mkfs.ext4 /dev/volgrp/home
```

Recall that, we have not create filesystem for the decrypted *cryptboot* yet. Just create filesystem on top LUKS container directly without LVM.

```bash
root@archiso ~ # mkfs.ext2 /dev/mapper/cryptboot
root@archiso ~ # lsblk
```

### Mounting

```bash
root@archiso ~ # blkid
root@archiso ~ # swapon /dev/volgrp/swap
root@archiso ~ # mount /dev/volgrp/root /mnt
root@archiso ~ # mkdir /mnt/home; mount /dev/volgrp/home /mnt/home
#
root@archiso ~ # mkdir /mnt/boot; mount /dev/mapper/cryptboot /mnt/boot
#
root@archiso ~ # mkdir /mnt/efi; mount /dev/sda2 /mnt/efi
#
root@archiso ~ # lsblk
```

1. Before mounting, we should use *blkid* to check relevant partition and filesystem are correctly prepared.
2. LUKS boot partition can be mounted directly as we have already create the filesystem on it.

## Mirrors

Get a state-of-the-art copy from [mirrorlist generator](https://www.archlinux.org/mirrorlist/):

```bash
curl -vo mirrorlist https://www.archlinux.org/mirrorlist/?country=all&protocol=http&protocol=https&ip_version=4&ip_version=6
```

Edit */etc/pacman.d/mirrorlist* and place geographically closest mirrors on top.

## base group

```bash
# base
root@archiso ~ # pacstrap /mnt base base-devel
# optional
root@archiso ~ # pacstrap /mnt nano
```

1. This will install all packages from *base* and *base-devel* to the *root* partition. Around 200 MiB packages will be downloaded and 700 MiB disk space consumed. Ignore the warning on *locale* failure that will be handled after *chroot*.

   *base* is a *meta* package while *base-devel* is a package *group*. Check [Meta package and package group](https://wiki.archlinux.org/index.php/Meta_package_and_package_group).
2. The *base* group does not even has a text editor. So we install *nano* explicitly here. Other groups or individual packages can also be appended to the *pacstrap* command. However, we'd better install only necessary packages at this stage. Of course, we can install packages within [chroot](#chroot) by *pacman* or when the OS is completely installed.

## fstab

```bash
root@archiso ~ # genfstab -U /mnt >> /mnt/etc/fstab
root@archiso ~ # cat /mnt/etc/fstab
```

1. `-U` and `-L` use UUID and label respectively.

Attention that, the separate *cryptboot* container is also included in */etc/fstab*. To support automount of */boot* partition after booting, we can resort to [/mnt/etc/crypttab](https://wiki.archlinux.org/index.php/Dm-crypt/System_configuration#crypttab):

```
# /mnt/etc/crypttab
cryptboot	/dev/sda2	/efi/arch-luks	cipher=aes-xts-plain64:sha512,size=512
```

## Copy Keyfile

Before chrooting, copy the key file to ESP partition.

```bash
root@archiso ~ # cp ~/arch-luks.gpg /mnt/efi/
```

## Chroot

```bash
root@archiso ~ # arch-chroot /mnt
```

Much simpler than that of [Gentoo](https://www.zhstar.win/2015/09/10/lvm-luks-lvm/). The shell prompt becomes `[root@archiso /#]` and the default shell becomes Bash.

To log all commands, set the following in *~/.bashrc* empty string or negative integer:

```
# ~/.bashrc
HISTSIZE=-1
HISTFILESIZE=-1
```

## Time

```bash
[root@archiso /#] ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# On VirtualBox
[root@archiso /#] hwclock -w/--systohc
```

## Localization

Mainly, we will modify two files */etc/locale.gen* and */etc/locale.conf*.

Uncomment the following lines from */etc/locale.gen*:

```bash
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GB18030 GB18030
```

Generate the locale:

```bash
[root@archiso / #] grep '^[^#]'/etc/locale.gen

[root@archiso / #] locale-gen
[root@archiso / #] locale -a

[root@archiso / #] echo LANG=en_US.UTF-8 >> /etc/locale.conf
```

Here we set system-wide locale to *en_US.UTF-8*. For per-user locale, leave it to *~/.config/locale.conf*.

To set customized keymap permanently, edit (create) */etc/vconsole.conf*:

```bash
[root@archiso / #] echo 'KEYMAP=emacs' >> /etc/vconsole.conf
```

Check */usr/share/keymaps/i386/qwerty* for predefined keymaps. Switch to a keymap temporarily:

```bash
root@archiso ~ # loadkeys emacs
```

Attention, this only affects keyboard layout in virtual console. X keyboard layout is discuessed [here](/2019/03/11/x/#x-keyboard).

## Network

Set hostname:

```bash
# Create /etc/hostname
[root@archiso / #] echo "myhostname" >> /etc/hostname
```

Update */etc/hosts*:

```bash
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

The third line in the request uses *127.0.1.1* instead of *127.0.0.1*. It does not matter which one is used, actually. But [hostname resolution](https://www.debian.org/doc/manuals/debian-reference/ch05.en.html#_the_hostname_resolution) provide some information.

Install NetworkManager:

```bash
[root@archiso / #] pacman -S networkmanager
```

NetworkManager has built-in DHCP client, so we don't need to install *dhcpcd*. Aslo, it will pull in `wpa_supplicant`. To configure `wpa_supplicant.conf`, check [Gentoo Installation](https://www.zhstar.win/2015/03/25/gentoo-installation/).

## Initramfs by mkinitcpio

```
# /etc/mkinitcpio.conf
MODULES=(vfat)
HOOKS=(base udev autodetect modconf block filesystems keyboard fsck keymap encrypt lvm2)
```

hook | info
--- | ---
keymap | needed as custom keymap (*emacs*) is set in *vconsole.conf*.
encrypt | LUKS
lvm2 | LVM

1. Add **vfat** module, otherwise *initramfs* fails load the key file.
2. This is a *busybox* based *initramfs* configuration. It is different from *systemd init* booting process, which is the next phase.
3. Use `mkinitpico -H <hook-name>` to check hooks info. Use `mkinitcpio -L` to list available hooks.
4. Remove *consolefont* hook as it is unnecessary.

Before creating the new *initramfs*, we should install *lvm2*, otherwise *mkinitcpio* cannot find the hook:

```bash
[root@archiso / #] pacman -S lvm2
```

Now we will re-create the *initramfs* to include the new hooks, though *pacstrap* above already created it upon installation of *linux* package (pulled in by the *base* group).

```bash
[root@archiso / #] mkinitcpio -P
```

*mkinitcpio* will install generate two images, a *default* and a *fallback* that skips the autodetect hook thus including a full range of mostly-unneeded modules. Obviously, the fallback image is much bigger in size than the default one as it attempts to pull in as many modules as possible. Therefore, *fallback* image generation usually reports [missing firmware](https://wiki.archlinux.org/index.php/Mkinitcpio#Possibly_missing_firmware_for_module_XXXX). If the system does not have such hardware, it can be safely ignored. Otherwise, just install the relevant modules like 'wd719x-firmware'.

## root password

```bash
[root@archiso / #] passwd
```

## Boot loader - Grub

```bash
[root@archiso / #] pacman -S grub efibootmgr intel-ucode
```

1. *efibootmgr* is used by the GRUB installation script to write boot entries to NVRAM.
2. *intel-ucode* is used to apply CPU stability and security updates during boot.

Since the boot partition is encrypted by LUKS, so we should enable encryption option for Grub.

```
# /etc/default/grub
GRUB_ENABLE_CRYPTODISK=y
```

This option is used by *grub-install* to generate *core.img*. Then we set the kernel parameters so that *initramfs* can unlock (using *encrypt* hook) the encrypted swap, root, home partitions. For the arguments format, check `mkinitcpio -H encrypt` before editing */etc/default/grub*.

```
# /etc/default/grub

# or PARTUUID
GRUB_CMDLINE_LINUX="cryptdevice=UUID=of-sda4:cryptlvm cryptkey=UUID=of-sda2:auto:/arch-luks resume=UUID=of-volgrp-swap root=UUID=of-volgrp-root"
# or /dev/mapper/volgrp-root
GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda4:cryptlvm cryptkey=/dev/sda2:vfat:/arch-luks resume=/dev/volgrp/swap root=/dev/mapper/volgrp-root"
```

1. It is error-prone to input UUID/PARTUUID without GUI, we can replace UUID with the device pathname.
2. Please make sure the key file is located under ESP partition.

   If you forget the copy the key file before choorting, just exit the chroot, copy key file, and re-enter.
3. [Kernel parameter](https://wiki.archlinux.org/index.php/Kernel_parameters)

   *resume* is used to *suspend to disk*. *root* is optional as long as *grub-mkconfig* is used to generate the boot menu.
4. *mkinitpico* by default, doe not support *gpg* hook. Therefore, we cannot use a gpg-procted key file unless [mkinitcpio-gnupg](https://aur.archlinux.org/packages/mkinitcpio-gnupg/) is adopted. However, using a gpg-protected key file does not add security level compared to [passphrase](#luks) as the protected key file is also protected by passphrase. Hence, we use file *arch-luks* directly.

Once the configured, we can install the bootloader to ESP partition:

```bash
[root@archiso / #] grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck --modules="part_gpt"
```

Attention that, there is no */dev/sda* argument as that is for legacy BIOS booting.

Then, generate boot menu:

```bash
[root@archiso / #] pacman -S os-prober
[root@archiso / #] grub-mkconfig -o /boot/grub/grub.cfg
```

In order for Grub to find Windows 10 system, please [turn off Fast Startup](https://wiki.archlinux.org/index.php/GRUB#Windows_8/10_not_found). Also make sure [os-prober* is installed](https://www.zhstar.win/2015/03/25/gentoo-installation/). By the way, */boot* and */efi* must also be mounted.

If the *grub-mkconfig* hangs there, probably read [LVM need access to /run/lvm under new root](https://bugs.archlinux.org/task/61040) and use the [suggested hack](https://bbs.archlinux.org/viewtopic.php?pid=1820949#p1820949).

```
root@archiso ~ # mkdir /mnt/hostlvm
root@archiso ~ # mount --bind /run/lvm /mnt/hostlvm
root@archiso ~ # arch-chroot /mnt
root@archiso ~ # ln -s /hostlvm /run/lvm
```

The following is for BIOS/GPT schema:

```bash
[root@archiso / #] pacman -S grub intel-ucode
[root@archiso / #] grub-install --target=i386-pc /dev/sda
[root@archiso / #] grub-mkconfig -o /boot/grub/grub.cfg
```

Although we install *x86_64* Arch Linux, the `--target` should be *i386-pc* due to BIOS booting scheme.

## Reboot

```bash
[root@archiso / #] cp .bash_history /path/to/usb-stick/bash.log (optionally as it remains after reboot but maybe overriden)
[root@archiso / #] exit/Ctrl-D
root@archiso ~ # cp .zsh_history /path/to/usb-stick/zsh.log
root@archiso ~ # umount -R /mnt
root@archiso ~ # reboot
```

We'd better unmount all partitions under */mnt* to determine busy partitions and diagnose with *fuser*.

# General Settings

Check [Post-installation](https://wiki.archlinux.org/index.php/General_recommendations).

1. Enable NetworkManager

   ```bash
   # optional on desktop as networkmanager will detect Wi-Fi interface and ask for Wi-Fi password
   [root@host ~]# wpa_passphrase ssid psk > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf
   
   [root@host ~]# systemctl enable/start NetworkManager
   ```

2. [New user account](/2015/03/25/gentoo-installation/)
4. Swap file

   Optional if swap partition is created.

   ```bash
   [root@host ~]# dd bs=1M count=1024 if=/dev/zero of=/swapfile
   [root@host ~]# chmod 600 /swapfile
   [root@host ~]# mkswap /swapfile
   [root@host ~]# swapon /swapfile
   [root@host ~]# swapon --show
   [root@host ~]# free -h
   ```

   Finally, addan entry to */etc/fstab*:

   >/swapfile none swap defaults 0 0

   1. We must use the exact swap file path instead of UUID or Label.
   2. To cease manual operation bother, try *systemd-swap* that automates swap management.

# X #

[X](/2019/03/11/x/).

# Time

Refer to [Linux Time](/2021/03/03/linux-time).

# pacman #

*/etc/pacman.conf* sets almost everything regarding pacman, like database, cache, log, repos etc.

pacman has two data warehouses:

1. main repo. the synced database that stores package metadata (`pacman -S`).

   ```bash
   ~ $ tar -tzpvf /var/lib/pacman/sync/core.db
   ```

2. local repo. the locally installed packages (`pacman -Q`).

```bash
[root@host ~ #] pacman -Ss regex                       # search for pkg
[root@host ~ #] pacman -Si pkg                         # show pkg info
[root@host ~ #] pacman -Sg grp                         # show group info
[root@host ~ #] pacman -S pkg1 pkg2                    # install pkg1 and pkg2

[root@host ~ #] pacman -Sy archlinux-keyring           # upgrade keyring first
[root@host ~ #] pacman -Syu                            # roll the system
[root@host ~ #] pacman -Syu pkg                        # roll the system and install pgk
                                                       
[root@host ~ #] pacman -Qs pkg                         # search for locally installed pkg 
[root@host ~ #] pacman -Qi pkg                         # show locally installed pkg info
[root@host ~ #] pacman -Qe                             # list explicitly installed pkgs
                      
[root@host ~ #] pactree pkg                            # show dependency

[root@host ~ #] pacman -F /path/to/file                # pathname in main repo
[root@host ~ #] pacman -Qo /path/to/file               # pathname in local repo
                                                       
[root@host ~ #] pacman -D --asdeps pkg                 # mark a package as non-explicitly installed
[root@host ~ #] pacman -D --asexplicit pkg             # mark a package as explicitly installed
                                                       
[root@host ~ #] pacman -R pkg                          # remove pkg but leave dependencies alone
[root@host ~ #] pacman -R grp
[root@host ~ #] pacman -Rs pkg                         # remove pkg and orphan dependencies; report error: argument '-' specified with empty stdin when no orphans

[root@host ~ #] pacman -Qtd                            # query orphaned packages
[root@host ~ #] pacman -Qtdq | pacman -Rns -           # remove orphaned packages

[root@host ~ #] paccache -ruk0                         # clean downloaded packages
[root@host ~ #] pacman -Scc                            # more aggressive
```

## AUR ##

Use [wps-office-cn](https://aur.archlinux.org/packages/wps-office-cn) as an example.

```bash
# get the PKGBUILD file
~ $ git \
-c 'url.https://aur.archlinux.org/wps-office-cn.git.insteadOf=https://aur.archlinux.org/wps-office-cn.git' \
clone --depth=1 \
https://aur.archlinux.org/wps-office-cn.git

# build the package
~ $ cd wps-office-cn$
~ $ makepkg

# install the package
~ $ sudo pacman -U wps-office-cn-11.1.0.11698-1-x86_64.pkg.tar.zst
```

*wpspdf* relies on [libtiff5](https://aur.archlinux.org/packages/libtiff5).

To upgrade the package, repeat the above procedures.

# Xterm

Install [xterm](https://wiki.archlinux.org/index.php/Xterm) package. Add the following into *~/.Xresources*:

```
XTerm.termName: xterm-256color
XTerm.vt100.locale: true
XTerm.vt100.metaSendsEscape: true
XTerm.vt100.reverseVideo: true

XTerm.vt100.backarrowKey: false
XTerm.ttyModes: erase ^?

XTerm.bellIsUrgent: true

XTerm.vt100.faceName: DejaVu Sans Mono:style=Book:antialias=true
XTerm.vt100.faceNameDoublesize: WenQuanYi WenQuanYi Bitmap Song
XTerm.vt100.faceSize: 10

XTerm.vt100.translations: #override \n\
    Ctrl Shift <Key>C: copy-selection(CLIPBOARD) \n\
    Ctrl Shift <Key>V: insert-selection(CLIPBOARD)
```

To take these settings into effect immediately:

```bash
[root@host ~ #] xrdb -merge ~/.Xresources
# -or-
[root@host ~ #] xrdb ~/.Xresources
```

*xrdb* without the `-merge` option will reload _.Xresources_, replacing current settings.

## Copy and Paste

X11 has two kinds of buffers, namely PRIMARY and CLIPBOARD.

To copy/paste to/from the CLIPBOARD , we select (highlight) text, press Ctrl-C and Ctrl-V. Selected text will be copied to PRIMARY buffer automatically. To paste from PRIMARY, press the middle mouse button. *Shift-insert* also pastes from PRIMARY, but support only by terminal emulators. Implicitly, PRIMARY buffer lives a short period and will be replaced with new selection.

Literally, we use *paste* and *insert*, *buffer* and *selection* interchangeably.

Buffer/Selection | In/Copy | Out/Paste/Insert
--- | --- | ---
PRIMARY | select | MiddleMouse/Shift-Insert
CLIPBOARD | select & Ctrl-C | Ctrl-V

The actual buffer and key shortcut used are determined by X application. When it comes to Xterm, selection goes to PRIMARY by default. However, we can set selection to CLIPBOARD by Ctrl-MiddbleMouse and enable *Select to Clipboard*. Alternatively, we can adjust *~/.Xresources*:

```
# ~/.Xresources
XTerm.vt100.selectToClipboard: true/false
```

Such settings will abandom the PRIMARY buffer. Instead, we add copy/paste shortcuts:

```
# ~/.Xresources
XTerm.vt100.translations: #override \n\
    Ctrl Shift <Key>C: copy-selection(CLIPBOARD) \n\
    Ctrl Shift <Key>V: insert-selection(CLIPBOARD)
```

Notice that each key binding line must be separated by `\n\`.

# Alacritty #

[xterm](#xterm) is really outdated, use [Alacritty](https://wiki.archlinux.org/title/Alacritty) on Arch Linux.

However, Alacritty sets ENV variable `TERM` to *alacritty* which would [break](https://wiki.archlinux.org/title/Alacritty#Terminal_functionality_unavailable_in_remote_shells) remote shell (e.g. SSH) like below.

```
WARNING: terminal is not fully functional
```

For general compatibility, we can configure Alacritty and set ENV `TERM` to *xterm-256color*.

```bash
# /usr/share/doc/alacritty/example/alacritty.yml
# /.config/alacritty/alacritty.yml

env:
  TERM: xterm-256color
```

# Resolution

## Virtual terminal

VirtualBox might fail to detect correct screen resolution for virtual terminal (console). Similar to [Android-x86](/2017/08/25/android-x86/) post, we should add custom kernel parameter through Grub bootloader.

In Grub menu, press *e*, and then *F2*. You are now in command line prompt, type *vbeinfo* to list possible resolutions. The one prefixed with *\** is the default. Choose a desired one (i.e. 1366x768) and press *ESC*. Append *video=1366x768* to *linux* line and press *F10*. If none of the listed values meet requirement, we can try *vboxmanage setextradata*:

```bash
~ $ VBoxManage setextradata archlinux "CustomVideoMode1" "1300x730x24"
```

The newly created value will be listed by *vbeinfo*. To make the desired resolution permanent, we can edit */etc/default/grub*:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet video=1300x730"
```

We can also set Grub's resolution by:

```
GRUB_GFXMODE="1366x768x24" (for Grub menu itself)
```

Do not forget to update the Grub menu:

```bash
[root@host ~ #] grub-mkconfig -o /boot/grub/grub.cfg
```

## Xorg

Most of the time, Xorg resolution works out of box. Command line tool *xrandr* and Xorg.conf can be used to set Xorg resolution.

Query resolution:

```bash
user@host ~ $ xrandr
```

The entry with star *\** means current resolution while with *+* means default preferred resolution. Set resolution:

```bash
user@host ~ $ echo $DISPLAY
user@host ~ $ xrandr --display :0 --output VGA-1 --mode 1366x768
```

We can put the commands into *~/.xinitrc* or *~/.config/awesome/autostart.sh*. Alternatively, create Xorg.conf */etc/X11/xorg.conf.d/10-resolution.conf*:

```
Section "Monitor"
  Identifier "VGA-1"
  Modeline "1368x768_60.00"   85.25  1368 1440 1576 1784  768 771 781 798 -hsync +vsync
  Option "PreferredMode" "1368x768_60.00"
EndSection

Section "Screen"
  Identifier "Screen 0"
  Monitor "VGA-1"
  DefaultDepth 24
  SubSection "Display"
    Depth 24
  EndSubSection
EndSection
```

You can get the Identifier value from *xrandr* output.

# Fcitx

Check [Fcitx](/2015/03/25/gentoo-installation/):

```bash
user@host ~ $ pacman -S fcitx-im fcitx-configtool
```

Then, append *run fcitx-autostart* into *~/.config/awesome/autostart.sh* and relaunch awesome.

If some QT applications cannot input Chinese, ensure the following packages are available.

```bash
user@host ~ $ pacman -Qi fcitx-qt5
user@host ~ $ pacman -Qi fcitx-qt6
```

# WPS #

WPS on Linux has two version, namely *wps-office-cn* and *wps-office*. The CN version has much more features!

```bash
~ $ git clone https://aur.archlinux.org/wps-office-cn.git
~ $ cd wps-office-cn
~ $ makepkg
~ $ sudo pacman -U wps-office-cn-11.1.0.10702-1-x86_64.pkg.tar.zst
```

To display math symbols, we need to install [AUR ttf-wps-fonts](https://aur.archlinux.org/packages/ttf-wps-fonts/).

After closing WPS, the background process refuses to exit:

```bash
~ $ sudo chmod -x /usr/lib/office6/wpsoffice
~ $ sudo chmod -x /usr/lib/office6/wpscloudsvr
```

However, remove the excution permission may disallow WPS login. It's a trade-off.
