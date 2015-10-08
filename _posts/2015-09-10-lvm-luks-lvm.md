---
layout: post
title: LVM over LUKS over LVM
---

1. sad7 and sda8 are separated by other NTFS partitions in use.
2. Merge them together as if they were a single partition/disk by LVM - our first LVM container.
3. Encrypt the new single LVM volume DM-crypt LUKS - our LUKS container.
4. Create LVM volumes in LUKS container as Gentoo / and /home - our second LVM container.
5. LVM -> LUKS -> LVM.
6. No swap partition. If possible, create a swapfile instead.
7. For boot and EFI, Gentoo resorts to USB stick. boot and EFI shares a single USB partition, say sdc1. This is different from traditional scheme, boot and EFI resides on different partitions no matter they are HDD or USB ones.
    1. The shared USB partition should be formated as vfat (FAT32).
    2. It should be mounted on /boot if necessary.
    3. Make a directory 'EFI' at root of the shared parition.
    4. The shared USB partition is not encrypted.
    5. You'd better use sdc2 for the shared partition to hide it from Windows. Deatils refer to *Hide in  Windows* next.
8. Windows uses its own EFI partition on HDD, say sda2.
9. Why now share boot and EFI on USB stick?
    1. A single USB can now help boot many PCs and notebooks, as long as their boot information is located on the USB stick.
    2. Kernels now stored in USB, prevent from attack online. We can unplug the USB when booting into Gentoo.
10. sda7,sda8 > pvcreate, vgcreate (vg), lvcreate (crypt), vg-crypt > cryptsetup, luksOpen (cryptroot) > pvcreate, vgcreate (cryptvg), lvcreate (root, home), cryptvg-root & cryptvg-home.

# USB preparation

1. parted -a optimal /dev/sdc

    ```
    mkpart primary fat32 0% 256MiB
    set 1 boot on
    quit
    ```
    EFI partition should be set 'boot' flag.
2. mkfs.vfat -F32 /dev/sdc1
3. USB shared partition is not encrypted. If you put the shared partition within the LVM-LUKS-LVM architecture, then you need add LUKS support to grub2.

# key-file

1. mkdir /mnt/sdc1
2. mount /dev/sdc1 /mnt/sdc1
3. mkdir /mnt/sdc1/luks-gnupg-key
4. $ dd if=/dev/urandom bs=8388607 count=1 | gpg --symmetric --cipher-algo AES256 --output ~/luks-gnupg-key.gpg

    LiveCD root acount does not support this command as 'gpg 2*' command need GUI popup to input password. Switch to normal user terminal in LiveCD.

    Without explicitly mark '$', all commands are executed in root account in LiveCD.
5. $ gpg --decrypt /mnt/sdc1/key/luks-key.gpg > ~/luks-gnupg-key

    This temporary decrypted key-file is used in root account. Don't expose decrypted key-file online or to anyone.
6. cp /home/gentoo/luks-gnupg-key.gpg /mnt/sdc1/luks-gnupg-key/

    The gpg-encrypted key-file is stored at the shared USB as well. You can store it in other media.
7. umount /mnt/sdc1

# sda7 & sda8 preparation

On sda, find the separated free space:

```
parted -a optimal /dev/sda
unit s
mkpart primary start end
name 7 'Gentoo partition'
mkpart primary start end
name 8 'Gentoo partition'
```
**start** and **end* should be exactly at the right *sector* edge. Otherwise, other partitions on sda would be ruined! If possible, resort to percentage like 0% or 100%.

We don't create filesystem 'ext4' on sda7 or sda8 directly since they are treated as underlying container under LVM -> LUKS -> LVM.

# 1st LVM container

1. vgcreate vg /dev/sda7 /dev/sda8

    'pvcreate' is optional since 'vgcreate' will automatically create PVs on sda7 and sda8 if needed.

    We can find /dev/vg directory created.
2. lvcreate -l +100%FREE  vg -name crypt

    Now, sda7 and sda8 is merged as a single LVM device /dev/mapper/vg-crypt or /dev/vg/crypt.

    Each time you create a LVM in VG, a new mapper device name will be in /dev/mapper/. The mapper device name will be 'VG name + LV name'. This mapper file is actually a simbolic link. Similarly, /dev/vg/crypt is also a symbolic linking to the same destination as /dev/mapper/vg-crypt.

    However, when booting into Gentoo, these mapper device will be real device file.
3. Try to use 'pvdisplay', 'vgdisplay', 'lvdisplay', 'lvscan' etc.

# LUKS container

1. cryptsetup --cipher serpent-xts-plain64 --key-size 512 --hash sha512 --key-file /home/gentoo/luks-gnupg-key luksFormat /dev/mapper/vg-crypt

        cryptsetup luksDump /dev/mapper/vg-crypt
2. cryptsetup --key-file /home/gentoo/luks-gnupg-key luksAddKey /dev/mapper/vg-crypt

    To add a fallback LUKS passphrase, just in case we lost the key-file.

    We must provide the the previous key-file or passphrase before adding a LUKS key slot. According to current experience, passphrase needs less time.

    Up to now, /dev/mapper/vg-crypt is protected by both LUKS key-file and passphrase. We have achieved **full disk encrytion**.

        cryptsetup luksDump /dev/mapper/vg-crypt
3. cryptsetup luksOpen /dev/mapper/vg-crypt cryptroot

    Treat vg-crypt as a normal disk partition which is encrypted by DM-crypt LUKS. To make use of it, we have to decrypt it first.

    Here, we use the fallback passphrase to decrypt. It is faster.

    When vg-crypt is decrypted, we also get a mapper device /dev/mapper/cryptroot. The device name can be set freely at your desire.

    We can now treat the decrypted device as a normal disk partition, on which we will create our 2nd LVM container.

# Why 2nd LVM container?

This question can be narrowed down to: why need LVM over LUKS?

    A decrypted LUKS device like /dev/mapper/cryptroot should'd better be treated as a partition instead of a disk (like /dev/sda). So command like 'parted -a optimal /dev/mapper/cryptroot' is NOT encouraged. It is complicated to make use of partitions created directly over LUKS device.

1. If I want to use only part of /dev/mapper/cryptroot storage for Gentoo, while leaving the remaining space for other usage like NTFS data or even another Arch Linux, use 2nd LVM.
2. If I want to separate /home, /usr, /tmp etc. mount points, use 2nd LVM.
3. etc.

Actually, up to now, we can install Gentoo on this decrypted LUKS partition directly (single partition Gentoo; /home, /swap etc are just normal directories).

    mkfs -t ext4 /dev/mapper/cryptroot
    mount -t ext4 /dev/mapper/cryptroot /mnt/gentoo

Afterwards, we can also re-format /dev/mapper/cryptroot for other usage like re-installing Gentoo.

With LVM over LUKS you need only one key/password to unlock all the partitions (LVM volumes); and you can rearrange, resize etc. the partitions without the hassle of re-encrypt everything again. For example, you want to extend / size while reduce /home size.

If you're going to simply dump everything on a single partition (which is totaly fine with some setups), there is no point in going for LVM. However, if you want to have several volumes, LVM will make it more convenient and easier to use, and this is why many people go after 'LVM over LUKS'.

If you really don't like the 2nd LVM and still need separate partitions for /, /home, etc. Please lvcreate LVM volumes for them in the 1st LVM container. After that, cryptsetup them repsectively with LUKS key-files or passphrases.

Refer to:

1. [luks without lvm](https://forums.gentoo.org/viewtopic-t-1028630.html)
2. [LUKS over LVM or LVM over LUKS](https://forums.gentoo.org/viewtopic-t-1028448-highlight-.html)
3. [dm-crypt/Encrypting an entire system](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system)

# 2nd LVM container

1. pvcreate /dev/mapper/cryptroot

    Like in the 1st LVM container, this step is optional.
2. vgcreate cryptvg /dev/mapper/cryptroot

    /dev/cryptvg/ directory created.
3. lvcreate --size 25G --name root cryptvg
4. lvcreate --extents 100%FREE --name home cryptvg

    Wee will see 'cryptvg-root' and 'cryptvg-home' symbolic links in /dev/mapper. Similarly, /dev/cryptvg{/root, /home} links created as well.

    Up to now, we have finished the basic LVM -> LUKS -> LVM architecture. We will install Gentoo on top the newly created LVM volumes (cryptvg-root and cryptvg-home).

# Format Gentoo partitions

1. mkfs.ext4 -L "root" /dev/mapper/cryptvg-root
2. mkfs.ext4 -m 0 -L "home" /dev/mapper/cryptvg-home
3. boot and EFI share USB partition /dev/sdc1.

# Mount Gentoo partitions

    When mount and umount, try 'lsblk', 'blkid' etc. commands.

1. mount -v -t ext4 /dev/mapper/cryptvg-root /mnt/gentoo
2. mkdir -v /mnt/gentoo/{home,boot}
3. mount -v -t ext4 /dev/mapper/cryptvg-home /mnt/gentoo/home
4. umount -v /dev/sdc1
4. mount -v -t vfat /dev/sdc1 /mnt/gentoo/boot
5. mkdir /mnt/gentoo/boot/EFI

    It might be upper case 'EFI' or lower case 'efi'. Currently, 'EFI' works fine.

# stage tarbar

1. cd /mnt/gentoo
2. tar xvjpf /path/to/stage3-amd64-20150319.tar.bz2

# Chroot

```
vgchange -a y [vg], for '-a y', VG name can be ommited.
cryptsetup luksOpen /dev/mapper/vg-crypt cryptroot, you could use key-file to decrypt vg-crypt as well.
vgchange -a y [vgcryptvg]
mount -v -t ext4 /dev/mapper/cryptvg-root /mnt/gentoo
mount -v -t ext4 /dev/mapper/cryptvg-home /mnt/gentoo/home
mount -v -t vfat /dev/sdc1 /mnt/gentoo/boot
```
If you return to LiveCD and try to chroot again, please execute the above six commands first. Otherwise, follow steps below.

1. cp -v -L /etc/resolv.conf /mnt/gentoo/etc/
2. mount -v -t proc proc /mnt/gentoo/proc2
3. mount -v --rbind /sys /mnt/gentoo/sys
4. mount -v --rbind /dev /mnt/gentoo/dev
5. mount -v --make-rslave /mnt/gentoo/sys
6. mount -v --make-rslave /mnt/gentoo/dev
7. chroot /mnt/gentoo /bin/bash
8. source /etc/profile && export PS1="(chroot) $PS1"

# Installation

Refer to [Gentoo Installation](2015-03-25-gentoo-installation.md) and [gentoo over lvm luks](2015-08-15-gentoo-over-lvm-luks.md).

1. Try to restore some config backup files like 'make.conf', 'fstab', 'wpa_supplicant.conf', '.bashrc' 'repos.conf/{gentoo.conf, layman.conf, local.conf} etc.
2. Of the most important, the fundamental difference is 'grub2' and 'genkernel initramfs'.

    Follow steps below.

# Difference

## initramfs

1. echo "sys-kernel/genkernel cryptsetup" > /etc/portage/package.use/genkernel
2. emerge -av genkernel
3. genkernel --lvm --luks --gpg --busybox --install initramfs

    Don't worry about '--gpg' problem occured in LiveCD above. genkernell will compile GnuPG 1 instead of 2.

## grub2

1. grub2-install --target=x86_64-efi --efi-directory=/boot --boot-directory=/boot --bootloader-id=grub2 --removable --modules=part\_gpt /dev/sdc
    - '--bootloader-id=grub2' and '/dev/sdc' (USB stick) might be optional.
    - '--efi-directory' specifies the mountpoint of the ESP. It replaces '--root-directory' which is deprecated.

    Refer to [EFI boot with GRUB2 on amd64, dual boot with Windows7 x64](https://forums.gentoo.org/viewtopic-p-7011836.html) and [grub2 zh-CN](https://wiki.gentoo.org/wiki/GRUB2/zh-CN).
2. Kernel and init arguments '/etc/default/grub'

    ```
GRUB_CMDLINE_LINUX="crypt_root=UUID='uuid of /dev/mapper/vg-crypt' dolvm root=/dev/mapper/cryptvg-root rootfstype=ext4 root_keydev=UUID='uuid of boot and EFI shared partition' root_key=/relative/path/to/luks-gnupg-key-file"
    ```
    1. crypt_root: the UUID of the partition which is encrypted by DM-crypt LUKS. In our case, this is LVM volume /dev/mapper/vg-crypt or /dev/vg/crypt.
    2. dolvm: activate LVM volumes on bootup. This needs support from LVM support in initramfs.
    3. root: the real / mount point for Gentoo. In our case, it is /dev/mapper/cryptvg-root or /dev/cryptvg/root.
    4. rootfstype: Gentoo root filesystem.
    5. root_keydev: the device where DM-crypt LUKS key-file is stored. In our case, it is boot and EFI partition on USB stick.
    6. root\_key: the path to DM-crypt LUKS key-file. The value should be relative path to root_keydev mount point.
    7. You can use device file name or use UUID instead for those arguments. It's free choince.
    8. The 2nd reference adds a parameter 'target=cryptroot' whose usage is unclear. Don't try this if not sure.
3. grub2-mkconfig -o /boot/grub/grub.cfg

    In chroot, grub2-mkconfig failed to probe (by calling os-prober) Windows on HDD. When getting into Gentoo, execute it again to make up.

    In real Gentoo system, *grub2-mkconfig* might complain about *lvmetad* issue which does no harm. If you really want to get rid of the warning, just run *rc-service lvmetad start* before *grub2-mkconfig*.

4. In our scheme, boot and EFI shared partition is not encrypted. If it is encrypted by LUKS too, we need more configurations. Read [grub2 advanced storage](https://wiki.gentoo.org/wiki/GRUB2/AdvancedStorage).

# Get out of chroot

1. exit
1. umount -lv /mnt/gentoo/home
2. umount -lv /mnt/gentoo/boot
3. umount -lv /mnt/gentoo/dev{/shm,/pts,}
4. umount -lv /mnt/gentoo{/proc,/sys,}
5. vgchange -a n cryptvg

    Should give the VG name.
6. cryptsetup luksClose cryptroot
7. vgchange -a n vg
8. reboot

# boot & EFI image backup

1. dd if=/dev/sdc1 | xz > boot-image-backup.xz, backup of boot and EFI shared partition
2. xzcat image-file.xz | dd of=/dev/sdc1, restore from backup

# Operations to USB sdc1

From the installation process, we find for sdc1, we only:

1. Copy the LUKS key there;
2. grub2-install command;
3. grub2-mkconfig;
4. Mount sdc1 as /boot in /etc/fstab

If data ruined in sdc1 or anything else undesirable happend, just repeat step 1-4 as long as LUKS key-file or fall-back passphrase exist.

# Hide in  Windows

The current USB is 1 GiB in size which only part of is needed for boot and EFI. So we could make use of the remaining around 750MiB for usual USB data storage.

We create the shared partition /dev/sdc1 formated as FAT32. However, what if this USB is inserted into Windows? Enverything in /dev/sdc1 will be exposed as normal USB stick, the luks-gnupg-key included.

Hence we would like to achieve:

1. Make use of remaining 750MiB capacity.
2. Hide the shared partition from Windows.

For 1, we just need to create a new partition on the remaining free space. For 2, please note that windows WILL NOT SHOW BOTH PARTITIONS OF USB at the same time. One partition will be hidden and another will be visible. Technically, Windows allocates a drive letter to only single partition on a USB drive. However on linux operating system both partitions will be visible without any problem.

So a better idea is to use sdc1 as the normal data storage partition, while sdc2 for the shared partition. It would be:

```
Number  Start   End     Size   File system  Name             Flags
 1      1049kB  752MB   751MB  ntfs         USB stick
 2      752MB   1002MB  251MB  fat32        Gentoo boot EFI  boot, esp
```

*sdc1 now is formated as ntfs*. If it was formated as mkfs.vfat (fat32), then UEFI firmware would NOT recognize sdc2 as the shared partition for system boot.

At real practice, you'd best not use sdc1 for data storage just in case for operation mistake to ruin sdc2.

Refer to [how-to-create-a-separate-hidden-boot-partition-on-usb/](https://tahirzia.wordpress.com/2012/11/20/how-to-create-a-separate-hidden-boot-partition-on-usb/) and [hide from windows](https://forums.gentoo.org/viewtopic-t-1028800.html).

# References

1. http://www.ms.unimelb.edu.au/~trice/linux/tricks/luksresize/
2. http://manual.aptosid.com/en/hd-install-crypt-en.htm
3. [dm-crypt/Encrypting an entire system](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system)

# Notes
1. deselect=y notifyd? what is the point? weechat
2. noatime in fstab
3. cjktty patch
4. remember to: libtool finsih 
5. list of files need backup
6. os-probe will find Windows In Gentoo.
