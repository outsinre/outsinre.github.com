---
layout: post
title: Gentoo Installation
---
> Gentoo installation along with Windows 8.1 and Ubuntu 14.04 with UEFI booting.

1. The official [handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64) is important but some items really out of date. If possible, refer to the corresponding ArchWiki part which is pretty excellent and detailed.
2. Download the `LiveDVD` like *livedvd-amd64-multilib-20140826.iso* instead of the so called `Minimal installation CD` like *install-amd64-minimal-20150319.iso*.
    1. The minimal CD cannot generates UEFI bootable USB stick.
    2. The LiveDVD support KDE desktop environment. Specially Wifi internet connection setting is much easier during the installation. You can also surf the internet while installing.
2. Use `Rufus` to create an UEFI USB stick. Similar tools are:
    1. UNetbootin;
    2. Universal USB Installer;
    3. *diskpart* terminal command;
    4. Just copy the ISO contents into a FAT32 format USB stick; 
    5. Ubuntu disk creater.
3. Boot with the USB stick into the default KDE desktop environment. There will be screen ask for user name `gentoo`'s password for loggin. Just wait for a while. It will automatically log into the system.
    1. The very first thing is to connect to WIFI or Ethernet through networkmanager.
    2. Use shortcut `F12` to Open/Retract Yakuake terminal in KDE destop.
    3. Default user and password are both *gentoo*. Use `sudo su -` command to switch to `root`. You can use `passwd USERNAME` to change the password for the user you are loggined into. As root, you can change ay user passworld by issuing the command `passwd username`. All the password issue within the LiveCD environment is not persistent for the new Gentoo system unless that is operated in `Chroot` environment.
    4. Refer to [Gentoo Ten LiveDVD Frequently Asked Questions](https://www.gentoo.org/proj/en/pr/releases/10.0/faq.xml).
4. _#_ sudo su -, switches to `root` account. The command prompt is `livecd ~ #` which is not the same as the handbook one `root #`. Maybe this is derived from not setting a temporary root password.
5. _#_ fdisk /dev/sda or parted -a optimal /dev/sda (I use the later one), checks the current disk partition scheme. Choose and free up the `/dev/sda10` NTFS partition for Gentoo.
    1. __NOTE__: `parted` takes effect immediately for each command without final confirmation like `fdisk`. So pay attention to the partition start and end position.
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
6. New `/dev/sda10` will be the boot partition while `/dev/sda12` the root partition. We don't need to create `swap` or `efi` partition since we already created it when installing Ubuntu or Windows. Just share these two partitions. If possible, you can also create a separate home partition.
7. Up to now, only the boot and root partition is prepared. We share swap and EFI partitions with Ubuntu and Windows. Now format the new partition. It's better to format boot partition as `ext2`.
    1. _#_ mkfs.ext2 /dev/sda10
    2. _#_ mkfs.ext4 /dev/sda12
    3. [optional] _#_ mkfs.ext4 /dev/sdaxy, sdaxy is for home partition.
8. From step 5 we know the Ubuntu swap partition is `/dev/sda7`. So we need to activate it:
    1. If you need to create a new swap partition, use command `mkswap /dev/sdaXY` to format the partition.
    2. _#_ swapon /dev/sda7
9. Mount the ewnly created partitions into the LiveDVD USB stick. Make sure the `gentoo` directory exists in /mnt/gentoo, otherwise create one.
    1. _#_ mount /dev/sda12 /mnt/gentoo
    2. _#_ mkdir /mnt/gentoo/boot
    3. _#_ mount /dev/sda10 /mnt/gentoo/boot
    4. [optional] _#_ `mkdir /mnt/gentoo/home; mount /dev/sdaXY /mnt/gentoo/home`.
9. Setting the date and time using `date` command.
10. Go to the Gentoo mountpoint where the root file system is mounted (most likely /mnt/gentoo): `cd /mnt/gentoo`.
11. Downloading the stage tarball with Chrome application in LiveDVD. Go to [Installation media](https://www.gentoo.org/main/en/where.xml) and then to [amd64 multilib](http://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64/). You will find `stage3-amd64-20150319.tar.bz2`. Just download!
    1. Verify the tarball integrity and compare the output with the checksums provided by the .DIGESTS or .DIGESTS.asc file.
    2. _#_ sha512sum /home/gentoo/Download/stage3-amd64-20150319.tar.bz2
13. Now unpack the downloaded stage onto the system. Attention: the current working directory is `/mnt/gentoo`.
    1. _#_ tar xvjpf /mnt/cdrom/stage3-amd64-20150319.tar.bz2. Usually, the stage file is not in /mnt/gentoo directory. So you need to specify the path to stage. For instance, /home/gentoo/Download/stage3...
    2. Make sure that the same options `xvjpf` are used. The x stands for Extract, the v for Verbose to see what happens during the extraction process (optional), the j for Decompress with bzip2, the p for Preserve permissions and the f to denote that we want to extract a File, not standard input.
15. Configuring compile options. To keep the settings, Portage reads in the /etc/portage/make.conf file, a configuration file for Portage.
    1. _#_ emacs /mnt/gentoo/etc/portage/make.conf
    2. `CFLAGS` and `CXXFLAGS`. Check your CPU architecture to set `-march=` parameter. Refer to [Intel](https://wiki.gentoo.org/wiki/Safe_CFLAGS#Intel). Command `grep -m1 -A3 "vendor_id" /proc/cpuinfo` will show the current CPU architecure information. That link also teach you how to precisely detect `-march=` parameter by touching, compiling and comparing two _.gcc_ files.
        1. CFALGS="-march=corei7-avx -O2 -pipe"
        2. CXXFLAGS="${CFLAGS}"
    3. The `MAKEOPTS` variable defines how many parallel compilations should occur when installing a package. The recommended value is the number of logical processors in the CPU plus 1..
        1. Add a line `MAKEOPTS="-j5"`
        2. The boot screen will show you several penguins, that is the number of logical cores.
        3. Refer to [MAKEOPTS](https://wiki.gentoo.org/wiki/MAKEOPTS).
17. Selecting mirrors.
    1. _#_ mirrorselect -s3 -b10 -o -D >> /mnt/gentoo/etc/portage/make.conf, choose the 3 fastest mirrors for kernal source code downloading.
    2. _#_ mirrorselect -i -r -o >> /mnt/gentoo/etc/portage/make.conf, selects the `rsync server` to use when updating the portage tree. It is recommended to choose a _rotation link_, such as _rsync.us.gentoo.org_, rather than choosing a single mirror. This helps spread out the load and provides a fail-safe in case a specific mirror is offline.
        1. The rotation link setting is changed to `/etc/portage/repos.conf/gentoo.conf` for `portageq --version` >= 2.2.16. But don't worry since we can modify it when chrooting into the new Gentoo system. For now, remain this setting in `/etc/portage/make.conf` since the LiveCD environment uses the old version of portage.
18. _#_ cp -L /etc/resolv.conf /mnt/gentoo/etc/,  to ensure that networking still works even after entering the new sda12 (/mnt/gentoo) environment. `/etc/resolv.conf` contains the name servers for the network. DON'T forget the `-L` parameter when copying.
19. Mounting the necessary filesystems.
    1. _#_ mount -t proc proc /mnt/gentoo/proc
    2. _#_ mount --rbind /sys /mnt/gentoo/sys
    3. _#_ mount --make-rslave /mnt/gentoo/sys
    4. _#_ mount --rbind /dev /mnt/gentoo/dev
    5. _#_ mount --make-rslave /mnt/gentoo/dev
    1. _#_ **chroot /mnt/gentoo /bin/bash**, chrooting into the new environment.
    2. _#_ source /etc/profile
    3. _#_ export PS1="(chroot) $PS1", this create a new different command prompt for the new environment.
21. Installing a portage snapshot:
    1. _#_ emerge-webrsync, it might complain about a missing `/usr/portage/` location. This is to be expected and nothing to worry about - the tool will create the location.
22. Choosing the right profile.
    1. _#_ eselect profile list
    2. _#_ eselect profile set 3, choose the `desktop` profile, **Not** the `desktop/gnome` or `desktop/kde`. We will install `xfce` later on.
23. For `USE` flag, use command `emerge --info | grep ^USE` to check the default flags. The default flags change along with different profile selected. Xfce will be installed as desktop.
    4. Refer to [xfce HOWTO](https://wiki.gentoo.org/wiki/Xfce/HOWTO#The_basics) about the USE flags:

        ```
USE="-gnome -kde -minimal -qt4 dbus jpeg lock session startup-notification thunar udev X"
        ```
Append these flags into `make.conf` file. Actually, only `-qt4` and `thunar` need inserted for the others are already included in `emerge --info | grep ^USE`.
    5. Check if `cjk` and `nls` is enabled by `emerge --info | grep ^USE`. If not, update `make.conf` file.
24. Localization
    1. _#_ cat /usr/share/i18n/SUPPORTED | grep zh_CN >> /etc/locale.gen
    2. Uncomment `en_US.UTF-8 UTF-8` in /etc/locale.gen.
    3. _#_ locale-gen
    4. If reminds: run ". /etc/profile" to reload the variable in your shell". If you run it, you need to run `export PS1="(chroot) $PS1"` again.
    5. _#_ locale -a, to see what locales are generated.
    1. _#_ eselect locale list
    2. _#_ eselect locale set 3, set system-wdie locale to `en_US.utf8`.
    2. The following steps can also be done after entering the new Gentoo system.
    2. _#_ emerge -av arphicfonts wqy-bitmapfont corefonts ttf-bitstream-vera
    3. _#_ nano -w /etc/env.d/02locale. This setting will keep the original English system while displaying Chinese fonts. If you set LANG="zh_CN.xxx", then the system will be Chinese. Try `UTF-8` first otherwise many Chinese filenames not displaying correctly.

        ```
LANG="en_US.UTF-8"
LC_CTYPE="zh_CN.UTF-8"
LC_COLLATE="C"
        ```
    4. _#_ env-update && source /etc/profile
    5. _#_ export PS1="(chroot) $PS1", use this command to reminds you are in `chroot` environment.
    5. Use `xx_YY.UTF-8` or `xx_YY.utf8`. Don't use `xx_YY.UTF8`.
    5. Refer to [Gentoo本地化设置](http://www.jianshu.com/p/9411ab947f96); [Locale系统介绍](http://www.jianshu.com/p/86358b185e53).
25. Install the kernel source.
    1. If you would like to install the newest >=4.0.0 kernel, then refer to _Upgrade kernel to **unstable 4.0.0**_.
    1. _#_ emerge --ask sys-kernel/gentoo-sources
    2. _#_ ls -l /usr/src/linux
26. Configuring the Linux kernel - Manual configuration.
    1. If you have a backup of kernel `.config` file, then this and the next step can be skipped. Refer to _Upgrade kernel to **unstable 4.0.0**_ below.
    1. _#_ emerge --ask sys-apps/pciutils
    2. _#_ cd /usr/src/linux
26. Details on kernel configuration. Use the command `lspci -n` and paste it's output to [device driver check page](http://kmuto.jp/debian/hcl); that site gives you the kernel modules needed in general. Then go to kernel configuration (e.g. menuconfig) and press `/` to search the options like `e1000e`, find their locations and activate them.
    1. _#_ make menuconfig, if you have a backup of old gentoo kernel config file, then you can `cp /path/to/backup/config /usr/src/linux/.config`. Or you can refer to the LiveCD's kernel config file.
    1. The search with `/` in `menuconfig` output is as follows. The `Prompt` part is usually which should be activated.

        ```
Symbol:
Type:
Prompt:
  Location:
        ```
    1. `i915 e100e snd-hda-intel iTCO-wdt ahci i2c-i801 iwlwifi sdhci-pci`: these are the dirvers that needs activated. When search "snd-hda-intel", replace the _hypen_: **-** with _dash_: **_**.
    2. During kernel config, search the kernel options on page [Linux-3.10-x86_64 内核配置选项简介](http://www.jinbuguo.com/kernel/longterm-3_10-options.html) and the file `gentoo-livecd-default-kernel-config-reference` copied in previous step to help clear items.
    3. Graphics: i915 known as `Intel 8xx/9xx/G3x/G4x/HD Graphics, DRM_I915` uses the default 'Y'.
    4. Ethernet: e1000e  known as `Intel (R) PRO/1000 PCI-Express Gigabit Ethernet support` set to 'M'.
    4. Audio: snd\_hda_intel known as `Intel HD Audio, CONFIG_SND_HDA_INTEL` THE default  is 'Y', now **MUST** set it as 'M'.
        1. Refer to [no sound](https://forums.gentoo.org/viewtopic-t-791967-start-0.html) for how to decide the audio cdoec support.
        2. _#_ cat /proc/asound/card0/codec#* | grep Codec, the output is as follows. **ATENTION**: execute this command in LiveCD environment by opening a new terminal.

            ```
Codec Conexant CX20590
Codec Intel CougarPoint HDMI
            ```
        3. So select the two correspoinding codecs as modules: `Build HDMI/DisplayPort HD-audio codec support, SND_HDA_CODEC_HDMI`, and `Conexant HD-audio support, SND_HDA_CODEC_CONEXANT`.
        4. `Enable generic HD-audio codec parser, SND_HDA_GENERIC` must be selected (default).
        5. You may notice: those options are all set to 'M'! Of course, you can also set them all to 'Y'. Never set some to 'M' while set others to 'Y', othewise you would get no sound at all.
        4. Set `Default time-out for HD-audio power-save mode, CONFIG_SND_HDA_POWER_SAVE_DEFAULT` to 10.
        5. Set `Pre-allocated buffer size for HD-audio driver` to 4096.
    5. watchdog:  `Intel TCO Timer/Watchdog, ITCO_WDT` set  to 'M'.
    6. SATA: ahci known as `AHCI SATA support, CONFIG_SATA_AHCI` for SATA disks selected 'Y' default. Disable `ATA SFF support (for legacy IDE and PATA), CONFIG_ATA_SFF` set to 'N' for SATA disk device.
    7. `Intel 82801 (ICH/PCH), I2C_I801` uses default 'Y'.
    8. Wireless: `Intel Wireless WiFi Next Gen AGN - Wireless-N/Advanced-N/Ultimate-N (iwlwifi), CONFIG_IWLWIFI` set to 'M'. By the way, `wpa_supplicant` needs `nl80211` wifi driver. Actually relates to `cfg80211 - wireless configuration API, CONFIG_CFG80211` which is set to 'Y' default already.
    2. MMC: sdhci\_pci known as `SDHCI support on PCI bus, CONFIG\_MMC\_SDHCI_PCI`, but you cannot positioninig the item since its parent `Secure Digital Host Controller Interface Support` is turned off by default. So turn this on first. By the way, set `Ricoh MMC Controller Disabler, CONFIG\_MMC\_RICOH_MMC` as 'Y'.
    3. Enable EFI stub support and EFI variables in the Linux kernel if UEFI is used to boot the system: `EFI stub support, CONFIG_EFI_STUB` and `EFI mixed-mode support, CONFIG_EFI_MIXED` under `Processor type and features`.
    5. Refer to [Xorg configruation](https://wiki.gentoo.org/wiki/Xorg/Configuration#Installing_Xorg) to enable Xorg kernel support. However, according to this reference, nothing needs updated.
    6. Remove several `AMD` items under `Processor type and features` by searching 'AMD'. They are: CONFIG_AGP_AMD64, CONFIG_X86_MCE_AMD, CONFIG_MICROCODE_AMD, AMD_NUMA, and CONFIG_AMD_IOMMU.
    7. `NTFS` support: `CONFIG_NTFS_FS=m` and `CONFIG_FUSE_FS=m` to on demand. Refer to [NTFS wiki](https://wiki.gentoo.org/wiki/NTFS). You need `emerge --ask sys-fs/ntfs3g` to install `ntfs3g` package. Since `ntfs-3g` already support NTFS write, **don't** enable `CONFIG_NTFS_RW is not set`.
    9. Turn on `CONFIG_PACKET` (default 'Y')  to support wireless tool `wpa_supplicant` which will be installed later on.
    9. Turn off `NET_VENDOR_NVIDIA` to 'N' since no `NVIDIA` card in x220 laptop.
    10. Set `CONFIG_FAT_DEFAULT_CODEPAGE` to `936`, `CONFIG_FAT_DEFAULT_IOCHARSET` to `gb2312` for displaying NTFS partition Chinese file names correctly for `FAT` partition.
        1. Don't use `gb18030`. It seems that kernel does not recognize `gb18030`.
    10. Set `NLS_CODEPAGE_936` and `NLS_CODEPAGE_950` to 'M'.
    10. You may found that when there is some problems related to kernel options, you can see them to 'M' instead of 'Y' which might be a potential solution.
    10. This link [wlan0-no wireless extensions (Centrino Advanced-N)](https://forums.gentoo.org/viewtopic-t-883211.html) offer ideas on how to find out the driver information.
    11. Reference links: [device driver check page](http://kmuto.jp/debian/hcl); [How do you get hardware info and select drivers to be kept in a kernel compiled from source](http://unix.stackexchange.com/a/97813); and [Working with Kernel Seeds](http://kernel-seeds.org/working.html).
27. Compiling and installing.
    1. _#_ make
    2. _#_ make modules_install
    2. _#_ make install, this will copy the kernel image into /boot/ together with the System.map file and the kernel configuration file.
        1. Actually you can use a copy command instead.
    3. _deprecated #_ mkdir -p /boot/efi/boot
    4. _deprecated #_ cp /boot/vmlinuz-3.18.9-gentoo /boot/efi/boot/bootx64.efi
    5. _#_ emerge -av genkernel
    6. _#_ genkernel --install initramfs, The resulting file can be found by simply listing the files starting with initramfs: _#_ ls /boot/initramfs*.
27. Kernel modules loading. Refer to handbook.
27. Some drivers require additional firmware to be installed on the system before they work. This is often the case for network interfaces, especially wireless network interfaces.
    1. _#_ emerge --ask sys-kernel/linux-firmware
28. Creating the fstab file. The default `/etc/fstab` file provided by Gentoo is not a valid fstab file but instead more of a template. Use backup fstab file is possible.

	```
/dev/sda10   /boot        ext2    defaults,noatime     1 2
/dev/sda12   /	           ext4    noatime              0 1
/dev/sda7    none         swap	   sw                   0 0
	```
This needs modified in the steps later on.
29. Set hostname.
    1. _#_ nano -w /etc/conf.d/hostname
    2. set hostname="zhtux"
30. Configuring the network.
	1. **DO NOT follow the handbook guide for network during installation**. We don't need `net-misc/netifrc` at all. `net-misc/netifrc` needs support of `dhcp`, while `net-misc/dhcpcd` can handle network configuration alone.
	2. _#_ emerge --ask net-misc/dhcpcd
	3. _#_ rc-update add dhcpcd default
	4. From now, the Ethernet part is OK. Nothing special needs configured. `dhcpcd` will manage Ethernet connection when startup. But for the Wireless part, we need to install another tool `net-wireless/wpa_supplicant`.
	5. _#_ emerge --ask net-wireless/wpa_supplicant
	6. wpa\_configuration: Wifi parameters should be put in `/etc/wpa\_supplicant/wpa_supplicant.conf` file:

		```
# This command is to show the default configuration:
# bzcat /usr/share/doc/wpa_supplicant-2.2-r1/wpa_supplicant.conf.bz2 | less
# or http://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf
# Except eap and phase2 arguments, the rest are default values. 'phase1' must be 0 NOT 1.
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
    7. Remove the recommended options from wiki `GROUP=wheel` and `update_config=1` for security reason. After configuration below it is a good idea change the permissions to ensure that WiFi passwords can not be viewed in plaintext by anyone using the computer:
        1. _#_ chmod 600 /etc/wpa_supplicant/wpa_supplicant.conf
        2. Replace the `identity` and `password` entries with your own Wifi information.
    7. When `wpa_configuration` is configured as above, `dhcpcd` will automatically connect to the `sMobileNet` through `wpa_supplicant`. No need to create so called `/etc/conf.d/net` file as the handbook.
    8. If you have installed `net-misc/netifrc` and created `/etc/ini.d/net.*` and `/etc/conf.d/net` files, refer to [Migration from Gentoo net.* scripts](
	https://wiki.gentoo.org/wiki/Network_management_using_DHCPCD#Migration_from_Gentoo_net..2A_scripts).
    9. In case the network interface card should be configured with a static IP address, entries can also be manually added to `/etc/dhcpcd.conf`.
    10. wpa_supplicant 2.4 might cause authentication problem for PEAP Wifi. Refer to [Downgrade Package && wpa_supplicant && local overlay](http://fangxiang.tk/2015/05/11/gentoo-downgrade-package/)
    10. If need Gui tool, use `networkmanager` instead of `wicd` since the later one don't support `nl80211` driver. Also `networkmanager` depends on `wpa_supplicant` and `dhcpcd or dhcpclient`. It is more like a wrapper of `wpa_supplicant`.
	10. Reference: [Network management using DHCPCD](https://wiki.gentoo.org/wiki/Network_management_using_DHCPCD); [wpa_supplicant](https://wiki.gentoo.org/wiki/Wpa_supplicant); [Handbook:AMD64/Networking/Wireless](https://wiki.gentoo.org/wiki/Handbook:AMD64/Networking/Wireless); [configuration example](http://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf); [wpa_supplicant.conf for sMobileNet in HKUST](http://blog.ust.hk/yang/2012/09/21/wpa_supplicant-conf-for-smobilenet-in-hkust/); [wpa_supplicant.conf](http://www.freebsd.org/cgi/man.cgi?wpa_supplicant.conf).
31. Set root password: _#_ passwd
33. System logger.
    1. _#_ emerge --ask app-admin/syslog-ng
    2. _#_ rc-update add syslog-ng default
    3. _#_ emerge --ask app-admin/logrotate
34. Cron daemon. A cron daemon executes scheduled commands. It is very handy if some command needs to be executed regularly (for instance daily, weekly or monthly).
    1. _#_ emerge --ask sys-process/cronie
    2. _#_ rc-update add cronie default
35. File indexing: emerge --ask sys-apps/mlocate
36. NTFS: emerge --ask sys-fs/ntfs3g
36. [optional] Remote access: rc-update add sshd default
38. Configuring the bootloader. Refer to [GRUB2 Quick Start](https://wiki.gentoo.org/wiki/GRUB2_Quick_Start).
    1. Add `GRUB_PLATFORMS="efi-64"` to `/etc/portage/make.conf`. This step must occur before installing the grub package. Otherwise it would show `error: /usr/lib/grub/x86_64-efi/modinfo.sh doesn't exist`.
    2. _#_ emerge --ask sys-boot/grub:2, currently it is version 2.
    3. _#_ emerge -av sys-boot/os-prober
    3. Mount the EFI partition /dev/sda2 to /boot/efi directory. Because Gentoo, Ubuntu, Windows share the EFI partition, we should mount the shared EFI partion here. Not just create a private EFI environment in Gentoo's private boot partition. **This step is really important!**.
        1. _#_ mkdir /boot/efi
        2. _#_ mount /dev/sda2 /boot/efi
    4. To install GRUB2 to EFI system `grub2-install --target=x86_64-efi`.
39. Chainload into Ubuntu Grub2 `nano -w /etc/grub.d/40_custom`, add the code below.
    1. The traditional `chainloader +1` does work for UEFI boot.

		```
menuentry "Microsoft Windows 8.1 x86_64" {
    insmod part_gpt
    insmod fat
    insmod search_fs_uuid
    insmod chain
    search --fs-uuid --no-floppy --set=root $hints_string $fs_uuid
    chainloader /efi/Microsoft/Boot/bootmgfw.efi
}
		```
    2. The next is to replace the two parameters `$hints_string` and `$fs_uuid`. This is where `os-prober` comes into playing a role.
        1. _#_ grub2-probe --target=hints\_string /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi, this command will print the value `$hints_string`.
        2. _#_ grub2-probe --target=fs\_uuid /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi, this command will print the value `fs_uuid`.
        3. Now replace the parameters with them real values in above `menuentry`.
        4. Refer to [Windows installed in UEFI-GPT Mode menu entry](https://wiki.archlinux.org/index.php/GRUB#Windows_installed_in_UEFI-GPT_Mode_menu_entry) and [Can GRUB2 share the EFI system partition with Windows?](http://unix.stackexchange.com/q/49165).
40. Generate GRUB2 configuration: `grub2-mkconfig -o /boot/grub/grub.cfg`.
41. [optional] You can install `xorg` and `xfce` now without reboot below. Reboot is just for basic system test.
32. Edit `/etc/conf.d/hwclock` to set the clock options.
    1. set `clock=local`, this is important when dual boot with Windows.
23. Set the timezone.
    1. I think this step should occur after `hwclock` thing. Otherwise the system time is usually ahead of locale time by 8 hours, thus resulting in portage tree time stamp issues. If possible, I recommend to leave the above two steps immediately before system reboot.
    1. _#_ ls /usr/share/zoneinfo
    2. _#_ echo "Asia/Hong_Kong" > /etc/timezone
    3. _#_ emerge --config sys-libs/timezone-data
    4. Check with `date` command.
41. Exit the chrooted environment: `exit` and unmount all mounted partitions:

    ```
exit
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount /mnt/gentoo/boot/efi
umount /mnt/gentoo/boot
umount /mnt/gentoo
    ```
You may reminded that `/mnt/gentoo` is busy, then you need to exit the current terminal, and opening a new one terminal will work. Just let it go. Then type in that one magical command that initiates the final, true test: `reboot` with your root account.
    1. When rebooting, if the LiveDVD usb stick is still plugged onto the computer, the chainload to Ubuntu grub does not work. It show _error: disk hd0,gpt2 not found_. This is because the grub2 treats the USB stick as _hd0_ while the hard disk as _hd1_. You can unplug the USB, and CTRL+ALT+DEL. Another way is to edit the Ubuntu grub2 chainlaod menu from _hd0_ to _hd1_, then press F10 to boot.
    2. The very first thing after rebooting is to create a regular user account:

        ```
useradd -g users -G wheel,audio,video -m zachary
passwd zachary
        ```
    3. [OPTIONAL] Update the system. If no need, don't update your system, otherwise your whole world would be in a mess.
        1. _#_ emerge --sync, changed to `emains sync` for new portageq --version >=2.2.16.
        2. _#_ emerge -avtuDN --with-bdeps=y @world
        3. _#_ emerge -av --depclean
        4. _#_ [deprecated] revdep-rebuild -av, replaced by the next step.
        5. _#_ [optional] emerge @preserved-rebuild, if prompted.
        5. _#_ dispatch-conf, usually if needed, you just need to input `u`.
    4.  From now on, a basic new gentoo system is installed. 
42. Probably, the new system cannot connect to the Wifi network (lack in network manager). But if you configure WPA_supplicant and dhcpcd correctly, this is not a problem. If really no network, you can `chroot` again into the gentoo system when installing new package:
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
    7. _#_ env-update && source /etc/profile
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
    11. [optional] _#_ echo XSESSION="Xfce4" > /etc/env.d/90xsession, refer to the 11th item in previous step.
        1. Remember to run `env-update && source /etc/profile` to update environment.
    7. Installation finished. Now reboot and loggin with the regular account to configure xfce.
    8. _$_ emerge --search consolekit, you can see consolekit is installed. So follow the 2nd reference:
    9. _$_ echo "exec startxfce4 --with-ck-launch" > ~/.xinitrc
        1. At first try, the `logout`, `shutdown` buttons are greyed out. Since those buttons are related to `consolekit`, check the `consolekit` and `dbus` wiki.
        2. Make sure `consolekit` is added to default run level. `consolekit` depends on `dbus`, so `dbus` no need added to default run level. Run `rc-status` with normal user account, you will see dbus is under `Dynamic Runlevel`
        3. After a system update, the issue is solved automatically.
    10. _#_ rc-update add consolekit default
    12. You'd better logout and then login again to test xfce: _$_ startx.

        > **Attention**: Use _startx_ command to launch xfce desktop. No graphical loggin configured.
    14. If you have a messed desktop setting, you can executing the following commands to have a default setting:
        1.  _#_  rm -r ~/.cache/sessions
        2.  _#_ rm -r ~/.config/xfce*
        3.  _#_  rm -r ~/.config/Thunar
45. [deprecated, replaced by method in fstab] When you get into the xfce desktop, you may found many unnecessary disk icons on the desktop or thunar sidebar. It's annoying. Use `udev, udisks` utility.
    1. _#_ nano -w /etc/udev/rules.d/99-hide-disks.rules
    2. put the following code:

        ```
KERNEL=="sdaXY", ENV{UDISKS_IGNORE}="1"
        ```
`XY` is the disk partition number you would like to hide. As noted in the reference below, `UDISKS_PRESENTATION_HIDE` is deprecated and replaced by `UDISKS_IGNORE`.
    3. Similarly, since the root and home partition is formatted in previous step, you should go into Ubuntu system to hide these two partitions.
    4. Refer to [udev 99-hide-disks.rules is no longer working](http://superuser.com/questions/695791/udev-99-hide-disks-rules-is-no-longer-working).
43. Partitions:
    1. sda1 Windows recovery partition
    2. sda2 EFI partition
    3. sda3 windows reserved
    4. sda4 windows C
    5. sda5 Data
    6. sda6 Misc
    7. sda7 WLShare
    8. sda8 Gentoo boot
    9. sda9 Gentoo swap
    10. sda10 Gentoo boot
    11. sda11 Gentoo home
43. [OPTIONAL] Re-compiling current kernel when you need to modify some kernel configurations.
    1. _#_ mount /dev/sda10 /boot
    1. _#_ mount /dev/sda2 /boot/efi
    1. _#_ cd /usr/src/linux
    2. _#_  make menuconfig
        1. You don't need to copy and convert the old kernel config file as specified on [Kernel/Upgrade](https://wiki.gentoo.org/wiki/Kernel/Upgrade) since we just re-compile the current working kernel and share the kernel source. So we share the basic `.config` file in `/usr/src/linux/.config`.
        2. Just make some changes to the old config file.
    3. _#_ make
    4. _#_ make modules_install
    5. _#_ make install
        1. This is add `.old` to original kernel and copy the new kernel to `/boot`.
        2. You mannually finish this by renaming and copying kernel files.
    6. _#_ genkernel --install initramfs, re-install `initramfs`.
    7. _#_ grub2-mkconfig -o /boot/grub/grub.cfg
    8. _#_ reboot
    9. If you need to compile a different kernel version, refer to the step below _Upgrade kernel_.
45. ALSA - sound.
    1. _#_ emerge --search alsa, check whether `media-libs/alsa-lib` and `media-libs/alsa-utils` are installed or not. If not, `emerge -av media-libs/alsa-lib` to install `ALSA` support.
    2. _#_ rc-update add alsasound boot
    3. _#_ speaker-test -t wav -c 2, test the speaker.
45. Applications:
    1. Web browser: Firefox. It will take a while installing.

        > Adblock Edge, ChatZilla
    2. fcitx install. Refer to [Install (Gentoo)](https://fcitx-im.org/wiki/Install_(Gentoo)).
        1. _#_ echo "app-i18n/fcitx gtk3" >> /etc/portage/package.use/fcitx
        2. _#_ emerge -av fcitx
        2. According to fcitx wiki, the following lines should be added to `~/.xinitrc`:
		
            ```
eval `dbus-launch --sh-syntax --exit-with-session`
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=xim
export XMODIFIERS=@im=fcitx
            ```
        But this will conflicts with `--with-ck-launch`. The solution is to remove the first line related to `dbus`. Details refer to steps below.
        3. **IMPORTANT**: these four lines should be put **AHEAD** of `exec startxfce4 --with-ck-launch`. Commands after `exec` won't be executed! Refer to [xfce4安装fcitx不能激活！很简单的一个原因！](https://bbs.archlinuxcn.org/viewtopic.php?pid=13921).
        4. _#_ emerge -av fcitx-sunpinyin
        5. _#_ emerge -av fcitx-configtool
    3. _#_ emerge -av mplayer
    4. emacs:
        1. _#_ echo "app-editors/emacs xft toolkit-scrool-bars" > /etc/portage/package.use/emacs, `xft` is to support Chinese display.
        2. _#_ emerge -av emacs
        3. Chinese input with fcitx. First, you need to set `LC_CTYPE=zh_CN.utf8`. Second, change the fcitx input method trigger to `WIN+I` instead of `CTRL+SPACE`. Up to now, in terminal `enamcs -nw` can input Chinese character. But the Window Emacs will not. The solution is to emerge two fonts: `media-fonts/font-adobe-100dpi` and `media-fonts/font-adobe-75dpi`. You can search with Google the following Ebuild message for Emacs:

            ```
    if use X; then
        elog "You need to install some fonts for Emacs."
        elog "Installing media-fonts/font-adobe-{75,100}dpi on the X server's"
        elog "machine would satisfy basic Emacs requirements under X11."
        elog "See also http://www.gentoo.org/proj/en/lisp/emacs/xft.xml"
        elog "for how to enable anti-aliased fonts."
        elog
    fi
            ```
    5. _#_ emerge -av www-plugins/adobe-flash
        1. Pay attention to update `package.license` file when needed.
    6. WPS office.
        1. overlay support
            1. Refer to _New portage plug-in sync system_ below. If `portageq --version > 2.2.16`, install `layman >= 2.3.0`. The code below is for old layman version.
            1. _#_ echo "app-portage/layman git subversion" > /etc/portage/package.use/layman
            2. _#_ emerge -av layman
            2. _#_ echo "source /var/lib/layman/make.conf" >> /etc/portage/make.conf
            3. _#_ layman -f -a gentoo-zh
            4. _#_ layman -S
        2. WPS 32-bit `abi_x86_32` support. You should activate abi\_x86_32 use flag for packages on which WPS office depends.
            1. emerge -pv wps-office. It will reminds you what packages needs `abi_x86_32` support. Just answer 'Y' to the question and run command `dispatch-conf` to update configuration file. Of course you can update those files manually. You can also cange `pv` to `-av`.
            2. package.use/wps-office: check WPS overlay [wps-office-9.1.0.4945_alpha16_p3.ebuild](https://github.com/microcai/gentoo-zh/blob/master/app-office/wps-office/wps-office-9.1.0.4945_alpha16_p3.ebuild). You'd better use dispatch-conf to finish this work.
            3. package.license/wps-office: app-office/wps-office WPS-EULA
            4. package.accept\_keywords/wps-office: =app-office/wps-office-9.1.0.4945\_alpha16_p3 ~amd64

                >To use a specific software version from the testing branch but don't want portage to use the testing branch for subsequent versions, add in the version in the package.accept_keywords location. In this case use the = operator. It is also possible to enter a version range using the <=, <, > or >= operators. In any case, if version information is added, an operator must be used. Without version information, an operator cannot be used. Refer to [mixing branches](https://wiki.gentoo.org/wiki/Handbook:AMD64/Portage/Branches).
            5. emerge -av wps-office
	    6. **fonts support** refer to [Fontconfig](http://www.fangxiang.tk/2015/04/13/fontconfig/)
    7. _#_ emerge --ask xfce4-volumed xfce4-mixer
    8. _#_ emerge -av mupdf
        1. The other application may draw in a lot of GTK or QT dependencies consuming many disk space.
    9. _#_ emerge -av dev-vcs/git
        1. _$_ git config --global user.name "Jim Green"
        2. _$_ git config --global user.email "username@users.noreply.github.com"
        3. _$_ git config --global core.editor emacs
        4. _#_ emerge -av jekyll, it will install the rubygems, nodejs etc dependencies.
        5. _$_ git clone xxx
    10. _#_ emerge -av wgetpaste
    11. _#_ emerge -av net-misc/dropbox xfce-extra/thunar-dropbox
        1. Xfce4 and Dropbox does not get along well. There is no application menu for Dropbox.
        2. The system `LANG` or `LC_CTYPE` cannot be `zh_CN.GB18030`, otherswise dropbox does not launch with errors like _Gdk Critical...failed_.
            1. I think this can be overcome by setting the dropbox language to `english`.
        3. _$_ dropbox start, Gentoo and Windows share the Dropbox location in /media/Misc/Dropbox. When firstly run this command, you need to configure dropbox as default setting (Dropbox folder in /home/zachary/Dropbox). But then exit dropbox immediately. And remove /home/zachary/Dropbox.
            1. _$_ rmdir /home/zachary/Dropbox
            2. _$_ ln -s /media/Misc/Dropbox /home/zachary/Dropbox
        4. _$_ dropbox &, run dropbox in the backgroud.
        5. We can create a ~/bin/dropbox.sh script:

            >\#!/bin/bash
            >
            >/usr/bin/dropbox &

            So each time just run `~/bin/dropbox.sh`.
    12. _#_ emerge --ask app-portage/gentoolkit
    13. _#_ emerge --ask app-portage/eix
        1. _#_ emacs -nw /etc/eix-sync.conf: this is deprecated for the new poratge > 2.2.16.

            ```
*
            ```
        2. _$_ eix-sync
    14. Archive
        1. _#_ emerge -av file-roller
        2. _#_ emerge -av thunar-archive-plugin
            1. [steps below deprecated]
            1. Up to now, this is a bug in that `thunar-archive-plugin` cannot find a suitable archive manager. This is due a filename convention difference. The solution:
            2. _#_ cd /usr/libexec/thunar-archive-plugin/
            3. _#_ ln -s file-roller.tap org.gnome.FileRoller.tap
            4. After that, `thunar-archive-plugin` can find `file-roller` correctly. Refer to [thunar archive plugin cannot integrate with file-roller](https://forums.gentoo.org/viewtopic-t-1006838.html?sid=bce8eeef9eab8d916c59b01cef493bb4) and [doesn't work anymore with recent file-roller](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=746504).
        3. _#_ echo "app-arch/unzip natspec" > /etc/portage/package.use/unzip, this command is to let `unzip` support `GBK` Chinese filenames.
            1. 使用 “natspec” USE Flag重新编译unzip（zip文件中没有保存压缩时使用的编码，故需将unzip打上编码探测的补丁）
        4. _#_ emerge -av unzip, up to now, Chinese `zip` file can be extracted correctly by `file-roller`.
        5. _#_ emerge -av p7zip, three commands `7z`, `7za` and `7zr` can be used to extract files.
            1. If `p7zip` is installed, `file-roller` takes `7z` or `7za` to handle `zip` file which cannot handle Chinese filenames.
            2. Solution: rename `/usr/bin/7z` and `/usr/bin/7za` to something else. 若需要安装p7zip，则安装完成后，移除或重命名/usr/bin下除7zr外7z*文件（fileroller在7z或者7za存在的情况下会优先使用。而7z和7za解压zip文件会出现文件名乱码，暂不知如何解决。故删除7z和7za，仅保留7zr以支持7z格式的压缩和解压。)
        6. Use `7z` format instead of `zip` since `7z` support unicode compression especially for filenames.
    15. [deprecated, use Alt+F2 app finder instead] _#_ emerge -av xfce4-verve-plugin
        1. Add this plugin to panel.
        2. Under `Settings Editor` --> `xfce4-kerboard-shortcuts` --> `commands` --> `custom`, set a shortcut for this plugin: `Super + R`.
    16. [deprecated, does not work at all]_#_ emerge -av xfce4-wavelan-plugin
        1. __this plugin cannot be added to panel currently__, cannot be used.
46. Configuration consistently.
    1. Mount partition. Up to now, everything is fine except internal partitions like /dev/sda8,9 cannot be mounted in Thunar. When clicking the partition label, an error message `Failed to mount XXX. Not authorized to perform operation`. If you search around google, you might find many suggestions on changing configuration files of `polkit`. Relevant links [thunar 无权限挂载本地磁盘](http://blog.chinaunix.net/uid-25906175-id-3030600.html) and [Can't mount drive in Thunar anymore](http://unix.stackexchange.com/q/53498). None of this suggestions work. Detailed description of the problem is here [startx Failed to mount XXX, Not authorized to perform operat](https://forums.gentoo.org/viewtopic-t-1014734.html).
        1. **dbus should NOT launch before consolekit; dbus is already added into default runlevel**. This is the key to solve problem. In 4.10, start Xfce with `startxfce4 --with-ck-launch`. This will start xfce4-session with ck-launch-session. In 4.10, **Xfce4-sesion will take care of the dbus-session launch**.
        2. currently the contents of `~/.xinitrc`:

            ```
eval `dbus-launch --sh-syntax --exit-with-session`
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=xim
export XMODIFIERS=@im=fcitx
exec startxfce4 --with-ck-launch

            ```
        3. You can find `dbus-launch` is at the beginning of the file while `--with-ck-launch` is at the end along with command `exec`. So just remove the first line on dbus part:

            ```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=xim
export XMODIFIERS=@im=fcitx
exec startxfce4 --with-ck-launch
            ```
        4. Refer to [ConsoleKit](http://docs.xfce.org/xfce/xfce4-session/advanced); [Why is pcmanfm such a headache when it comes to mounting filesystems?](http://unix.stackexchange.com/q/30059); [ dwm and .xinitrc - thunar-daemon not mounting usb](http://crunchbang.org/forums/viewtopic.php?id=30373).
    2. fstab including NTFS partition [NTFS-3G](https://wiki.archlinux.org/index.php/NTFS-3G):

        ```
/dev/sda8 /boot ext2 noauto,noatime 1 2
/dev/sda2 /boot/efi vfat noauto,noatime 1 0
/dev/sda10 / ext4 noatime,errors=remount-ro 0 1
/dev/sda9 none swap sw 0 0
/dev/sda11 /home ext4 defaults 0 0
#/dev/cdrom /mnt/cdrom auto noauto,ro 0 0
#/dev/fd0 /mnt/floppy auto noauto 0 0
/dev/sda1 /mnt/Recovery ntfs-3g noauto,ro 0 0
/dev/sda4 /mnt/Win81 ntfs-3g noauto,ro 0 0
/dev/sda5 /media/Data ntfs-3g noauto,nls=utf8,locale=zh_CN.utf8,uid=zachary,gid=users,dmask=022,fmask=133 0 0
/dev/sda6 /media/Misc ntfs-3g noauto,nls=utf8,locale=zh_CN.utf8,uid=zachary,gid=users,dmask=022,fmask=133 0 0
/dev/sda7 /media/WLShare ntfs-3g noauto,nls=utf8,locale=zh_CN.utf8,uid=zachary,gid=users,dmask=022,fmask=133 0 0
        ```
        1. We should create the corresponding directory under `/media/` NOT under `/mnt/`. The reason can be found here [What is the difference between mounting in fstab and by mounting in file manager](http://unix.stackexchange.com/questions/169571/what-is-the-difference-between-mounting-in-fstab-and-by-mounting-in-file-manager).
        2. Pay attention to `nls` support which help displaying Chinese filenames correctly.
        3. But when you create a new Chinese filename in Thunar and copy it to NTFS partition, errors same as above step appear. If you change the mount option in `/etc/fstab` to `en_US.utf8`, you can handle Chinese filenames between Thunar and ntfs partition smoothly which will eventually conflicts with the above step. So you can; creating new Chinese filenames in virtual terminal.
        4. The first line /dev/sda4 is the Windows partition, this will hide it from Thunar sidebar.
        5. I think the most important thing is: the `locale` in `fstab` should be the same as the one in system `LC_CTYPE`. Also as an English system supporting Chinese, `zh_CN.UTF-8` (or `zh_CN.utf8`) is better than `zh_CN.GB2312` or `zh_CN.GB18030`. The later ones are for pure Chinese systems. If set to `zh_CN.GB18030` or `zh_CN.GB2312`, the Chinese folder names cannot be displayed in Thunar's address bar.
        6. Refer to [Mounting a Local Microsoft Windows Partition on Linux Systems](http://docs.oracle.com/cd/E19253-01/819-0918/localization-13).
        7. By default, /boot not auto-mounted for security reason. Similarly, the EFI System Partition is not auto-mounted to `/boot/efi` as well at startup.
    3. Don't use temporary USE flags in command line when emerge a package. Use `package.use` directory instead.
    4. `package.use`,`package.license` etc might be files or directories. I prefer directory ones and create specific files with finenames exactly the same as package name under directory

        > `> = < ~` operators are used for per-package configuration in these files/directories. Use `=` at best if allowed.
    5. New portage plug-in sync system. This new sync system requires `emerge -av \>=sys-apps/portage-2.2.16` and `emerge -av \>=app-portage/layman-2.3.0`.
        1. After a world update, my `portage` has updated to version `2.2.18`. Commands related to `emerge` reminds:

            ```
!!! SYNC setting found in make.conf.
    This setting is Deprecated and no longer used.  Please ensure your 'sync-type' and 'sync-uri' are set correctly in /etc/portage/repos.conf/gentoo.conf
            ```
        2. Refer to [Portage/Sync](https://wiki.gentoo.org/wiki/Project:Portage/Sync)
        3. _#_ mkdir /etc/portage/repos.conf
        4. _#_ cp /usr/share/portage/config/repos.conf /etc/portage/repos.conf/gentoo.conf
            1. This default setting is enough for the official portage sync. The argument `sync-uri` can be changed to someone near your local region. For instalce, it can be replaced by the one in your original `/etc/portage/make.conf`: `SYNC="rsync://rsync.cn.gentoo.org/gentoo-portage"`.
        5. Install new >=layman-2.3.0 supporting the new portage plug-in sync system.
            1. Add `sync-plugin-portage` USE flag to `/etc/package.use/layman` file.
            2. _#_ emerge -avt \>=app-portage/layman-2.3.0, the new layman package will create `/etc/portage/repos.conf/layman.conf` automatically.
            3. _#_ rm /var/lib/layman/make.conf, delete is the old-style layman config file.
        6. Edit `etc/portage/make.conf` and commnet out the lines `source /var/lib/layman/make.conf` and `SYNC="rsync://rsync.cn.gentoo.org/gentoo-portage"`.
        7. Primary control of all sync operations has been moved from `emerge` to `emaint`. `emerge --sync` now just calls the `emaint sync` module with the `--auto` option. The `--auto` option performs a sync on only those repositories with the auto-sync setting not set to `no` or `false`. If it is absent, then it will default to `yes` and "emerge --sync" will sync the repository. This means the original `emerge --sync` is just a wrapper of `emaint sync` with default argument `--auto` or `-a`.
        8. Note: `eix-sync` can update both overlays and portage while the new sync system will add overlays to `/etc/portage/repos.conf/layman` as well. So when `eix-sync` is called, the new procedure is likely: `layman -S; emerge --sync`. But the new `emerge --sync` will also update overlays in `/etc/portage/repos.conf/layman.conf`.

            ```
NOTE: As a result of the default auto-sync = True/Yes setting, commands 
 like "eix-sync", "esync -l", "emerge --sync && layman -S" will cause 
 many repositories to be synced multiple times in a row. Please edit 
 your configs or scripts to adjust for the new operation.
            ```
            9. To erase the duplicate updates incurred by `eix-sync` in new sync system, just remove `/etc/eix-sync.conf` or comment out overlays in that file.
        10. Choose one of the follwing command for daily operation:
            1. _#_ emaint sync -a
            2. _#_ emerge --sync
            3. _#_ eix-sync
        11. [sys-apps/portage-2.2.16 发布，支持多种同步方式](http://www.gentoo-cn.info/article/new-portage-plug-in-sync-system/); [Gentoo的portage已支持直接更新第三方源（overlay）](http://phpcj.org/portage-emerge-overlay-on-gentoo/).
47. Upgrade kernel to **unstable 4.0.0**
    1. _#_ echo "~sys-kernel/gentoo-sources-4.0.0 ~amd64" > /etc/portage/package.accept_keywords/gentoo-sources, this step needs `eix` command support to find out which unstable package version is located in portage mirror.
    2. _#_ emerge --sync
    3. _#_ emerge -av gentoo-sources
    4. _#_ eselect kernel list
    5. _#_ eselect kernel set 2, this is update the `/usr/src/linx` symbol link pointing to the new 4.0.0 kernel.
    5. _#_ mount /boot
    5. _#_ mount /dev/sda2 /boot/efi
    6. _#_ cd /usr/src/linux
    7. _#_ cp ../linux-3.18.11-gentoo/.config ./linux/, copy the old kernel config to the new kernel sources directory
    8. _#_ make silentoldconfig, choose all the new settings to default ones. It only shows new and different kernel options incurred in new kernel version.
        1. _#_ You can also run `make olddefconfig` to convert the old kernel config to new one without too many questions to answer. Command `make oldconfig` is similar to `make silentoldconfig` excpet also asking you the same kernel options between two kernel versions.
    9. _#_ make
    10. _#_ make modules_install
    11. _#_ make install
    12. _#_ genkernel --install initramfs
    13. _#_ grub2-mkconfig -o /boot/grub/grub.cfg
    14. _#_ reboot
48. Remove old kernels
    1. _#_ emerge -av app-admin/eclean-kernel
    2. _#_ eclean-kernel -n 4 -p, the option `-p` is to pretend removal, just showing which kernels will be removed.
    3. _#_ eclean-kernel -n 4 --ask
        1. Option `--ask` is very important, it will ask for your confirmation for each kernel to be removed by an interactive way. You can just remove the kernels you don't need. You can also adjust the number from `4` to something else to pop out the desired kernel for removal.
        2. At last, I removed the kernel as  `3.18.11-gentoo.old`
    4. `eclean-kernel` will update the grub menu automatically.
    5. **special note**: the kernel is manually compiled by `make, make modules_install, make install, ...` following the basic procedures of official handbook. But `initramfs` is built by `genkernel` resulting in file names different from conventional format. You can `ls -l /boot` finding that `initramfs`'s file name format is different from that of `vmlinuz, System.map`. Tough the file name format won't cause problems since `grub2-mkconfig` is smart enough to generate the correct boot parameters. However, `eclean-kernel` will remind you removing `-genkernel-x86_64-4.0.0-gentoo: vmlinuz does not exist` which is an undesired action. Details refer to [Bug 464576](https://bugs.gentoo.org/464576)
