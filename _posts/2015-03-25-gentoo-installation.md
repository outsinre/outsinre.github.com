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
4. `sudo su -` switches to `root` account. The command prompt is `livecd ~ #` which is not the same as the handbook one `root #`.
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
9. Setting the date and time using `date` command.
10. Go to the Gentoo mountpoint where the root file system is mounted (most likely /mnt/gentoo): `cd /mnt/gentoo`.
11. Downloading the stage tarball with Chrome application in LiveDVD. Go to [Installation media](https://www.gentoo.org/main/en/where.xml) and then to [amd64 multilib](http://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64/). You will find `stage3-amd64-20150319.tar.bz2`. Just download!
12. Verify the tarball integrity and compare the output with the checksums provided by the .DIGESTS or .DIGESTS.asc file.
    1. _#_ sha512sum /home/gentoo/Download/stage3-amd64-20150319.tar.bz2
13. Now unpack the downloaded stage onto the system. Attention: the current working directory is `/mnt/gentoo`.
    1. _#_ tar xvjpf stage3-amd64-20150319.tar.bz2. Usually, the stage file is not in /mnt/gentoo directory. So you need to specify the path to stage. For instance, /home/download/stage3...
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
    2. _#_ eselect profile set 3, choose the `desktop` profile, **Not** the `desktop/gnome` or `desktop/kde`. We will install `xfce` later on.
    3. For `USE` flag, use command `emerge --info | grep ^USE` to check the default flags for profile selected above.
    4. Refer to [xfce HOWTO](https://wiki.gentoo.org/wiki/Xfce/HOWTO#The_basics) about the USE flags needs excluded. Use `emerge --info | grep XXX` to test each flag whether `/etc/portage/make.conf` needs updated. At last, I add `-qt4` and `thunar` to USE flag in make.conf file.

        ```
USE="-gnome -kde -minimal -qt4 dbus jpeg lock session startup-notification thunar udev X"
        ```

23. Set the timezone.
    1. _#_ ls /usr/share/zoneinfo
    2. _#_ echo "Asia/Shanghai" > /etc/timezone
    3. _#_ emerge --config sys-libs/timezone-data
24. Configure locales that the system supports.
    1. _#_ nano -w /etc/locale.gen
    2. Choose en-US ISO-8859-1, en_US.UTF-8 UTF-8, zh_CN.UTF-8 UTF-8 and zh_TW.UTF-8 UTF-8.
    3. _#_ locale-gen
    4. If reminds: run ". /etc/profile" to reload the variable in your shell". If you run it, you need to run `export PS1="(chroot) $PS1"` again.
25. Set the system-wide locale.
    1. _#_ eselect locale list
    2. _#_ eselect locale set 4, set system-wdie locale to en_US.utf8.
    3. _#_ env-update && source /etc/profile
    4. _#_ export PS1="(chroot) $PS1"
25. Install the kernel source.
    1. _#_ emerge --ask sys-kernel/gentoo-sources
    2. _#_ ls -l /usr/src/linux
26. Configuring the Linux kernel - Manual configuration.
    1. _#_ emerge --ask sys-apps/pciutils
    2. _#_ cd /usr/src/linux
    3. _#_ make menuconfig
26. Details on kernel configuration. Use the command `lspci -n` and paste it's output to [device driver check page](http://kmuto.jp/debian/hcl); that site gives you the kernel modules needed in general. Then go to kernel configuration (e.g. menuconfig) and press `/` to search the options like `e100e`, find their locations and activate them.
    1. `i915 e100e snd-hda-intel iTCO-wdt ahci i2c-i801 iwlwifi sdhci-pci`: these are the dirvers that needs activated, especially for the network drivers. When search "snd-hda-intel", replace the _hypen_: **-** with _dash_: **_**.
    2. When `sdhci_pci`, you get something like `... on PCI bus`, but you cannot find the item since its parent item `Secure Digital Host Controller Interface Support` is turned off by default. So turn it on first.
    3. Enable EFI stub support and EFI variables in the Linux kernel if UEFI is used to boot the system: `EFI stub support` and `EFI mixed-mode support` under `Processor type and features`.
    4. The search with `/` in `menuconfig` output is as follows. The `Prompt` part is usually which should be activated.

		```
Symbol:
Type:
Prompt:
  Location:
		```
    5. Refer to [Xorg configruation](https://wiki.gentoo.org/wiki/Xorg/Configuration#Installing_Xorg) to enable corresponding kernel model support. However, according to this reference, nothing needs updated for kernel configuration.
    6. Remove several `AMD` items under `Processor type and features`.
	7. Enable `NTFS` support to mount windows partition on demand. Refer to [NTFS wiki](https://wiki.gentoo.org/wiki/NTFS). You need `emerge --ask sys-fs/ntfs3g` to install `ntfs3g` package in step 42.
	8. For `iwlwifi`, remember to enable debug support: IWLWIFI\_DEBUG, IWLWIFI\_DEBUGFS, and IWLWIFI\_DEVICE_TRACING.
	9. Turn on `CONFIG_PACKET` to support wireless tool `wpa_supplicant` which will be installed later on.
	10. This link [wlan0-no wireless extensions (Centrino Advanced-N)](https://forums.gentoo.org/viewtopic-t-883211.html) offer ideas on how to find out the driver information.
    11. Reference links: [device driver check page](http://kmuto.jp/debian/hcl); [How do you get hardware info and select drivers to be kept in a kernel compiled from source](http://unix.stackexchange.com/a/97813); and [Working with Kernel Seeds](http://kernel-seeds.org/working.html).
27. Compiling and installing.
    1. _#_ make && make modules_install
    2. _#_ make install
    3. _deprecated #_ mkdir -p /boot/efi/boot
    4. _deprecated #_ cp /boot/vmlinuz-3.18.9-gentoo /boot/efi/boot/bootx64.efi
    5. _#_ emerge genkernel
    6. _#_ genkernel --install initramfs, The resulting file can be found by simply listing the files starting with initramfs: ls /boot/initramfs*.
27. Some drivers require additional firmware to be installed on the system before they work. This is often the case for network interfaces, especially wireless network interfaces.
    1. _#_ emerge --ask sys-kernel/linux-firmware
28. Creating the fstab file. The default `/etc/fstab` file provided by Gentoo is not a valid fstab file but instead more of a template.

	```
/dev/sda10   /boot        ext2    defaults,noatime     1 2
/dev/sda12   /	           ext4    noatime              0 1
/dev/sda7    none         swap	   sw                   0 0
	```
29. Set hostname.
    1. _#_ nano -w /etc/conf.d/hostname
    2. set hostname="x220gentoo"
30. Configuring the network.
	1. **DO NOT follow the handbook guide for network during installation**. We don't need `net-misc/netifrc` at all. `net-misc/netifrc` needs support of `dhcp`, while `net-misc/dhcpcd` can handle network configuration alone.
	2. _#_ emerge --ask net-misc/dhcpcd
	3. _#_ rc-update add dhcpcd default
	4. From now, the Ethernet part is OK. Nothing special needs configured. `dhcpcd` will manage Ethernet connection when startup. But for the Wireless part, we need to install another tool `net-wireless/wpa_supplicant`.
	5. _#_ emerge --ask net-wireless/wpa_supplicant
	6. wpa\_configuration: Wifi parameters should be put in `/etc/wpa_supplicant/wpa_supplicant.conf` file:

		```
# This command is to show the default configuration:
# bzcat /usr/share/doc/wpa_supplicant-2.2-r1/wpa_supplicant.conf.bz2 | less
# or http://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf
# Except eap and phase2 arguments, the rest are default values.
# This command is to test the wpa_supplicant configuration:
# wpa_supplicant -i wlp3s0 -D nl80211 -c /etc/wpa_supplicant/wpa_supplicant.conf -d
ctrl_interface=DIR=/var/run/wpa_supplicant
ctrl_interface_group=0
eapol_version=1
ap_scan=1
fast_reauth=1
network={
	ssid="sMobileNet"
	proto=WPA RSN
	key_mgmt=WPA-EAP
	pairwise=CCMP TKIP
	group=CCMP TKIP 
	eap=PEAP
	identity="XXXXXX"
	password="YYYYYY"
	ca_cert="/etc/ssl/certs/Thawte_Premium_Server_CA.pem"
	phase1="peaplabel=0"
	phase2="auth=MSCHAPV2"
}
		```

	7. When `wpa_configuration` is configured as above, `dhcpcd` will automatically connect to the `sMobileNet` through `wpa_supplicant`. No need to create so called `/etc/conf.d/net` file as the handbook.
	8. If you have installed `net-misc/netifrc` and created `/etc/ini.d/net.*` and `/etc/conf.d/net` files, refer to [Migration from Gentoo net.* scripts](
	https://wiki.gentoo.org/wiki/Network_management_using_DHCPCD#Migration_from_Gentoo_net..2A_scripts).
	9. In case the network interface card should be configured with a static IP address, entries can also be manually added to `/etc/dhcpcd.conf`.
	10. Reference: [Network management using DHCPCD](https://wiki.gentoo.org/wiki/Network_management_using_DHCPCD); [wpa_supplicant](https://wiki.gentoo.org/wiki/Wpa_supplicant); [configuration example](http://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf); [wpa_supplicant.conf for sMobileNet in HKUST](http://blog.ust.hk/yang/2012/09/21/wpa_supplicant-conf-for-smobilenet-in-hkust/); [wpa_supplicant.conf](http://www.freebsd.org/cgi/man.cgi?wpa_supplicant.conf).
31. Set root password: _#_ passwd
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
36. NTFS: emerge --ask sys-fs/ntfs3g
36. Remote access: rc-update add sshd default
38. Configuring the bootloader. Refer to [GRUB2 Quick Start](https://wiki.gentoo.org/wiki/GRUB2_Quick_Start).
    1. Add `GRUB_PLATFORMS="efi-64"` to `/etc/portage/make.conf`. This step must occur before installing the grub package. Otherwise it would show `error: /usr/lib/grub/x86_64-efi/modinfo.sh doesn't exist`.
    2. _#_ emerge --ask sys-boot/grub
    3. Delete the folder *boot* and  file *bootx64.efi* created in step 27.[3,4], while keep the `/boot/efi` directory. Then `mount /dev/sda2 /boot/efi`. `/dev/sda2` is the `EFI partition system`. Because Gentoo, Ubuntu, Windows share the EFI partition, so we should mount the shared EFI partion here. Not just create a private EFI environment in Gentoo's private boot partition. **This step is really important!**
    4. To install GRUB2 on an EFI capable system `grub2-install --target=x86_64-efi`.
39. Chainload into Ubuntu Grub2 `nano -w /etc/grub.d/40_custom`, add the code below.
    1. The traditional `chainloader +1` does work for UEFI boot.

		```
menuentry "UEFI GRUB2 UBUNTU 14.04 on /dev/sda2" {
	insmod fat
	insmod part_gpt
	insmod chain
	set root='hd0,gpt2'
	chainloader (${root})/EFI/ubuntu/grubx64.efi
}
		```

40. To generate the final GRUB2 configuration: `grub2-mkconfig -o /boot/grub/grub.cfg`.
41. [optional] You can install `xorg` and `xfce` now without reboot below. Reboot is just for basic system test.
41. Exit the chrooted environment: `exit` and unmount all mounted partitions:

    ```
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount /mnt/gentoo{/boot,/sys,/proc,}
    ```
You may reminded that some device is busy. Just let it go. Then type in that one magical command that initiates the final, true test: `reboot`.
    1. When rebooting, if the LiveDVD usb stick is still plugged onto the computer, the chainload to Ubuntu grub does not work. It show _error: disk hd0,gpt2 not found_. This is because the grub2 treats the USB stick as _hd0_ while the hard disk as _hd1_. You can unplug the USB, and CTRL+ALT+DEL. Another way is to edit the Ubuntu grub2 chainlaod menu from _hd0_ to _hd1_, then press F10 to boot.
    2. The very first thing after rebooting is to create a regular user account:

        ```
useradd -g users -G wheel video -m zachary
passwd zachary
        ```

42. From now on, a basic gentoo system is installed. You can login with `root` account. Probably, the new system cannot connect to the Wifi network (lack in network manager). You can `chroot` again into the gentoo system when installing new package:
    1. Boot with LiveDVD
    2. _#_ swapon /dev/sda7
    3. _#_ mount /dev/sda12 /mnt/gentoo
    4. _#_ mount /dev/sda10 /mnt/gentoo/boot
    5. _#_ cp -L /etc/resolv.conf /mnt/gentoo/etc
    6. _#_ mount -t proc proc /mnt/gentoo/proc
    7. _#_ mount --rbind /sys /mnt/gentoo/sys
    8. _#_ mount --make-rslave /mnt/gentoo/sys 
    9. _#_ mount --rbind /dev /mnt/gentoo/dev 
    10. _#_ mount --make-rslave /mnt/gentoo/dev
    11. _#_ chroot /mnt/gentoo /bin/bash
    12. _#_ source /etc/profile
    13. _#_ export PS1="(chroot) $PS1"
    14. Now you can install `xorg` and `xfce` for gentoo system with the help of LiveDVD KDE wifi connection.
43. Xorg installaion.
    1. Refer to [Xorg/Configuration](https://wiki.gentoo.org/wiki/Xorg/Configuration).
    2. For the kernel support part, already done in previous step.
    3. Add the following lines into `/etc/portage/make.conf`:

        ```
## (For mouse, keyboard, and Synaptics touchpad support)
INPUT_DEVICES="evdev synaptics"
## (For intel cards)
VIDEO_CARDS="intel"
        ```
    4. _#_ emerge --ask --verbose --pretend x11-base/xorg-drivers, check the dependency.
    5. _#_ echo "x11-base/xorg-server udev" >> /etc/portage/package.use/xorg-server. Actually this step is unnecessary since `udev` is enabled by default when selecting the system profile in previous step.
    6. _#_ emerge --ask x11-base/xorg-server
    7. _#_ env-update
    8. _#_ source /etc/profile
    9. _#_ export PS1="(chroot) $PS1"
    10. The official wiki suggests installing `x11-wm/twm` and `x11-terms/xterm` to test `xorg` installation. However, we are currently chrooting, startx is already running supporting the LiveDVD KDE environment. Hence, we cannot test by issuing command `startx` in chroot environment. It's only possible when reboot into the genuine gentoo system. So skip this step.
    11. For this command: echo XSESSION="Xfce4" > /etc/env.d/90xsession, we have not installed `xfce` yet. So leave it for the next step.
44. Xfce installation & Configuration.
    1. Refer to [Xfce](https://wiki.gentoo.org/wiki/Xfce) for installation and [Xfce/HOWTO](https://wiki.gentoo.org/wiki/Xfce/HOWTO) for configuration.
    2. _#_ eselect profile list, you will find `…/desktop` is the default profile (not `…/gnome` or `…/kde`).
    3. [optional] _#_ echo 'app-text/poppler -qt4' >> /etc/portage/package.use/poppler, since `-qt4` is already set globally in previous step when installing the basic gentoo system.
    4. [optional] _#_ echo 'dev-util/cmake -qt4' >> /etc/portage/package.use/cmake
    3. _#_ echo 'gnome-base/gvfs -http' >> /etc/portage/package.use/gvfs
    4. _#_ echo 'XFCE_PLUGINS="brightness clock trash"' >> /etc/portage/make.conf
    5. **Attention** _#_ emerge --ask xfce4-meta xfce4-notifyd; emerge --deselect y xfce4-notifyd, the 1st reference mixed this command order with step 4.
    6. _#_ emerge --ask x11-terms/xfce4-terminal
    7. Installation finished. Now reboot and loggin with the regular account to configure xfce.
    8. _$_ emerge --search consolekit, you can see consolekit is installed. So follow the 2nd reference:
    9. _$_ echo "exec startxfce4 --with-ck-launch" > ~/.xinitrc
    10. _$_ rc-update add consolekit default
	11. _#_ echo XSESSION="Xfce4" > /etc/env.d/90xsession, refer to the 11th item in previous step.
    12. You'd better logout and then login again to test xfce: _$_ startx.
45. When you get into the xfce desktop, you may found many unnecessary disk icons on the desktop or thunar sidebar. It's annoying. Use `udev, udisks` utility.
    1. _#_ nano - /etc/udev/rules.d/99-hide-disks.rules
	2. put the following code:
	
	    ```
KERNEL=="sdaXY", ENV{UDISKS_IGNORE}="1"
        ```
	  `XY` is the disk partition number you would like to hide. As noted in the reference below, `UDISKS_PRESENTATION_HIDE` is deprecated and replaced by `UDISKS_IGNORE`.
	3. Refer to [udev 99-hide-disks.rules is no longer working](http://superuser.com/questions/695791/udev-99-hide-disks-rules-is-no-longer-working).
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
44. Work to do:
    1. firefox browser
