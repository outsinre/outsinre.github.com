---
layout: post
title: Gentoo Installation
---
> This is the procedures of Gentoo UEFI installation along with Windows 8.1 and Ubuntu 14.04.

1. The official [handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64) is important but really out of date.
2. Download the `LiveDVD` as *livedvd-amd64-multilib-20140826.iso* instead of the so called `Minimal installation CD` as *install-amd64-minimal-20150319.iso*.
    1. The minimal CD cannot generates UEFI bootable USB stick.
    2. The LiveDVD support KDE desktop environment among others. Specially the internet connection setting is much easier during the installation.
2. Use `Rufus` to create an UEFI USB stick. Similar tools are:
    1. UNetbootin;
    2. Universal USB Installer;
    3. *diskpart* terminal command;
    4. Just copy the ISO contents into a FAT32 format USB stick; 
    5. Ubuntu disk creater.
3. Boot with the USB stick into the default KDE desktop environment. There will be screen ask for user name `gentoo`'s password for loggin. Just wait for a while.
    1. Default user and password are both *gentoo*. Use `sudo su -` command to switch to `root`. You can use `passwd` to change the password for the user you are loggined into. As root, you can change ay user passworld by issuing the command `passwd username`.
    2. [Gentoo Ten LiveDVD Frequently Asked Questions](https://www.gentoo.org/proj/en/pr/releases/10.0/faq.xml).
4. `sudo su -` switches to `root` account. The command prompt is _livecd ~ #_ which is not the same as the handbook one _root #_.
5. `fdisk /dev/sda` or `parted -a optimal /dev/sda` (default one) checks the current disk partition scheme. Choose and free up the `/dev/sda10` NTFS partition for Gentoo.
    1. _#_ parted -a optimal /dev/sda
    2. _#_ p
    3. _#_ unit MB
    3. _#_ rm 10
	3. _#_ p
    4. _#_ mkpart primary 309921MB 310049MB, create a boot partition sda10 for Gentoo
    5. _#_ p
    6. _#_ name 10 'Gentoo boot partition'
    7. _#_ mkpart primary 310049MB -1, create root partition sda12 for Gentoo
    8. _#_ p
    9. _#_ name 12 'Gentoo root partition'
    10. _#_ p
    11. The annoying thing is that the partition `Type` is `Basic data partition` when checking with `fdisk /dev/sda`. We can change it by Disk GUI application in Ubuntu.
6. `/dev/sda10` will be the boot partition while `/dev/sda12` the root partition. We don't need to create `swap` or `efi` partition since we already created it when installing Ubuntu or Windows.
7. Up to now, only the boot and root partition is prepared. We share swap and EFI partitions with Ubuntu and Windows. Now format the new partition. It's better to format boot partition as `ext2`.
    1. _#_ mkfs.ext2 /dev/sda10
    2. _#_ mkfs.ext4 /dev/sda12
8. From step 5 we know the Ubuntu swap partition is `/dev/sda7`. So we need to activate it: `swapon /dev/sda7`.
9. Mount the ewnly created partitions into the LiveDVD USB stick. Make sure the `gentoo` directory exists in /mnt/gentoo, otherwise create one.
    1. _#_ mount /dev/sda12 /mnt/gentoo
    2. _#_ mkdir /mnt/gentoo/boot
    3. _#_ mount /dev/sda10 /mnt/gentoo/boot
10. Go to the Gentoo mountpoint where the root file system is mounted (most likely /mnt/gentoo): `cd /mnt/gentoo`.
11. Downloading the stage tarball with Chrome application in LiveDVD. Go to [Installation media](https://www.gentoo.org/main/en/where.xml) and then to [amd64 multilib](http://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64/). You will find `stage3-amd64-20150319.tar.bz2`. Just download!
12. Verify the tarball integrity and compare the output with the checksums provided by the .DIGESTS or .DIGESTS.asc file.
    1. _#_ sha512sum /home/gentoo/Download/stage3-amd64-20150319.tar.bz2
13. Now unpack the downloaded stage onto the system. Attention: the current working directory is `/mnt/gentoo`.
    1. _#_ tar xvjpf stage3-amd64-20150319.tar.bz2
    2. Make sure that the same options `xvjpf` are used. The x stands for Extract, the v for Verbose to see what happens during the extraction process (optional), the j for Decompress with bzip2, the p for Preserve permissions and the f to denote that we want to extract a File, not standard input.
14. Configuring compile options. To keep the settings, Portage reads in the /etc/portage/make.conf file, a configuration file for Portage.
    1. _#_ emacs /mnt/gentoo/etc/portage/make.conf
15. CFLAGS and CXXFLAGS. Check your CPU architecture to set `-march=` parameter. Refer to [Intel](https://wiki.gentoo.org/wiki/Safe_CFLAGS#Intel). Command `grep -m1 -A3 "vendor_id" /proc/cpuinfo` will show the current CPU architecure information. That link also teach you how to precisely detect `-march=` parameter by touching, compiling and comparing two _.gcc_ files.
    1. CFALGS="-march=corei7-avx -O2 -pipe"
    2. CXXFLAGS="${CFLAGS}"
16. The `MAKEOPTS` variable defines how many parallel compilations should occur when installing a package. Usually this value is number of cpu cores + 1.
    1. Add a line `MAKEOPTS="-j3"`
17. Selecting mirrors.
    1. _#_ `mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf` choose the neareast mirror for kernal source code downloading.
    2. _#_ `mirrorselect -i -r -o >> /mnt/gentoo/etc/portage/make.conf` selects the `rsync server` to use when updating the portage tree. It is recommended to choose a _rotation link_, such as _rsync.us.gentoo.org_, rather than choosing a single mirror. This helps spread out the load and provides a fail-safe in case a specific mirror is offline.
18. _#_ `cp -L /etc/resolv.conf /mnt/gentoo/etc/`  to ensure that networking still works even after entering the new sda12 (/mnt/gentoo) environment. `/etc/resolv.conf` contains the name servers for the network. DON'T forget the `-L` parameter when copying.
19. Mounting the necessary filesystems.
    1. _#_ mount -t proc proc /mnt/gentoo/proc
    2. _#_ mount --rbind /sys /mnt/gentoo/sys
    3. _#_ mount --make-rslave /mnt/gentoo/sys
    4. _#_ mount --rbind /dev /mnt/gentoo/dev
    5. _#_ mount --make-rslave /mnt/gentoo/dev
20. Entering the new environment.
    1. _#_ chroot /mnt/gentoo /bin/bash
    2. _#_ source /etc/profile
    3. _#_ export PS1="(chroot) $PS1", this create a new different command prompt for the new environment.
21. Installing a portage snapshot: _#_ `emerge-webrsync`.
22. Choosing the right profile.
    1. _#_ eselect profile list
    2. _#_ eselect profile set 4, install the Gnome desktop profile.
    3. In `/etc/portage/make.conf` add `-qt4 -kde` into `USE` variable. Actually this step is not necessary. When selecting to install Gnome desktop, KDE support is removed automatically.
23. Set the timezone.
    1. _#_ ls /usr/share/zoneinfo
    2. _#_ echo "Asia/Shanghai" > /etc/timezone
    3. _#_ emerge --config sys-libs/timezone-data
24. Configure locales that the system supports.
    1. _#_ nano -w /etc/locale.gen
    2. Choose en-US ISO-8859-1, en_US.UTF-8 UTF-8, zh_CN.UTF-8 UTF-8 and zh_TW.UTF-8 UTF-8.
    3. _#_ locale-gen
    4. It will remind: run ". /etc/profile" to reload the variable in your shell". If you run it, you need to run `export PS1="(chroot) $PS1"` again.
25. Set the system-wide locale.
    1. _#_ eselect locale list
    2. _#_ eselect locale set 4, set system-wdie locale to en_US.utf8.
    3. _#_ env-update && source /etc/profile
    4. _#_ export PS1="(chroot) $PS1"
26. Configuring the Linux kernel - Manual configuration.
    1. _#_ emerge --ask sys-apps/pciutils
    2. _#_ cd /usr/src/linux
    3. _#_ make menuconfig
    4. Enable EFI stub support and EFI variables in the Linux kernel if UEFI is used to boot the system: `EFI stub support` and `EFI mixed-mode support` under `Processor type and features`.
27. Compiling and installing.
    1. _#_ make && make modules_install
    2. _#_ make install
    3. _#_ mkdir -p /boot/efi/boot
    4. _#_ cp /boot/vmlinuz-3.18.9-gentoo /boot/efi/boot/bootx64.efi
    5. _#_ emerge genkernel
    6. _#_ genkernel --install initramfs, The resulting file can be found by simply listing the files starting with initramfs: ls /boot/initramfs.
28. Creating the fstab file. **The default `/etc/fstab` file provided by Gentoo is not a valid fstab file but instead more of a template**.
	>/dev/sda10   /boot        ext2    defaults,noatime     1 2
	>/dev/sda12   /	           ext4    noatime              0 1
	>/dev/sda7    none         swap	   sw                   0 0
29. Set hostname.
    1. _#_ nano -w /etc/conf.d/hostname
    2. set hostname="x220gentoo"
30. **??** /etc/conf.d/net
31. Set root password ro15ot
32. Edit `/etc/conf.d/hwclock` to set the clock options.
    1. set `clock=local`, this is important when dual boot with Windows.
33. System logger.
    1. _#_ emerge --ask app-admin/syslog-ng
    2. _#_ emerge --ask app-admin/logrotate
    3. _#_ rc-update add syslog-ng default
34. Cron daemon. A cron daemon executes scheduled commands. It is very handy if some command needs to be executed regularly (for instance daily, weekly or monthly).
    1. _#_ emerge --ask sys-process/cronie
    2. _#_ rc-update add cronie default
35. File indexing: emerge --ask sys-apps/mlocate
36. Remote access: rc-update add sshd default
37. Although optional, the majority of users will find that they need a DHCP client to get on their network. Please install a DHCP client. If this is forgotten, then the system might not be able to get on the network, and thus cannot install a DHCP client afterwards as well.
    1. emerge --ask net-misc/dhcpcd
38. Configuring the bootloader. Refer to [GRUB2 Quick Start](https://wiki.gentoo.org/wiki/GRUB2_Quick_Start).
    1. Add `GRUB_PLATFORMS="efi-64"` to `/etc/portage/make.conf`. This step must occur before installing the grub package. Otherwise it would show `error: /usr/lib/grub/x86_64-efi/modinfo.sh doesn't exist`.
    2. _#_ emerge --ask sys-boot/grub
    3. Del the folder *boot* and *bootx64.efi* created in step 27.4. Then `mount /dev/sda2 /boot/efi`. `/dev/sda2` is the `EFI partition system`. Because Gentoo, Ubuntu, Windows share the EFI partition, so we should mount the shared EFI partion here. Not just create a private EFI environment in Gentoo's private boot partition.
    4. To install GRUB2 on an EFI capable system `grub2-install --target=x86_64-efi`.
39. Chainload into Ubuntu Grub2 `nano -w /etc/grub.d/40_custom`, add the following lines:
	>menuentry "UEFI GRUB2 UBUNTU 14.04 on /dev/sda2" {
	>	insmod fat
	>	insmod part_gpt
	>	insmod chain
	>	set root='hd0,gpt2'
	>	chainloader (${root})/EFI/ubuntu/grubx64.efi
	>}
    1. The traditional `chainloader +1` does work for UEFI boot.
40. To generate the final GRUB2 configuration: `grub2-mkconfig -o /boot/grub/grub.cfg`.
41. Exit the chrooted environment `exit` and unmount all mounted partitions `umount -l /mnt/gentoo/dev{/shm,/pts,}` and `umount /mnt/gentoo{/boot,/sys,/proc,}`. Then type in that one magical command that initiates the final, true test: `reboot`.
    1. When rebooting, if the LiveDVD usb stick is still plugged onto the computer, the chainload to Ubuntu grub does not work. It show _error: disk hd0,gpt2 not found_. This is because the grub2 treats the USB stick as _hd0_ while the hard disk as _hd1_. You can unplug the USB, and CTRL+ALT+DEL. Another way is to edit the Ubuntu grub2 chainlaod menu from _hd0_ to _hd1_, then press F10 to boot.
42. `/dev/sda10` is the boot partition for Gentoo. But by default, it's not auto-mounted for security reason. Similarly, the /dev/sda2 is the EFI System Partition. It's also not automatic mounted to `/boot/efi` as well at startup.
43. Partitions:
    1. sda1 Windows recovery partition
    2. sda2 EFI partition
    3. sda3 windows reserved
    4. sda4 windows C
    5. sda5 Ubuntu boot
    6. sda6 Ubuntu root
    7. sda7 swap for Ubuntu and Gentoo
    8. sda8 Windows D
    9. sda9 windows E
    10. sda10 Gentoo boot
    11 sda11 Ubuntu home
    12. sda12 Gentoo home
44. Network settings/ gnome doesnot work.