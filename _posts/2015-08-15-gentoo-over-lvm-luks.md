---
layout: post
title: Gentoo rootfs over LVM encrypted in LUKS container.
---

# System scheme

1. '*/boot*' partition resides on USB stick, say *sdc1* partition.
2. '*/boot/efi*' is shared among Windows and Gentoo.
3. *rootfs* ('*/, swap, /home*') mountpoints were created op top of LVM group, say *vg1*.
4. *vg1* LVM group lays over *Dm-Crypt, LUKS* encrypted sda8*.
5. Use *keyfile* to encrypt *rootfs*. *keyfile* isteself was encrypted by *GnuPG* and placed at USB stick, say *sdc1* (same as */boot*).
6. The installation process depends mainly on Gentoo LiveCD.

# Prepare partitions

Use `parted -a optimal /dev/sda` to get a disk partition for *rootfs*. I will use *rootfs* for '*/, swap, /home*' mountpoints in this post.

>Run the very first command `unit s` in *parted* to display disk partitions as *sector*. When creating partition with `s` unit, it will be well aligned.

For *parted* example, refer to [gentoo installation](http://www.fangxiang.tk/2015/03/25/gentoo-installation/) and [kali usb persistence](http://www.fangxiang.tk/2015/07/23/kali-usb-persistence/).
    
Say '*/dev/sda8*' is created for *rootfs*.

Use `parted -a optimal /dev/sdc` to create '*/boot*' partition on USB stick '*/dev/sdc1*'.

'*/boot*' is *NOT* encrypted! You can use the whole USB space to hold '*/boot*', or just spare around 256MB parition.

Then formate the partition as *ext2*:

```
mkfs.ext2 -L 'usb boot' /dev/sdc1
```

# cryptsetup keyfile

## dm-crypt and LUKS

**dm-crypt** is a disk encryption system using the kernels *crypto API* framework and *device mapper* subsystem.

With *dm-crypt*, administrators can encrypt entire disks, logical volumes, partitions, but also single files.

The *dm-crypt* subsystem supports the *Linux Unified Key Setup* (LUKS) structure, which allows for multiple keys to access the encrypted data, as well as manipulate the keys (such as changing the keys, adding additional passphrases, etc.)

Although *dm-crypt* supports *non*-LUKS setups as well, this article will focus on the LUKS functionality mostly due to its flexibility, manageability as well as broad support in the community.

Refer to [Dm-crypt](https://wiki.gentoo.org/wiki/Dm-crypt) and [Dm-Crypt LUKS](https://wiki.gentoo.org/wiki/DM-Crypt_LUKS).

## Keyfile

We use `cryptsetup` command from LiveCD to encrypt *rootfs* partition '*/dev/sda8*'. We can use **passphrase** or **keyfile** as encryption key.

Considering the length, randomness and complexity of encryption requirements a *keyfile* seems to be the right spot.

We first generate a ramdom *keyfile* for *rootfs* encryption. This *keyfile* is usually put at a secure place, i.e. USB stick.

>In this scheme, *keyfile* and '*/boot*' share the same USB stick partition.

If attacker gets the USB stick, system is decrypted!! So next, we need to encrypt this *keyfile* by **GnuPG** before storing it on USB stick.

```bash
# mkdir /mnt/sdc1
# mount /dev/sdc1 /mnt/sdc1
# mkdir /dev/sdc1/key
$ dd if=/dev/urandom bs=8388607 count=1 | gpg --symmetric --cipher-algo AES256 --output ~/Desktop/luks-key.gpg
# cp /path/to/luks-key.gpg /mnt/sdc1/key/
```

Refer to [Preparing the LUKS-LVM Filesystem and Boot USB Key](https://wiki.gentoo.org/wiki/Sakaki%27s_EFI_Install_Guide/Preparing_the_LUKS-LVM_Filesystem_and_Boot_USB_Key#Creating_a_Password-Protected_Keyfile_for_LUKS).

You will be asked for passphrase to encrypt *keyfile*. Don't forget passphrase!

If you run `gpg` in LiveCD *root* shell, you might get the following error even though you set 'export GPG_TTY=$(tty)' as the reference mentioned:

```
gpg: directory `/root/.gnupg' created
gpg: new configuration file `/root/.gnupg/gpg.conf' created
gpg: WARNING: options in `/root/.gnupg/gpg.conf' are not yet active during this run
gpg: keyring `/root/.gnupg/pubring.gpg' created
No protocol specified

(pinentry:10992): Gtk-WARNING **: cannot open display: :0
gpg-agent[10991]: can't connect to the PIN entry module: End of file
gpg-agent[10991]: command get_passphrase failed: No pinentry
gpg: problem with the agent: No pinentry
gpg: error creating passphrase: Operation cancelled
gpg: symmetric encryption of `[stdin]' failed: Operation cancelled
```

Refer to [Genkernel initramfs with LUKS+GPG key](https://forums.gentoo.org/viewtopic-p-7485020.html) and [Using gpg in mkinitcpio hook, gpg doesn't ask for a password](https://bbs.archlinux.org/viewtopic.php?id=120181).

This might be related to *>app-crypt/gnupg-2.0* version and *pinentry* support. To get this issue solved:

1. Generate *keyfile* in another workable environment.
2. Switch to normal user account shell when run `gpg` command and get he decrypted temporary *luks-key* file for LiveCD.

# Format *rootfs* using LUKS

```bash
$ gpg --decrypt /mnt/sdc1/key/luks-key.gpg > ~/Desktop/luks-key
# cryptsetup --cipher serpent-xts-plain64 --key-size 512 --hash sha512 --key-file /path/to/luks-key luksFormat /dev/sda8
```

Pay attention to `--cipher serpent-xts-plain64 --key-size 512 --hash sha512` which reminds us the related kernel option must be 'Y' **NOT** 'M'.

Check the formating effect:

```bash
# cryptsetup luksDump /dev/sda8
```

## Adding a Fallback Passphrase (Optional Step)

If we lost the *gpg* passphrase or the *keyfile*, we lost the entire system data. Besides the *keyfile*, we add a fallback passphrase.

```bash
# cryptsetup --key-file /path/to/luks-key luksAddKey /dev/sda8
# cryptsetup luksDump /dev/sda8
```

# Creating the LVM Structure on Top of LUKS

```bash
# cryptsetup --key-file /path/to/luks-key luksOpen /dev/sda8 gentoo
# ls /dev/mapper
# pvcreate /dev/mapper/gentoo
# vgcreate vg1 /dev/mapper/gentoo
# lvcreate --size 4G --name swap vg1
# lvcreate --size 18G --name root vg1
# lvcreate --extents 100%FREE --name home vg1
# pvdisplay
# vgdisplay
# lvdisplay
# vgchange --available y
# lvscan
# ls /dev/mapper
```

# Formatting and Mounting the LVM Logical Volumes (LVs)

```bash
# mkswap -L "swap" /dev/mapper/vg1-swap
# mkfs.ext4 -L "root" /dev/mapper/vg1-root
# mkfs.ext4 -m 0 -L "home" /dev/mapper/vg1-home
# swapon -v /dev/mapper/vg1-swap
# mount -v -t ext4 /dev/mapper/vg1-root /mnt/gentoo
# mkdir -v /mnt/gentoo/{home,boot}
# mount -v -t ext4 /dev/mapper/vg1-home /mnt/gentoo/home
# umount -v /dev/sdc1
# mount -v -t ext2 /dev/sdc1 /mnt/gentoo/boot
# mkdir /mnt/gentoo/boot/efi
# mount -v /dev/sda2 /mnt/gentoo/boot/efi
```

# Chroot for installing

Final preparation:

```bash
# mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
# mkdir -p -v /mnt/gentoo/etc/portage/repos.conf
# cp -v /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
# nano -w /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

Make *gentoo.conf* contents as:

```
[DEFAULT]
main-repo = gentoo

[gentoo]
location = /usr/portage
sync-type = rsync
sync-uri = rsync://rsync.cn.gentoo.org/gentoo-portage
auto-sync = yes

# for daily squashfs snapshots
#sync-type = squashdelta
#sync-uri = mirror://gentoo/../snapshots/squashfs
```

The old portage sync system use *SYNC=rsync://rsync.cn.gentoo.org/gentoo-portage* in *make.conf*. In new >portage-2.2.16, it is *sync-url* in *gentoo.conf*. To get the *sync-url* value:

```bash
# mirrorselect -i -r -o | sed 's/^SYNC=/sync-uri = /;s/"//g' >> /mnt/gentoo/etc/portage/repos.conf/gentoo.conf 
```

Finally *chroot* into new environment:

```bash
# cp -v -L /etc/resolv.conf /mnt/gentoo/etc/
# chroot /mnt/gentoo /bin/bash
# source /etc/profile && export PS1="(chroot) $PS1"
```

<s> After entering *chroot* environment, the very first is to mask GnuPG 2.0. Refer to *Tips -> GnuPG* at the end of post. </s>

## Return to *chroot* environment

```bash
# cryptsetup --key-file=/path/to/luks-key luksOpen /dev/sda8 gentoo
# vgchange --available y
# lvscan
# ls /dev/mapper/
# swapon -v /dev/mapper/vg1-swap
# mount -v -t ext4 /dev/mapper/vg1-root /mnt/gentoo
# mount -v -t ext4 /dev/mapper/vg1-home /mnt/gentoo/home
# mount -v -t ext2 UUID=63053222-d5f9-43af-a7ce-76f0ca5a14ed /mnt/gentoo/boot
# mount -v /dev/sda2 /mnt/gentoo/boot/efi
# cp -v -L /etc/resolv.conf /mnt/gentoo/etc/
# mount -v -t proc proc /mnt/gentoo/proc
# mount -v --rbind /sys /mnt/gentoo/sys
# mount -v --rbind /dev /mnt/gentoo/dev
# mount -v --make-rslave /mnt/gentoo/sys
# mount -v --make-rslave /mnt/gentoo/dev
# chroot /mnt/gentoo /bin/bash
# source /etc/profile && export PS1="(chroot) $PS1"
```

# *initramfs* and *grub2*

After getting into *chroot* environment, the most procedures, pleae refer to [gentoo installation](http://www.fangxiang.tk/2015/03/25/gentoo-installation/).

The main difference from that link, is *genkernel* and *grub2*.

## Kernel option support

From the above, we use `--cipher serpent-xts-plain64` and ` --hash sha512` which reminds us the related kernel option must be **Y** instead of 'M'.

>CRYPTO_SERPENT Y

>CRYPTO_SHA512 Y

Why they are should be 'Y' instead of 'M'? The boot process is basically:

1. Power on;
2. EFI firmware find the default bootloader Gentoo *grub2* at '*/dev/sda2/EFI/gentoo/grub64.efi*';
3. *grub2* launch *initramfs* into RAM;
4. *initramfs* found *GnuPG keyfile* at '*/boot/key*' and ask user for decryption; decrypted *keyfile* is used to decrypt *rootfs*.
5. Control pass to *init* scripts. After that, kernel modules loaded.

You find that if the two kernel options compiled as modules 'M', then you cannot pass the 4th step. No *serpent* and *sha512* function for *keyfile* decryption. This is the error message at boot:

>http://pastebin.com/fZgMwyNw

## genkernel

```bash
# echo "sys-kernel/genkernel cryptsetup" > /etc/portage/package.use/genkernel
# emerge -av genkernel
# genkernel --lvm --luks --gpg --busybox --install initramfs
```

You see *genkernel* takes *lvm*, *luks*, *gpg*, and *busybox* parameter. Don't worry about the `gpg` version issue mentioned earlier! `man genkernel` shows it includes *GnuPG 1.x*:

```
--[no-]gpg
           Includes or excludes support for GnuPG 1.x, the portable standalone
           branch of GnuPG. A key can be made from gpg --symmetric -o
           /path/to/LUKS-key.gpg /path/to/LUKS-key . After that, re-point the
           root_key argument to the new .gpg file.
```

Some posts mention `emerge -av app-crypt/gnupg lvm2 busybox cryptsetup` in the *chroot* system before *genkernel*. Basically, these tools are only needed in LiveCD and 'genkernel'. In our new Gentoo system, it is not a must.

*sys-kernel/genkernel cryptsetup* USE flag will draw in *cryptsetup*, and then *lvm2* in *chroot*, while *busybox* is an essential *@system* package which is included in *stage3* tar bar. You can run `equery u busybox`, and find *static* USE flag. Try:

```bash
# emerge -pvc cryptsetup
```

It will reminds:

```
Calculating dependencies... done!
  sys-fs/cryptsetup-1.6.5 pulled in by:
    sys-kernel/genkernel-3.4.49.2 requires sys-fs/cryptsetup

>>> No packages selected for removal by depclean
Packages installed:   549
Packages in world:    38
Packages in system:   44
Required packages:    549
Number to remove:     0
```

There is no need to `rc-update add lvm boot`, for this is only useful when there is any other LVM devices like external removable LVM disks, USB stick etc.

## Grub2

The core is to set the correct `GRUB_CMDLINE_LINUX` parameters for *kernel* and *init* scripts. Some of those parameters are indeed consumed by the kernel (a reasonable complete list of which is provided by kernel.org); however, others are realy targeted at the *init* script. The Linux kernel passes the *init* script (or program) any parameters it has not already rocessed as arguments, and the *init* script can also read the full command line in any event, via /proc/cmdline.

```
# nano -w /etc/default/grub
GRUB_CMDLINE_LINUX="crypt_root=UUID=8e105495-5a45-4297-b6ad-6e2d97abb461 dolvm root=/dev/mapper/vg1-root rootfstype=ext4 root_keydev=UUID=63053222-d5f9-43af-a7ce-76f0ca5a14ed root_key=key/luks-key.gpg"
```

1. crypt_root: this tells *init* (provided by *genkernel*) script which partition it should attempt to decrypt with *cryptsetup*. It is set to UUID of the LUKS formatted partition, say *sda8*.
2. dolvm: activate LVM volumes on bootup.
3. rootfstype: root file system type.
4. root: the location of the root filesystem to the kernel.
5. root\_keydev: if necessary provide the name of the device that carries the root_key. If unset while using root_key, it will automatically look for the device in every boot. It specifies the device path of device on which the *keyfile* is located. In this scheme, it is the '*/boot*' USB stick.
6. root_key: In case your root is encrypted with a key, you can use a device like a usb pen to store the key. This value should be the key path relative to the mount point. In this scheme, it is in the sub-directory *key/luks-key.gpg*.

   If it has a `.gpg` extension (as our case), the *init* script will treat it as being a *gpg* encrypted *keyfile*, and prompt for a passphrase to unlock the *keyfile* first (either textually at the console, or using the splash screen manager if the system sets that).
7. Here we use UUID instead of device file path '*/dev/sda8*' or '*/dev/sdc1*'. Since UUID is unique while device path name might change. For example, if current PC is plugged in 3 usb sticks as '*/dev{/sdb,/sdc,/sdd}*' and none of them is the boot stick, *initramfs* even cannot find the *gpg keyfile* location.
8. More about these command please read `man genkernel` and *kernel.org*.

```bash
# grub2-mkconfig -o /boot/grub/grub.cfg
```

In *chroot* environment, *grub2-mkconfig* and  *os-prober* might fail to generate Windows menu. Don't worry! You can use the following template to edit '*/etc/grub.d/40_custom*'. Or run *grub2-mkconfig* again when log into the real new system.

```bash
menuentry 'Windows Boot Manager (on /dev/sda2)' --class windows --class os $menuentry_id_option 'osprober-efi-DAD1-F557' {
        insmod part_gpt
        insmod fat
        set root='hd0,gpt2'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt2 --hint-efi=hd0,gpt2 --hint-baremetal=ahci0,gpt2  DAD1-F557
        else
          search --no-floppy --fs-uuid --set=root DAD1-F557
        fi
        chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
```

# fstab

```
# UUID of usb stick
UUID="aaa-bbb-ccc-ddd..."	/boot	ext2	noauto,noatime	1 2
/dev/sda2	/boot/efi	vfat	noauto,noatime	1 0
/dev/mapper/vg1-root	/	ext4	noatime,errors=remount-ro	0 1
/dev/mapper/vg1-swap	none	swap	sw	0 0
/dev/mapper/vg1-home	/home	ext4	defaults	0 0
/dev/sda1	/mnt/Recovery	ntfs-3g		noauto,ro	0 0
/dev/sda4	/mnt/Win81	ntfs-3g		noauto,ro	0 0
/dev/sda5	/media/Data	ntfs-3g		noauto,nls=utf8,locale=zh_CN.utf8,uid=zachary,gid=users,dmask=022,fmask=133	0 0
/dev/sda6	/media/Misc	ntfs-3g		nls=utf8,locale=zh_CN.utf8,uid=zachary,gid=users,dmask=022,fmask=133	0 0
/dev/sda7	/media/WLshare	ntfs-3g		nls=utf8,locale=zh_CN.utf8,uid=zachary,gid=users,users,dmask=022,fmask=133	0 0
```

# Get out of *chroot*

```bash
# exit
# swapoff -v /dev/mapper/vg1-swap
# umount -lv /mnt/gentoo/home
# umount -lv /mnt/gentoo/boot{/eif,}
# umount -lv /mnt/gentoo/dev{/shm,/pts,}
# umount -lv /mnt/gentoo{/proc,/sys,}
# vgchange --available n
# cryptsetup luksClose gentoo
# reboot
```

# Tips

1. mtab

   ```bash
   # ln -vsf /proc/self/mounts /etc/mtab
   ```

2. XSESSION

   When emerging *xorg* and *xfce4*, the wiki recommends setting:

   ```bash
   # echo XSESSION="Xfce4" > /etc/env.d/90xsession
   ```

   for all users on the system. This is not a good idea since *root* account can run X as well, which is NOT a good scheme.

   So remove '*/etc/env.d/90xsession*' if exists.
3. Editor

    ```bash
    # eselect editor set "/usr/local/bin/ect"
    ```
    Run *eselect* as root set the system default editor for all users. Of course, you can also use method in [emacs configuration](http://www.fangxiang.tk/2014/07/12/emacs-configuration/).
4. [outdated] GnuPG

   > This is outdated.

   Though not necessary to emerge GnuPG in the new system. But later on, package *layman* will draw in *git* which in turn draws in GnuPG. As mentioned, *>app-crypt/gnupg-2.0" will cause *pinentry* issue.

   ```bash
   # echo "=app-crypt/gnupg-2*" > /etc/portage/package.mask/gnupg
   ```

   This should be the 1st thing after entering *chroot*.

# Reference

1. http://ceyes.github.io/2015-06/Gentoo-Setup-LVM-over-LUKS/
1. https://wiki.gentoo.org/wiki/Sakaki%27s_EFI_Install_Guide/Preparing_the_LUKS-LVM_Filesystem_and_Boot_USB_Key
1. http://www.gentoo-wiki.info/SECURITY_System_Encryption_DM-Crypt_with_LUKS
1. http://blog.marek.sapota.org/article/2012/10/07/installing-gentoo-on-an-encrypted-partition.html
1. http://iswwwup.com/t/c1ced3aa98c4/encryption-luks-storing-keyfile-in-encrypted-usb-drive.html
1. http://www.gossamer-threads.com/lists/gentoo/user/300698
1. http://unix.stackexchange.com/questions/183666/booting-gentoo-on-lvm-inside-luks-with-gpg-encrypted-keyfile
1. https://bbs.archlinux.org/viewtopic.php?id=120181
1. https://forums.gentoo.org/viewtopic-p-7485020.html
