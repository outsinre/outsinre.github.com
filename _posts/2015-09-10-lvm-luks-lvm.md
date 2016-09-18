---
layout: post
title: LVM over LUKS over LVM
---

1. *sda7* and *sda8* are separated by other NTFS partitions in use.
2. Merge them together as if they were a single partition/disk by LVM - our first LVM container.
3. Encrypt the new single LVM volume by DM-crypt LUKS - our LUKS container.
4. Create LVM volumes in LUKS container as Gentoo / and /home - our second LVM container.
5. LVM -> LUKS -> LVM.
6. No swap partition. If possible, create a swapfile instead referring to [swapfile](http://www.fangxiang.tk/2015/03/25/gentoo-installation/).
7. BOOT and EFI partitions share and reside on a USB partition, say *sdc1*. This is different from traditional scheme.
   1. The shared USB partition should be formated as vfat (FAT32) to satisfiy EFI filesystem requirement.
   2. It should be mounted on /boot if necessary.
   3. Make a directory 'EFI' at root of the shared parition.
   4. The shared USB partition is not encrypted.
   5. You'd better use *sdc2* for the shared partition to hide it from Windows. Deatils refer to *Hide in  Windows* next.
   6. Encryption & decryption key-file is stored on USB partition as well.
8. Windows uses its own EFI partition on HDD, say *sda2*.
9. Why now share boot and EFI on USB stick?
   1. A single USB can now help boot many PCs and notebooks, as long as their boot information is located on the USB stick.
   2. Kernels now stored in USB, prevent from attack online. We can unplug the USB when booting into Gentoo.
10. sda7, sda8 > pvcreate, vgcreate (vg), lvcreate (crypt), vg-crypt > cryptsetup, luksOpen (cryptroot) > pvcreate, vgcreate (cryptvg), lvcreate (root, home), cryptvg-root & cryptvg-home.

# USB preparation

1. Create shared USB partition:

   ```bash
   # parted -a optimal /dev/sdc
   # mkpart primary fat32 0% 256MiB
   # set 1 boot on
   # quit
   ```
   
   EFI partition should be set *boot* flag.
2. Create VFAT filesystem:

   ```bash
   # mkfs.vfat -F32 /dev/sdc1
   ```
   
3. USB shared partition is not encrypted. If you put the shared partition within the LVM-LUKS-LVM architecture, then you need add LUKS support to *grub2*.

# key-file

1. Preparations

   ```bash
   # mkdir /mnt/sdc1
   # mount /dev/sdc1 /mnt/sdc1
   # mkdir /mnt/sdc1/luks-gnupg-key
   ```
   
4. Generate GPG-encrypted key-file:

   ```bash
   $ dd if=/dev/urandom bs=8388607 count=1 | gpg --symmetric --cipher-algo AES256 --output ~/luks-gnupg-key.gpg
   ```

   LiveCD *root* acount does not support this command as *gpg 2* need GUI popup to prompt password. Switch to normal user account in LiveCD.

   Without explici mark '$', all commands are executed under *root* account in LiveCD.
6. Save key-file to USB stick:

   ```bash
   # cp /home/gentoo/luks-gnupg-key.gpg /mnt/sdc1/luks-gnupg-key/
   # umount /mnt/sdc1
   ```
   
   The gpg-encrypted key-file is stored on the shared USB for booting. You can store it in other media though unnecessary and tedious.
6. Decrypt key-file for temporal usage:

   ```bash
   $ gpg --decrypt ~/luks-gnupg-key.gpg > ~/luks-gnupg-key
   # cp /home/gentoo/luks-gnupg-key .
   ```

   This temporary decrypted key-file is used in root account. Don't expose decrypted key-file online or to anyone.

# *sda7* & *sda8* preparation

On *sda*, find the separated free space:

```bash
# parted -a optimal /dev/sda
# unit s
# mkpart primary START END
# name 7 'Gentoo partition'
# mkpart primary START END
# name 8 'Gentoo partition'
```

1. START and END should exactly be the right *sector*.

   Otherwise, other partitions in use would be ruined! Take */dev/sda7* for example, its previous ancestor partition */dev/sda4*'s last sector/unit is *100000000s*. Then the START of */dev/sda7* must be *100000001s*. Remember to **PLUS ONE**. We can verify this by examine other successive partitions in use.

   If possible, resort to percentage like 0% or 100%.

2. We don't create filesystem *ext4* on *sda7* or *sda8* directly since they are treated as underlying container: LVM -> LUKS -> LVM.

# 1st LVM container

```bash
# pvcreate /dev/sda7 /dev/sda8
# vgcreate vg /dev/sda7 /dev/sda8
# lvcreate -l +100%FREE vg -name crypt
# pvdisplay / vgdisplay / lvdisplay / lvscan
```

1. *pvcreate* is optional since *vgcreate* will automatically create PVs on *sda7* and *sda8* if needed.
2. We can find /*dev/vg* directory created.
3. Now, *sda7* and *sda8* is merged as a single LVM device */dev/mapper/vg-crypt* or */dev/vg/crypt*.

   Each time you create a LVM in VG, a new mapper device will be created */dev/mapper/*. The mapper device name will be 'VG name + LV name'. This mapper device is actually a simbolic link. Similarly, /*dev/vg/crypt* is also a symbolic linking to the same destination as */dev/mapper/vg-crypt*.

   However, when booting into Gentoo, these mapper device will be real device file.
4. Try to use *pvdisplay*, *vgdisplay*, *lvdisplay*, *lvscan* etc. anytime to get information.

# LUKS container

```bash
# cryptsetup --cipher serpent-xts-plain64 --key-size 512 --hash sha512 --key-file /home/gentoo/luks-gnupg-key luksFormat /dev/mapper/vg-crypt
# cryptsetup --key-file /home/gentoo/luks-gnupg-key luksAddKey /dev/mapper/vg-crypt
# cryptsetup luksOpen /dev/mapper/vg-crypt cryptroot
# cryptsetup luksDump /dev/mapper/vg-crypt
```

1. Create LUKS container. Try `cryptsetup luksDump /dev/mapper/vg-crypt`.
2. To add a fallback LUKS passphrase, just in case we lost the key-file. According to current experience, passphrase needs less time. This passphrase is different from the one used to encrypt key-file above.

   We must provide the the previous key-file or passphrase before adding a LUKS key slot. Attention, we use the decrypted key-file instead of the encrypted one since *root* account cannot pop up box for decryption.

   Up to now, */dev/mapper/vg-crypt* is protected by both LUKS key-file (gpg-encrypted) and passphrase. We have achieved **full disk encrytion**. Try `cryptsetup luksDump /dev/mapper/vg-crypt`.
3. Treat *vg-crypt* as a normal disk partition which is encrypted by DM-crypt LUKS. To make use of it, we have to decrypt it first. Here, we use the fallback passphrase to decrypt. It is faster.

   When *vg-crypt* is decrypted, we also get a mapper device */dev/mapper/cryptroot*. The device name can be set freely at your desire.

   We can now treat the decrypted device as another normal disk partition, on which we will create our 2nd LVM container.

# Why 2nd LVM container?

This question can be narrowed down to: why need LVM over LUKS?

A decrypted LUKS device like */dev/mapper/cryptroot* should'd better be regarded as a partition instead of a disk (like */dev/sdb*). So command like `parted -a optimal /dev/mapper/cryptroot` is NOT encouraged. It is complicated to make use of partitions created directly over LUKS device.

1. What if I take only part of */dev/mapper/cryptroot* to install Gentoo, while leaving the rest for NTFS data or even another system Arch Linux?
2. What if I separate */home*, */usr*, */tmp* etc. mount points?
3. What if I want to adjust Gentoo's storage?

Of course, we can prepare Gentoo installation on this decrypted LUKS partition directly if don't care above issues:

```bash
mkfs -t ext4 /dev/mapper/cryptroot
mount -t ext4 /dev/mapper/cryptroot /mnt/gentoo
```

Sometime, we can also re-format */dev/mapper/cryptroot* for other usage like re-installing Gentoo.

With LVM over LUKS we unlock all the partitions (LVM volumes) by one key-file / passphrase, and easily resize volums without the hassle work above. For example, you want to extend */* size while reducing */home* size.

If you're going to simply dump everything on a single partition (which is totaly fine with some setups), there is no point of 2nd LVM container. However, if you want to have several volumes, LVM will make it more convenient and easier to use, and this is why many people go after 'LVM over LUKS'.

If you really don't like the 2nd LVM but need separate */*, */home*, etc. Please *lvcreate* LVM volumes directly over the 1st LVM container. After that, *cryptsetup* them repsectively with LUKS key-files or passphrases. What a nightmare!

Refer to:

1. [luks without lvm](https://forums.gentoo.org/viewtopic-t-1028630.html)
2. [LUKS over LVM or LVM over LUKS](https://forums.gentoo.org/viewtopic-t-1028448-highlight-.html)
3. [dm-crypt/Encrypting an entire system](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system)

# 2nd LVM container

```bash
# pvcreate /dev/mapper/cryptroot
# vgcreate cryptvg /dev/mapper/cryptroot
# lvcreate --size 25G --name root cryptvg
# lvcreate --extents 100%FREE --name home cryptvg
# pvdisplay / vgdisplay / lvdisplay
```

1. Like in the 1st LVM container, this step is optional.
2. */dev/cryptvg/* directory created.
3. Create LVM *root* device as */* mount point.
4. Create LVM *home* device as */home* mount point.

   Wee will see *cryptvg-root* and *cryptvg-home* symbolic links under */dev/mapper*. Similarly, */dev/cryptvg{/root, /home}* links created as well.

   Up to now, we have finished the basic LVM -> LUKS -> LVM architecture. We will install Gentoo over the newly created devices of 2nd LVM.

# Format Gentoo partitions

When we say *format sdxy as ext4*, we actually means *create ext4 filesystem on sdxy*.

```bash
# mkfs.ext4 -L "root" /dev/mapper/cryptvg-root
# mkfs.ext4 -m 0 -L "home" /dev/mapper/cryptvg-home
```

BOOT and EFI share USB partition */dev/sdc1* (VFAT).

# Mount Gentoo partitions

During *mount* and *umount*, try *lsblk*, *blkid* etc.

```bash
# mount -v -t ext4 /dev/mapper/cryptvg-root /mnt/gentoo
# mkdir -v /mnt/gentoo/{home,boot}
# mount -v -t ext4 /dev/mapper/cryptvg-home /mnt/gentoo/home
# umount -v /dev/sdc1
# mount -v -t vfat /dev/sdc1 /mnt/gentoo/boot
# mkdir /mnt/gentoo/boot/EFI
```

The directory name could be upper case 'EFI' or lower case 'efi'. Currently, 'EFI' works fine.

# stage tarbar

```bash
# cd /mnt/gentoo
# tar xvjpf /path/to/stage3-amd64-20150319.tar.bz2
```

# Chroot

```bash
# vgchange -a y vg, VG name can be ommited.
# cryptsetup luksOpen /dev/mapper/vg-crypt cryptroot, you could use key-file to decrypt vg-crypt as well.
# vgchange -a y vgcryptvg
# mount -v -t ext4 /dev/mapper/cryptvg-root /mnt/gentoo
# mount -v -t ext4 /dev/mapper/cryptvg-home /mnt/gentoo/home
# mount -v -t vfat /dev/sdc1 /mnt/gentoo/boot
```

If you return to LiveCD and try to chroot again, please execute the above six commands first. Then follow:

```bash
# cp -v -L /etc/resolv.conf /mnt/gentoo/etc/
# mount -v -t proc proc /mnt/gentoo/proc2
# mount -v --rbind /sys /mnt/gentoo/sys
# mount -v --rbind /dev /mnt/gentoo/dev
# mount -v --make-rslave /mnt/gentoo/sys
# mount -v --make-rslave /mnt/gentoo/dev
# chroot /mnt/gentoo /bin/bash
# source /etc/profile && export PS1="(chroot) $PS1"
```

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

1. grub2-install --target=x86_64-efi --efi-directory=/boot --boot-directory=/boot --bootloader-id=grub2 --removable --modules=part\_gpt
   - '--bootloader-id=grub2' and '/dev/sdc' (USB stick) might be optional.
   - '--efi-directory' specifies the mountpoint of the ESP (i.e. the USB sdc1 is mounted at /boot). It replaces '--root-directory' which is deprecated.
   -  No need to append '/dev/sdc' since '--efi-directory=' is enough. The 'INSTALL_DEVICE' parameter of 'grub2-install' is mainly for BIOS boot.

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
2. rc-service lvmetad start

   After getting into new Gentoo system, *grub2-mkconfig* might complain about *lvmetad* issue which does NO harm. If you really want to get rid of the warning, just run *rc-service lvmetad start* before *grub2-mkconfig* and remember to stop afterwards.
3. grub2-mkconfig -o /boot/grub/grub.cfg

   In chroot, grub2-mkconfig might fail to probe (by calling os-prober) Windows on HDD. When getting into Gentoo, execute it again to make up.

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

1. dd if=/dev/sdb2 | xz > boot-image-backup.xz, backup of boot and EFI shared partition
2. xzcat image-file.xz | dd of=/dev/sdb2, restore from backup

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
