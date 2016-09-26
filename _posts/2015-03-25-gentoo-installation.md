---
layout: post
title: Gentoo Installation
---

> Gentoo installation along with Windows 8.1 and Ubuntu 14.04 with UEFI booting.

# Tips

1. Follow [Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64).
2. To *emerge* packages, you'd better read ArchWiki.

# LiveDVD USB stick

2. Download the LiveDVD like *livedvd-amd64-multilib-20140826.iso* instead of Minimal installation CD like *install-amd64-minimal-20150319.iso*.
   1. Minimal CD cannot generates UEFI bootable USB stick.
   2. LiveDVD gives desktop environment to ease network connection and WWW surfing.
2. Create UEFI USB stick. Similar tools are:
   1. Rufus;
   1. UNetbootin;
   2. Universal USB Installer;
   3. *diskpart* terminal command;
   4. Just copy the ISO contents into a FAT32 format USB stick;
   5. Use *dd* command; 
   6. Ubuntu disk creater.

3. Boot into the KDE desktop environment with USB stick made before. By default, there is no password required for account *gentoo*.
   1. The very first thing is to connect to WIFI or Ethernet through networkmanager.
   2. Use shortcut F12 to Open/Retract Yakuake/Guake terminal in KDE destop.
   3. Refer to [Gentoo Ten LiveDVD Frequently Asked Questions](https://www.gentoo.org/proj/en/pr/releases/10.0/faq.xml).
4. `sudo su -` to *root*.

   You can use `passwd USERNAME` to change the password of *gentoo*. As *root*, you can change any account passworld by `passwd username`. All the password thing within the LiveDVD environment is volatile.

   The command prompt is *livecd ~ #* different from the handbook one - *root #*.

# Disk preparation
5. `parted -a optimal /dev/sda` or `fdisk /dev/sda`. Erase */dev/sda10* (NTFS partition) to install Gentoo on.

   **NOTE**: `parted` takes effect immediately without final confirmation like *fdisk*. Read [Kali Linux Live USB Persistence](/2015/07/23/kali-usb-persistence/) first.

   ```bash
   1. # parted -a optimal /dev/sda
   2. (parted) help
   2. (parted) p
   3. (parted) unit MiB
   3. (parted) rm 10
   3. (parted) p
   4. (parted) mkpart primary 309921MiB 310049MiB, create a boot partition sda10 for Gentoo
   5. (parted) p
   6. (parted) name 10 'Gentoo boot partition'
   7. (parted) mkpart primary 310049MiB -1, create root partition sda12 for Gentoo
   8. (parted) p
   9. (parted) name 12 'Gentoo root partition'
   10. (parted) p
   11. (parted) q
   ```
   
   The partition *Type* is *Basic data partition* when checking with *fdisk /dev/sda*. You change that through Disk GUI application under Ubuntu.
6. Newly created */dev/sda10* will be the *boot* partition while */dev/sda12* the *root* partition.

   We don't create *swap* or *efi* partition. Just share them with Ubuntu and Windows. You can also create a separate home partition, say *sda13*.
7. Format the new partition. You are suggested to format boot partition as *ext2*.

   ```bash
   # mkfs.ext2 /dev/sda10
   # mkfs.ext4 /dev/sda12
   # mkfs.ext4 /dev/sda13
   ```

8. Ubuntu *swap* partition is */dev/sda7*. So we need to *activate* it:

   ```bass
   # mkswap /dev/sdaXY (opt)
   # swapon /dev/sda7
   ```

   If you prefer an independent *swap* partition, *mkswap* to format it firstly.
9. Mount partitions.

   ```bash
   # mount /dev/sda12 /mnt/gentoo
   # mkdir /mnt/gentoo/boot && mount /dev/sda10 /mnt/gentoo/boot
   # mkdir /mnt/gentoo/home && mount /dev/sda13 /mnt/gentoo/home
   ```

# Install *stage3*

9. Check date and time using `date` command. Set by `date MMDDhhmm` if it's incorrect.
11. Downloading and verify *stage3* tarball. Usually, it's downloaded beforehand along with LiveDVD ISO.
12. Now unpack the downloaded stage onto the system.

    ```bash
    # cd /mnt/gentoo
    # tar xvjpf /mnt/cdrom/stage3-amd64-20150319.tar.bz2 --xattrs
    ```
    
    1. `x` stands for Extract;
    2. `v` for Verbose to see what happens during the extraction process (optional);
    3. `j` to Decompress with *bzip2*;
    4. `p` for Preserve permissions;
    5. `f` denotes that we want to extract a File, not standard input;
    5. `--xattrs` includes the extended attributes stored in the archive as well.

# *make.conf*

2. `CFLAGS` == `CXXFLAGS`

   Check your CPU architecture to set `-march=` parameter. Refer to [Intel](https://wiki.gentoo.org/wiki/Safe_CFLAGS#Intel). `grep -m1 -A3 "vendor_id" /proc/cpuinfo` offers CPU architecure. That link also teach you how to precisely detect `-march=` parameter by touching, compiling and comparing two *.gcc* files.

   1. CFALGS="-march=sandybridge -O2 -pipe"
   2. CXXFLAGS="${CFLAGS}"

3. The `MAKEOPTS` variable defines how many parallel compilations should occur when emerging packages.

   The recommended value is the number of logical processors in the CPU plus 1. But this rule is outdated. I have 4GB memory and *swap/swapfile* enabled, *emerge* will make use of *swap* havily. *swap* naturally slow down application performance though support more parallel tasks. So turn down to 2 or 3 to reduce usage of *swap* file.

   1. Add a line `MAKEOPTS="-j3"`.
   2. The boot screen shows several penguins, that is the number of logical hardware cores. If kernel `X86_SYSFB` and `FB_SIMPLE` were turned off, you could not see boot penguins.
   3. This value does not apply to *kernel compiling*. We explicitly specify *make -j3*.
   4. Refer to [MAKEOPTS](https://wiki.gentoo.org/wiki/MAKEOPTS).

4. `CPU_FLAGS_X86`: The 'USE' flags corresponding to the CPU instruction sets and other features specific to the *x86/amd64* architecture are *being* moved into a separate USE flag group called *CPU\_FLAGS_X86*.
   1. Refer to [cpu\_flags_x86 instruction](https://www.gentoo.org/support/news-items/2015-01-28-cpu_flags_x86-introduction.html).
   2. Emerge *app-portage/cpuinfo2cpuflags* and run in LiveDVD. Edit 'make.conf' to set *CPU\_FLAGS_X86*.
   4. Up to now, some packages in *portage* and overlays are not yet migrating those flags from 'USE' to 'CPU\_FLAGS\_X86'. So:
   5. Remove the old CPU specific flags from 'USE'. Then add *${CPU\_FLAGS_X86}* to 'USE'.

# Chrooting

18. `cp -L /etc/resolv.conf /mnt/gentoo/etc/`,  to ensure that networking after chrooting.
19. Mounting the necessary filesystems.

    ```bash
    # mount -t proc proc /mnt/gentoo/proc
    # mount --rbind /sys /mnt/gentoo/sys
    # mount --make-rslave /mnt/gentoo/sys
    # mount --rbind /dev /mnt/gentoo/dev
    # mount --make-rslave /mnt/gentoo/dev
    # chroot /mnt/gentoo /bin/bash
    # source /etc/profile && export PS1="(chroot) $PS1"
    ```

# Mirrors and Sync

17. Selecting mirrors
    
    ```bash
    # mirrorselect -s3 -b10 -o -D >> /mnt/gentoo/etc/portage/make.conf (opt)
    ```

    Choose the 3 fastest mirrors for package sources downloading. This will takes around 10 minutes. So usually, we just manually choose 3 physically close servers.
18. [deprecated] <s>Portage sync</s>

    ```bash
    # mirrorselect -i -r -o >> /mnt/gentoo/etc/portage/make.conf
    ```

    Selects the *rsync* server to use when synchronizing portage tree. It is recommended to choose a *rotation link*, such as *rsync.us.gentoo.org*, rather than choosing a single mirror (i.e. *ftp*). This helps spread out the load and provides a fail-safe in case a specific mirror is offline.

17. The new plug-in [sync system](https://wiki.gentoo.org/wiki/Project:Portage/Sync)

    The new *plug-in sync system* currently supports *rsync git svn websync webrsync cvs laymansync* synchronizing types. And configurations reside now under */etc/portage/repos.conf/*. More details are discusses after system installation.

    After installing *stage3*:

    ```bash
    # mkdir /mnt/gentoo/etc/portage/repos.conf
    # cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
    ```

# Portage tree

21. Portage snapshot

    ```bash
    # emerge-webrsync
    ```

    It might complain about a missing */usr/portage/* location. This is to be expected and nothing to worry about - it will be created on demand.
22. Profile

    ```bash
    # eselect profile list
    # eselect profile set 3
    ```

    Choose the *desktop* profile instead of *desktop/gnome* or *desktop/kde*. We will install XFCE later on.
23. USE flags

    Try `emerge --info | grep ^USE` to check USEs set by profile and *make.conf*. Refer to [Xfce/Guide](https://wiki.gentoo.org/wiki/Xfce/Guide#The_basics).

    ```
    USE="${CPU_FLAGS_X86} -qt4"
    ```
       
    By default, *bindist* is enabled by *stage3*'s default *make.conf*. If we don't plan to distribute packages, remove it. *bindist* may cause conflicts, i.e. between *openssh* and *openssl*.

# Localization

22. Time
    1. UTC (Universal Time Coordinated) is absolute global time.
    2. Local Time (UTC + Timezone) is what we use daily.

       Setting operating Software Time equal correctly to Local Time is our goal.
    3. Hardware Time resides on BIOS, which can be set manually and updated by operating system. Both Windows and Linux sets Software Time based on Hardware Time.

       It's regarded as *permanent placeholder* to set Software Time.
    4. Windows treats Hardware Time as Local Time and sets Software Time identically to Hardware Time without any translation on boot. Timezone is to synchronize time with time server. Upon shutdown, Windows writes back Software Time to Hardware Time.
    3. Linux treats Hardware Time as UTC and adds Timezone (i.e. +8) to Hardware Time on boot. Similarly, Software Time is written back to Hardware Time.
    4. Suppose Windows Software Time and Hardware Time are correctly synched to Local Time, Linux (dual boot installation afterwards) Software Time will be Hardware Time + Timezone = Windows Software Time + Timezone = Local Time + 2xTimezone. So usually Linux Software Time is ahead of Localtion for Timezone.

       If we shutdown Linux and boot Windows, Windows Software Time will be ahead of Local Time too. Each time we switch back and forth, Software Time will increase linearly by Timezone step.
    5. We must tell Linux that Hardware Time is Local Time instead of UTC *before* Software Timezone is configured.

32. hwclock

    Set *clock=local* in */etc/conf.d/hwclock*. We can find other options concerning reading/updating Hardware Time.

    Afterwards, set timezone information.

23. Timezone

    This step should be after */etc/conf.d/hwclock* update. Otherwise the Linux Software Time is usually ahead of local time by 8 hours, thus resulting in portage tree time stamp issues.

    ```bash
    # ls /usr/share/zoneinfo
    # echo "Asia/Shanghai" > /etc/timezone
    # emerge --config sys-libs/timezone-data
    ```

    Check with `date` command.
24. locale

    ```bash
    # cat /usr/share/i18n/SUPPORTED \| grep zh_CN >> /etc/locale.gen
    # uncomment en_US.UTF-8 UTF-8
    # locale-gen
    # locale -a
    # eselect locale list
    ```

    In the 3rd step, if reminded to run "*. /etc/profile*" to reload the variable, just remember `export PS1="(chroot) $PS1"`.

    Use `xx_YY.UTF-8` (or `xx_YY.utf8`. But this one might not work for some packages) instead of `xx_YY.UTF8`. How to achieve this? Use the *free form* of Gentoo *eselect*.

    ```
    # eselect locale set en_US.UTF-8
    ```
       
25. Chinese display

    */etc/env.d/02locale* offers fine-grained *locale* settings. We keep the original English system while displaying Chinese characters. If you set LANG="zh_CN.xxx", then the system will be Chinese. Try `UTF-8` first otherwise many Chinese filenames not displaying correctly.

    ```
    LANG="en_US.UTF-8"
    LC_CTYPE="zh_CN.UTF-8"
    LC_COLLATE="C"
    ```
       
    Remember to

    ```bash
    # env-update && source /etc/profile && export PS1="(chroot) $PS1"
    ```

    in the end.

    1. In order to display Chinese characters, we need to install Chinese fonts, refer to [fontconfig](/2015/04/13/fontconfig/).
    2. Refer to [Gentoo本地化设置](http://www.jianshu.com/p/9411ab947f96); [Locale系统介绍](http://www.jianshu.com/p/86358b185e53) and [Jin Buguo](http://jinbuguo.com).

# Kernel building

25. Install the kernel source

    ```bash
    # emerge -avt sys-kernel/gentoo-sources
    # ls -l /usr/src
    # cd /usr/src/linu && patch [--dry-run] -p1 < /path/to/cjktty.patch (opt)
    ```

    This is the first time we installIf kernel sources. Therefore, */etc/src/linux* symlink is created automatically.

    If possible, apply kernel patches like *cjktty.patch*.
26. Tips
    1. You'd best have a *.config* backup to start with.
    2. If possible, try *sys-apps/pciutils, sys-apps/usbutils, and sys-apps/hwinfo* in LiveDVD. Don't emerge those packages in *chroot*, it's not necessary.
    3. During kernel config, search the kernel options on page [Linux-3.10-x86_64 内核配置选项简介](http://www.jinbuguo.com/kernel/longterm-3_10-options.html) and refer to LiveDVD's configuration.
    5. Try `lspci -n` and paste it's output to [device driver check page](http://kmuto.jp/debian/hcl); that site gives kernel options that must be enabled.
    6. We can refer to the LiveDVD's kernel config directly.
    7. Search with `/`. When search "snd-hda-intel", replace the hypen with dash.

        Usually there are several search outputs numerated (1, 2, 3 ...). Press the number to enter the kernel option.
    8. *exit* or two successive ESCs to get back.
    4. When confronted with issues related to kernel options, we can choose 'M' instead of 'Y' which might be a solution.

    Suppose we have an old *.config* backup:

    ```bash
    # cd /usr/src/linux
    # cp /path/to/.config-backup .
    # make help
    # make silentoldconfig
    ```

    6. If the *.config* backup belongs to the same kernel version, we omit *make silentoldconfig*.
    7. *oldconfig* asks for both new and old options.
    8. *silentoldconfig* only asks for NEW options while preserving old ones. Press ENTER to choose default setting.
    9. *olddefconfig* is similar but sets NEW options to default without confirmation.

    We might tune kernel options by graphical interface *make menuconfig* below.
26. Menuconfig

    ```bash
    # cd /usr/src/linux
    # make menuconfig
    ```

    1. `i915 e100e snd-hda-intel iTCO-wdt ahci i2c-i801 iwlwifi sdhci-pci`.

        Basic dirvers that needs activated.

    2. `IKCONFIG` should be Y instead of M. `IKCONFIG_PROC` can be enabled as you wish. This allows you to inspect the configuration of the kernel while it is running, without having to worry whether you've changed or cleaned the source directory after it was built. 
    2. `NR_CPUS` set to 4 since laptop has 2 cores and 4 threading. The smaller this value is, the smaller the kernel image is and the less memory is consumed by kernel.
    3. `MICROCDE` and `MICROCDE_INTEL` are Y by default. Refer to *microcode-ctl* below.
    3. Graphics. `i915` = *Intel 8xx/9xx/G3x/G4x/HD Graphics, DRM_I915* uses the default Y.
    3. `FB FRAMEBUFFER_CONSOLE X86_SYSFB FB_SIMPLE` all set to Y.

       Or `FB FRAMEBUFFER_CONSOLE FB_EFI` to Y.
    4. Ethernet. `e1000e` = *Intel (R) PRO/1000 PCI-Express Gigabit Ethernet support* set to M.
    4. Audio. `snd_hda_intel` = *Intel HD Audio, CONFIG_SND_HDA_INTEL*. The default value is Y, now MUST be M. Choose the audio codec:

       Refer to [no sound](https://forums.gentoo.org/viewtopic-t-791967-start-0.html) for how to decide the audio cdoec support.

       Execute `cat /proc/asound/card0/codec#* \| grep -i codec` in LiveDVD:
          
       ```
       Codec Conexant CX20590
       Codec Intel CougarPoint HDMI
       ```
          
       3. Then select those two codecs as M. *Build HDMI/DisplayPort HD-audio codec support* =` SND_HDA_CODEC_HDMI`. *Conexant HD-audio support* = `SND_HDA_CODEC_CONEXANT`.
       4. *Enable generic HD-audio codec parser* = `SND_HDA_GENERIC` must be selected (default).
       4. Set *Default time-out for HD-audio power-save mode* = `CONFIG_SND_HDA_POWER_SAVE_DEFAULT` to 10.
       5. Set *Pre-allocated buffer size for HD-audio driver* to 4096.
       5. You notice these options are ALL set to M! You can also set them all to Y. But never set some to M while set others to Y, othewise you would get no sound at all.

    5. Webcamera
       1. lsusb

          ```
          Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
          Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
          Bus 001 Device 006: ID 04f2:b217 Chicony Electronics Co., Ltd Lenovo Integrated Camera (0.3MP)
          Bus 001 Device 005: ID 0a5c:217f Broadcom Corp. BCM2045B (BDC-2.1)
          Bus 001 Device 004: ID 147e:2016 Upek Biometric Touchchip/Touchstrip Fingerprint Sensor
          Bus 001 Device 003: ID 046d:c52b Logitech, Inc. Unifying Receiver
          Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
          Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
          ```

          The 3rd item is the integrated webcamera which is a multimedia USB device.
       2. Enable `MEDIA_SUPPORT` = *Multimedia support*, `MEDIA_CAMERA_SUPPORT` = *Cameras/video grabbers support*, `MEDIA_USB_SUPPORT` = *Media USB Adapters*, and `USB_VIDEO_CLASS` = *USB Video Class (UVC)*. The 2nd and 4th items are the key to make webcamera to work.

    5. Processor type and features
       1. `lscpu | grep -i 'CPU family'`. If the ouput is 6, select the 3rd item as *Core 2/newer Xeon* = `CONFIG_MCORE2`, otherwise the output would be 15, please select the 5th item.
       2. Turn off `NUMA` = *Numa Memory Allocation and Scheduler Support*. Refer to [What Is NUMA?](https://forums.gentoo.org/viewtopic-t-911696-view-next.html?sid=7c550d5e3f0942fbfcbc9ad80df53c57).
       3. Remove several `AMD` items under *Processor type and features* by searching AMD. They are `AGP_AMD64 X86_MCE_AMD MICROCODE_AMD AMD_NUMA AMD_IOMMU`.
       4. Enable *EFI runtime service support* = `EFI` if UEFI is used to boot the system. Turn off *EFI stub support* = `EFI_STUB`, *EFI mixed-mode support* = `EFI_MIXED`, *Built-in kernel command line* = `CMDLINE_BOOL`, *Built-in kernel command string* = `CONFIG_CMDLINE` and *Initramfs source file(s)* = `INITRAMFS_SOURCE`. These options are closely related mainly for compiling kernel boot arguments and *initramfs* into kernel image. Then the system can boot and load kernel directly from EFI firmware instead of bootloader. Details refer to [efi stub kernel](https://wiki.gentoo.org/wiki/EFI_stub_kerne).

    5. Set `INPUT_PCSPKR` as M or Y for the standard PC Speaker to be used for bells and whistles. Don't enable `SND_PCSP` (read the kernel help message)!
    5. Set `X86_X32` = *x32 ABI for 64-bit mode* to Y. This option is useful for 32 bit binaries to take advantage of the system 64-bit registers.
    5. Set `IOMMU_SUPPORT`, `INTEL_IOMMU`, and `INTEL_IOMMU_DEFAULT_ON` set to N since laptop lacks VT-d (but has VT-x). VT-d is what Intel calls their IOMMU implementation. Refer to [i5 2410M](http://ark.intel.com/products/52224/Intel-Core-i5-2410M-Processor-3M-Cache-up-to-2_90-GHz). Without IOMMU enabled, HD3000 graphics will tear and corrupt.
    5. Watchdog. *Intel TCO Timer/Watchdog* = `ITCO_WDT` set  to M.
    6. SATA. `ahci` = *AHCI SATA support*, `SATA_AHCI` for SATA disks selected 'Y' default. Disable *ATA SFF support (for legacy IDE and PATA)* = `ATA_SFF` set to N.
    7. *Intel 82801 (ICH/PCH)* = `I2C_I801` uses default 'Y'.
    8. Wireless. *Intel Wireless WiFi Next Gen AGN - Wireless-N/Advanced-N/Ultimate-N (iwlwifi), CONFIG_IWLWIFI* set to M. By the way, `wpa_supplicant` needs `nl80211` wifi driver. Actually relates to *cfg80211 - wireless configuration API* = `CFG80211` which is set to Y default already.
    9. (opt) Bluetooth. `BT` = *Bluetooth subsystem support* to M. Enter and find `BT_RFCOMM` = *RFCOMM protocol support*. I think this should be 'M', otherwise package like `obexfs` or `obexftp` did not work. Choose USB driver `BT_HCIBTUSB` = *HCI USB driver* as M. The sub-option `BT_HCIBTUSB_BCM` turned on automatically. `BT_HIDP` = *HIDP protocol support* is for human interface device like bluetooth mouse, bluetooth headset, bluetooth keyboard etc. Since I dont' use them, so leave it as N. My bluetooth *Logitech mouse* works perfectly even when turnning off all the related bluetooth kernel options. That is due to the extra `LOGITECH` related kernel drivers. Read more from *bluetooth - bluez obexfs*.
    2. MMC. `sdhci_pci` = *SDHCI support on PCI bus* = `MMC_SDHCI_PCI`, but you cannot positioninig the item since its parent *Secure Digital Host Controller Interface Support* is turned off by default. So turn this on first. By the way, set *Ricoh MMC Controller Disabler* = MMC_RICOH_MMC` as Y.
    5. Refer to [Xorg configruation](https://wiki.gentoo.org/wiki/Xorg/Configuration#Installing_Xorg) to enable Xorg kernel support. However, according to this reference, nothing needs updated.
    7. NTFS. `NTFS_FS` and `FUSE_FS` to M. Refer to [NTFS wiki](https://wiki.gentoo.org/wiki/NTFS). You need to install `ntfs-3g` package later on. Since *ntfs-3g* embeds NTFS write, set`NTFS_RW` to N.
    9. Turn on `PACKET` (default Y)  for wireless tool `wpa_supplicant` which will be installed later on.
    9. Turn off `NET_VENDOR_NVIDIA` since no `NVIDIA` card in laptop.
    10. [Deprecated, use the default 437 and iso8859-1].
    
        Set `FAT_DEFAULT_CODEPAGE` to 936, `FAT_DEFAULT_IOCHARSET` to *gb2312* for NTFS partition Chinese file names correctly for displayed on FAT partition. Don't use *gb18030*. It seems that kernel does not recognize *gb18030*.

        Set `NLS_CODEPAGE_936` and `NLS_CODEPAGE_950` to M.

        On windows, FAT is now mainly used as USB bootable stick, EFI partition, etc. For file storage, NTFS is a better choice.
    10. Dm-crypt. *Device mapper support* = `BLK_DEV_DM` is set Y by default. *Crypt target support* = `DM_CRYPT` must be M or Y. *XTS support* = `CRYPTO_XTS` and *AES cipher algorithms (x86_64)* = `CRYPTO_AES_X86_64` optionally set to M (recommended). Refer to [Dm-crypt](https://wiki.gentoo.org/wiki/Dm-crypt). Refer to *Cryptsetup* step below. `CRYPTO_SERPENT` and `CRYPTO_SHA512` must be Y instead of M if relevant LUKS algorithms are used. Refer to [gentoo over lvm luks](2015/08/15/gentoo-over-lvm-luks/).
    10. Iptables. `NETFILTER_ADVANCED` & `XT_MATCH_OWNER` (*-m owner*). `IP_NF_TARGET_REDIRECT` (-j REDIRECT) which will enable `NF_NAT_REDIRECT` (in return, `NETFILTER_XT_TARGET_REDIRECT` be enabled as well). We only need `NETFILTER_XT_TARGET_REDIRECT` for iptables REDIRECT, so to simplify kernel, just enable it alone. If necessary, turn on `IP6_NF_NAT` (`NF_NAT_IPV6` as dependency) to enable ip6tables *nat* table. Turn on `NETFILTER_XT_MATCH_MULTIPORT` for *-m multiport*
    10. System log: `SECURITY_DMESG_RESTRICT` to Y. Refer to [Restrict unprivileged access to kernel syslog](https://lwn.net/Articles/414813/).

        More refer to *syslog-ng* below.
    10. Filesystem. Set `UDF_FS` as M to mount ISO file.
    10. (opt) Thinkpad-related. *ThinkPad ACPI Laptop Extras* = `THINKPAD_ACPI` set to M.
    10. [e-sources / cjktty patch specific options].`FONTS` and `FONT_8x16` set to Y. And *console 16x16 CJK font ( cover BMP )* = `FONT_16x16_CJK` which is *cjktty* patch. These options are for Chinese characters display in TTY (Ctrl + Alt + Fn).
    11. Reference links: [Linux-3.10-x86_64 内核配置选项简介](http://www.jinbuguo.com/kernel/longterm-3_10-options.html); [Linux Kernel in a Nutshell](http://www.kroah.com/lkn/); [kernel-seeds](http://kernel-seeds.org/); [device driver check page](http://kmuto.jp/debian/hcl); [How do you get hardware info and select drivers to be kept in a kernel compiled from source](http://unix.stackexchange.com/a/97813); and [Working with Kernel Seeds](http://kernel-seeds.org/working.html).

27. Compiling and installing

    ```bash
    # cd /usr/src/linux
    # mount /boot; /boot/efi (opt)
    # make clean/mrproper/distclean; echo 3 > /proc/sys/vm/drop_caches (opt)
    # make -j3 && make modules_install && make install
    # emerge -avt genkernel; genkernel --install initramfs
    # emerge -avt @module-rebuild
    # reboot
    ```

    1. Get into kernel sources tree.
    2. For kernel upgrading/re-compiling.
    3. Clean previous building leftovers.
       1. *clean* removes most generated files but keep the *.config* and enough build support to build external modules (such that we overlook *make modules_prepare*).
       2. *mrproper* removes all generated files + *.config* + various backup files.
       3. *distclean* = *mrproper* + remove editor backup and patch files.
       4. *drop_caches* is set to 3.

       They are chosen if we re-compile kernel sources in case of compiling error. What's worse, a successful re-compiling generates unworkable kernel binaries.

       Please be noted that *mrproper* and *disclean* will remove backup files, especially the *.config*.

    4. Compiling.
       1. Compiles the kernel. Keep `-j3` since MAKEOPTS in *make.conf* does not apply to kernel compiling.
       2. Install kernel modules into */lib/modules/\`uname -r\`*. If we are re-compiling the kernel, old modules will be overriden. Ether backup the old modules or install modules to a new location (by `INSTALL_MOD_PATH`) when we want the old kernel be bootable (i.e. we remove a module from kernel).
       3. Install kernel binaries *System.map, config, initramfs, vmlinuz* into */boot* and rename old identical version kernel binaries on demand. We can copy and even rename those binaries manually as long as filenames are kept consistent.
    5. For LVM/LUKS containers, emerge *genkernel* with *cryptsetup* USE.

       Extra arguments `--lvm --luks --gpg --busybox` should be supplied upon generates *initramfs*. Refer to [Gentoo rootfs over LVM encrypted in LUKS container](/2015/08/15/gentoo-over-lvm-luks/) and [lvm luks lvm](/2015/09/10/lvm-luks-lvm/).

       Like modules, *genkernel* does not backup *initramfs* automatically when re-compiling the kernel. We are responsible for bakcuping manually, especially when the *genkernel* arguments are different.
    6. Although we are running system with old kernel, *@module-rebuild* knows how to re-install external kernel module as long as */usr/src/linux* symlink pointing the new kernel (seee *eselect kernel list*).
    7. *make modules_prepare* is somewhat complicated.

       External kernel modules (i.e. self-written codes; VirtualBox binary kernel modules) are built against kernel tree (*/usr/src/linux/*). That's because external modules building need support from kernel sources support (i.e. kernel sources' head file).
    
       So the kernel tree should be prepared to support external kernel modules building, which can be achieve by several ways:

       1. If the kernel tree is brand new (i.e. just unpacked) and we will build external kernel modules before kernel building, we should prepare by *make modules_prepare* under */usr/src/linux/*.
       2. If the kernel have already been built (i.e. *make -j3*), it's already prepared. Need to prepare.

27. Linux firmware

    Some drivers require additional firmware to be installed on the system before they work. This is often the case for network interfaces, especially wireless network interfaces.

    ```bash
    # emerge -avt --fechonly sys-kernel/linux-firmware
    ```

    Firstly just fetch the package sources. If network failed connect after booting into new kernel, emerge it then.

#  System configuration

28. *fstab*

    Creating the */etc/fstab* file. Use backup at best. The default */etc/fstab* file provided by Gentoo is not a valid fstab file but instead more of a template.

    ```
    /dev/sda10   /boot		ext2    defaults,noatime	1 2
    /dev/sda12   /   		ext4    noatime	       0 1
    /dev/sda7    none	  	swap	sw	       0 0
    proc         /proc          proc    nosuid,nodev,noexec,hidepid=2,gid=wheel 0 0
    none         /tmp           tmpfs   nodev,nosuid,noexec     0 0
    ```

    Double-check the */etc/fstab* file, save and quit to continue. Fault *fstab* results in boot failure.
29. *hostname* and *domain*
    1. nano -w */etc/conf.d/hostname*

       ```
       hostname="myhost"
       ```

    2. On login, usually get message like *This is myhost.unkown_domain ...* which is not a error, but annoying. There are two ways to get rid of it.

       The first way is to set a fake domain for localhost, edit */etc/hosts*. Add fake *hostname.domain* to *localhost*.
       
       ```
       # <ip address>	<fully qualified hostname>	<aliases>
       127.0.0.1       myhost.jiantu.boxes      myhost   localhost
       ::1             myhost.jiantu.boxes      myhost   localhost
       ```
       
       Now try `ping myhost.jiantu.boxes/myhost/localhost` to test.

       Another method is edit */etc/issue*, remove `.\O`.
30. Enable OpenRC log

   ```bash
   # nano -w /etc/rc.conf
   ```

31. Set root password

    ```bash
    # passwd
    ```

# [Networking](/2016/03/01/networking/)

30. *dhcpcd*

    We don't follow [Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64)'s *net config*. *net-misc/netifrc* requires support from DHCP, while *net-misc/dhcpcd* can handle network configuration alone.

    ```bash
    # emerge -avt net-misc/dhcpcd
    # rc-update add dhcpcd default
    # emerge -avt net-wireless/wpa_supplicant
    ```

    2. In case of the network interface card should be configured with a static IP address, entries can also be manually added to */etc/dhcpcd.conf*.
    3. If need GUI tool, use *networkmanager* instead of *wicd* since the later one don't support *nl80211* driver. Also *Networkmanager* depends on *wpa_supplicant* and *dhcpcd* or *dhcpclient*.
    4. Reference: [Network management using DHCPCD](https://wiki.gentoo.org/wiki/Network_management_using_DHCPCD); [wpa_supplicant](https://wiki.gentoo.org/wiki/Wpa_supplicant); [Handbook:AMD64/Networking/Wireless](https://wiki.gentoo.org/wiki/Handbook:AMD64/Networking/Wireless); [configuration example](http://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf); [wpa_supplicant.conf for sMobileNet in HKUST](http://blog.ust.hk/yang/2012/09/21/wpa_supplicant-conf-for-smobilenet-in-hkust/); [wpa_supplicant.conf](http://www.freebsd.org/cgi/man.cgi?wpa_supplicant.conf).

31. *wpa_supplicant*

    1. *dhcpcd* manages Ethernet connection on boot while revoking *wpa_supplicant* hook script to initiate wireless networking.
    2. Remove the recommended options  `GROUP=wheel` and `update_config=1` from Wiki for security reason.

    */etc/wpa_supplicant/wpa_supplicant.conf*.

    ```
    ctrl_interface=DIR=/var/run/wpa_supplicant
    ctrl_interface_group=0
    fast_reauth=1
    eapol_version=1
    ap_scan=1

    network={
	    disabled=0
	    ssid="public-wifi"
	    key_mgmt=NONE
	    priority=-999
    }

    network={
	    disabled=1
	    ssid="Network1"
	    proto=WPA RSN
	    key_mgmt=WPA-EAP
	    pairwise=CCMP TKIP
	    group=CCMP TKIP 
	    eap=PEAP
	    identity="XXXXXXXX"
	    password="YYYYYYYY"
	    ca_cert="/etc/ssl/certs/Thawte_Premium_Server_CA.pem"
	    phase1="peaplabel=0"
	    phase2="auth=MSCHAPV2"
	    priority=10
    }

    network={
	    disabled=1
	    ssid="Network2"
	    proto=WPA RSN
	    key_mgmt=WPA-EAP
	    pairwise=CCMP TKIP
	    group=CCMP TKIP 
	    eap=PEAP
	    identity="XXXXXXXX"
	    password="YYYYYYYY"
    # Don't need certificate
    #       ca_cert="/etc/ssl/certs/Thawte_Premium_Server_CA.pem"
    # If use the latest wpa_supplicant, try 'tls_disable_tls_v1_2=1'
    #       phase1="tls_disable_tlsv1_2=1"
	    phase1="peaplabel=0"
	    phase2="auth=MSCHAPV2"
	    priority=20
    }

    network={
	    disabled=0
	    ssid="Network3"
	    #psk="12345678"
	    psk=aaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbccccccccccccccccccc
	    priority=30
    }
    ```

    For home wireless connection, usuaully you just need to specify *ssid* and *psk* arguments. *psk* can be a normal wifi password (call *passphrase*) or a 256-character string. When use normal password, need quotes, while 256-character string does not. How to generate the long string?

    ```bash
    # wpa_passphrase wifi-ssid wifi-password >> /etc/wpa_supplicant/wpa_supplicant.conf
    ```

    We'd better change the permissions to ensure that WiFi passwords can not be viewed in *psk* or *passphrase* by normal user account.

    ```bash
    # chmod 600 /etc/wpa_supplicant/wpa_supplicant.conf`.
    ```

    *dhcpcd* revokes *wpa_supplicant* through */lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant* hook. From *dhcpcd-6.10.0* onward, *10-wpa_supplicant* hook is no longer there by default. We should copy */usr/share/dhcpcd/hooks/10-wpa_supplicant* to */lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant*.

    `lspci -k` shows wireless driver in use is *iwlwifi*. However if specify `-D iwlwifi` on command line, *wpa_supplicant* will fail:

    >Unsupported driver 'iwlwifi'

    The actual driver in use was *nl80211*.

    1. If you have installed `net-misc/netifrc` (by default from *stage3*) and created `/etc/ini.d/net.*` and `/etc/conf.d/net` files, refer to [Migration from Gentoo net.* scripts](https://wiki.gentoo.org/wiki/Network_management_using_DHCPCD#Migration_from_Gentoo_net..2A_scripts).
    3. *wpa_supplicant-2.4* might cause authentication problem for PEAP Wifi. Refer to [Downgrade Package && wpa_supplicant && local overlay](/2015/05/11/gentoo-downgrade-package/)

# System tools

33. System logger.

    ```bash
    # emerge -avt app-admin/syslog-ng app-admin/logrotate
    # rc-update add syslog-ng default
    ```

    By default, there is a line

    ```
    log { source(src); destination(console_all); };
    ```

    in */etc/syslog-ng/syslog-ng.conf*, which outputs system logs to */dev/tty12*. That's to say, any users can read system logs by switching to the 12th virtual terminal. For security reason, comment out that line if there are multiple users.

    Details on logrotate for cron jobs refer to [Cronie and Anacron](/2015/07/19/cronie/).
34. Cron daemon

    A cron daemon executes scheduled commands. It is very handy if some command needs to be executed regularly (for instance daily, weekly or monthly).

    ```bash
    # echo "sys-process/cronie anacron" > /etc/portage/package.use/cronie (opt)
    # emerge -avt sys-process/cronie
    # rc-update add cronie default
    ```

    1. *anaron* USE is not must if no rigid shechuled tasks.
    2. *cronie* must be in a runlevel to shedule *logrotate*.
    1. Details on running scheduled tasks based on input from the command *crontab*, refer to [Cronie and Anacron](/2015/07/19/cronie/).

35. More tools

    ```bash
    # emerge -avt sys-apps/mlocate, file indexing
    # emerge -avt sys-fs/ntfs3g, mounting NTFS filesystem
    # rc-update add sshd default, incoming SSH connection (opt)
    ```

# Bootloader - Grub2

38. Configuring the bootloader. Refer to [GRUB2 Quick Start](https://wiki.gentoo.org/wiki/GRUB2_Quick_Start).
    1. Add `GRUB_PLATFORMS="efi-64"` to `/etc/portage/make.conf`. This step must occur before installing the grub package. Otherwise it would show `error: /usr/lib/grub/x86_64-efi/modinfo.sh doesn't exist`.
    2. \# emerge -avt sys-boot/grub:2, currently it is version 2.
    3. \# emerge -av sys-boot/os-prober
    3. Mount the EFI partition /dev/sda2 to /boot/efi directory. Because Gentoo, Ubuntu, Windows share the EFI partition, we should mount the shared EFI partion here. Not just create a private EFI environment in Gentoo's private boot partition. **This step is really important!**.
        1. \# mkdir /boot/efi
        2. \# mount /dev/sda2 /boot/efi
    4. To install GRUB2 to EFI system `grub2-install --target=x86_64-efi`.
39. [deprecated, as long as the `/boot` and `/boot/efi` partitions are mounted, grub2-mkconfig will automatically detect the windows operating system in `/etc/grub.d/30_os_prober` through `sys-boot/os-prober`] Chainload Windows system `nano -w /etc/grub.d/40_custom`, add the code below.
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
       1. \# grub2-probe --target=hints\_string /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi, this command will print the value `$hints_string`.
       2. \# grub2-probe --target=fs\_uuid /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi, this command will print the value `fs_uuid`.
       3. Now replace the parameters with them real values in above `menuentry`.
       4. Refer to [Windows installed in UEFI-GPT Mode menu entry](https://wiki.archlinux.org/index.php/GRUB\#Windows_installed_in_UEFI-GPT_Mode_menu_entry) and [Can GRUB2 share the EFI system partition with Windows?](http://unix.stackexchange.com/q/49165).
40. Generate GRUB2 configuration: `grub2-mkconfig -o /boot/grub/grub.cfg`.
41. [optional] You can install `xorg` and `xfce` now without reboot below. Reboot is just for basic system test.
41. Exit the chrooted environment: `exit` and unmount all mounted partitions:

    ```
    # exit
    # umount -lv /mnt/gentoo/home
    # umount -lv /mnt/gentoo/boot{/efi,}
    # umount -lv /mnt/gentoo/dev{/shm,/pts,}
    # umount -lv /mnt/gentoo{/proc,/sys,}
    ```

    You may be reminded that `/mnt/gentoo` is busy, then you need to exit the current terminal, and opening a new one terminal will work. Just let it go. Then type in that one magical command that initiates the final, true test: `reboot` with your root account.

    1. When rebooting, if the LiveDVD usb stick is still plugged onto the computer, the chainload to Ubuntu grub does not work. It show _error: disk hd0,gpt2 not found_. This is because the grub2 treats the USB stick as _hd0_ while the hard disk as _hd1_. You can unplug the USB, and CTRL+ALT+DEL. Another way is to edit the Ubuntu grub2 chainlaod menu from _hd0_ to _hd1_, then press F10 to boot.
    2. The very first thing after rebooting is to create a regular user account:

       ```
       useradd -g users -G wheel,audio,video -m zachary
       passwd zachary
       ```

    3. [OPTIONAL] Update the system. If no need, don't update your system, otherwise your whole world would be in a mess.
       1. \# eix-sync
          1. For new portageq --version >=2.2.16, use `emaint sync` instead of `emerge --sync`.
       2. \# emerge -avtuDN --with-bdeps=y @world
       6. [optional] # dispatch-conf, if prompted, you just need to input `u`.

          This command will help update files in `/etc/portage` when needed as well. I would like to separate per-package settings by filenames. Simply input `u` will merge several package settings together, which is undesirable. Hence, first check the updates by `diff` the `._cfg*` in corresponding directory. And then rename `._cfg*` to relevant package name.
       3. \# perl-cleaner --all

          If *perl* issues still occurs, replace *--all* with *--reallyall*.
       4. [optional] # revdep-rebuild -pv
          1. \# revdep-rebuild -v
          2. It is recommended to perform the 4th step. As a tool of `Gentoolkit`, `revdep-rebuild` is Gentoo's Reverse Dependency rebuilder. It will scan the installed ebuilds to find packages that have become broken as a result of an upgrade of a package they depend on. It can emerge those packages for users automatically but it can also happen that a given package does not work with the currently installed dependencies, in which case you should upgrade the broken package to a more recent version. revdep-rebuild will pass flags to emerge which lets you use the --pretend flag to see what is going to be emerged again before going any further. 
       5. [optional] # emerge @preserved-rebuild, if prompted.
          1. $ emerge --info \| grep FEATURES
          2. In Gentoo profile, there might be `preserve-libs` enabled for `FEATURES`, which is usually the real case. This setting will cause `Portage` to preserve libraries when `soname`s change during upgrade or downgrade, only as necessary to satisfy shared library dependencies of installed consumers/packages. Preserved libraries are automatically removed when there are no remaining consumers, which occurs when consumer packages are rebuilt or uninstalled. Ideally, rebuilds are triggered automatically during updates, in order to satisfy slot-operator dependencies.
          3. However, before emerge exits after installing updates, if there are remaining preserved libraries because slot-operator dependencies have not been used to trigger automatic rebuilds, then emerge will display a message like the following:

             ```
             !!! existing preserved libs:
             >>> package: sys-libs/libfoo-1
              * - /lib/libfoo.so.1
              *      used by /usr/bin/bar (app-foo/bar-1)
             Use emerge @preserved-rebuild to rebuild packages using these libraries
             ```

          4. This is when `emerge @preserved-rebuild` come into effects. Refer to [preserve-libs](https://wiki.gentoo.org/wiki/Preserve-libs).
        3. \# emerge -av --depclean
           1. Cleans the system by removing packages that are  not  associated with  explicitly merged packages. Depclean works by creating the full dependency tree from the @world set, then comparing it to installed packages. Packages installed, but not part of the dependency tree, will be uninstalled by depclean.
    4.  From now on, a basic new gentoo system is installed. 
42. Probably, the new system cannot connect to the Wifi network (lack in network manager). But if you configure WPA_supplicant and dhcpcd correctly, this is not a problem. If really no network, you can `chroot` again into the gentoo system when installing new package:
    1. Boot with LiveDVD
    2. \# swapon /dev/sda7
    3. \# mount /dev/sda12 /mnt/gentoo
    4. \# mount /dev/sda10 /mnt/gentoo/boot
    5. \# cp -L /etc/resolv.conf /mnt/gentoo/etc
    6. \# mount -t proc proc /mnt/gentoo/proc
    7. \# mount --rbind /sys /mnt/gentoo/sys
    8. \# mount --make-rslave /mnt/gentoo/sys 
    9. \# mount --rbind /dev /mnt/gentoo/dev 
    10. \# mount --make-rslave /mnt/gentoo/dev
    11. \# chroot /mnt/gentoo /bin/bash
    12. \# source /etc/profile
    13. \# export PS1="(chroot) $PS1"
    14. Now you can install `xorg` and `xfce` for gentoo system with the help of LiveDVD KDE wifi connection.
43. Xorg installaion.
    1. Refer to [Xorg/Configuration](https://wiki.gentoo.org/wiki/Xorg/Configuration).
    2. For the kernel support part, already done in previous step.
    3. Add the following lines into `/etc/portage/make.conf`:

       ```
       ## (For mouse, keyboard, and Synaptics touchpad support)
       INPUT_DEVICES="evdev synaptics"
       ## (For intel cards)
       VIDEO_CARDS="intel i965"
       or
       VIDEO_CARDS="intel"
       ```

       Do NOT set to `VIDEO_CARDS="intel i915". The Wiki writes *Gen 2 and Gen 3* using *intel i915*. However, the *Gen* output of *lspci* does not reflect the real VGA generation.

       *intel* alone will enable both *i915* and *i965* Intel video drivers for *media-libs/mesa*. Specifying *intel i965* will only build *i965* Intel drivers.

       *ATTENTION*: this *i965* and *i915* split in this context is only at application (*media-libs/mesa*) level. The kernel side always enabled *i915* instead.
    4. \# emerge -avt --verbose --pretend x11-base/xorg-drivers, check the dependency.
    5. \# echo "x11-base/xorg-server udev" >> /etc/portage/package.use/xorg-server. Actually this step is unnecessary since `udev` is enabled by default when selecting the system profile in previous step.
    6. \# emerge -avt x11-base/xorg-server
    7. \# env-update && source /etc/profile
    9. \# export PS1="(chroot) $PS1"
    10. The official wiki suggests installing `x11-wm/twm` and `x11-terms/xterm` to test `xorg` installation. However, we are currently chrooting, startx is already running supporting the LiveDVD KDE environment. Hence, we cannot test by issuing command `startx` in chroot environment. It's only possible when reboot into the genuine gentoo system. So skip this step.
    11. For this command: echo XSESSION="Xfce4" > /etc/env.d/90xsession, we have not installed `xfce` yet. So leave it for the next step.
44. Xfce installation & Configuration.
    1. Refer to [Xfce](https://wiki.gentoo.org/wiki/Xfce) for installation and [Xfce/HOWTO](https://wiki.gentoo.org/wiki/Xfce/HOWTO) for configuration.
    2. \# eselect profile list, you will find `…/desktop` is the default profile (not `…/gnome` or `…/kde`).
    3. [optional] # echo 'app-text/poppler -qt4' >> /etc/portage/package.use/poppler, since `-qt4` is already set globally in previous step when installing the basic gentoo system.
    4. [optional] # echo 'dev-util/cmake -qt4' >> /etc/portage/package.use/cmake
    3. \# echo 'gnome-base/gvfs -http' >> /etc/portage/package.use/gvfs
    4. \# echo 'XFCE_PLUGINS="brightness clock trash"' >> /etc/portage/make.conf
    5. **Attention** # emerge -avt xfce4-meta xfce4-notifyd; emerge --deselect y xfce4-notifyd, the 1st reference mixed this command order with step 4.
    6. \# emerge -avt x11-terms/xfce4-terminal

        By default, xfce4-terminal disables beep by default.

        ```bash
        grep -i bell ~/.config/xfce4/terminal/terminalrc
        MiscBell=TRUE
        printf "\7"; sleep 1; printf "\a" 
        ```

        To turn on beep in terminal is useful especially for IRC Weechat (when others messaging you).
    11. <s>[optional] # echo XSESSION="Xfce4" > /etc/env.d/90xsession, refer to the 11th item in previous step.
        1. Remember to run `env-update && source /etc/profile` to update environment.</s>
        2. This is not a good scheme to set *XSESSION* for all users on the system.
    7. Installation finished. Now reboot and loggin with the regular account to configure xfce.
    9. _$_ echo "exec startxfce4 --with-ck-launch" > ~/.xinitrc
        1. At first try, the `logout`, `shutdown` buttons are greyed out. Since those buttons are related to `consolekit`, check the `consolekit` and `dbus` wiki.
        2. Make sure `consolekit` is added to default run level. `consolekit` depends on `dbus`, so `dbus` no need added to default run level. Run `rc-status` with normal user account, you will see dbus is under `Dynamic Runlevel`
        3. After a system update, the issue is solved automatically.
    8. \$ emerge --search consolekit, you can see consolekit is installed. So follow the 2nd reference:
    10. \# rc-update add consolekit default
    11. Disable automatic launch of *gpg-agent* or *ssh-agent*

        ```bash
        $ xfconf-query -c xfce4-session -p /startup/gpg-agent/enabled -n -t bool -s false
        $ xfconf-query -c xfce4-session -p /startup/ssh-agent/enabled -n -t bool -s false
        ```

        More refer to post *GnuPG 2*.
    12. You'd better logout and then login again to test xfce: _$_ startx.

        > **Attention**: Use _startx_ command to launch xfce desktop. No graphical loggin configured.
    13. If you really want to start X upon login without manually typing `startx`:

        ```bash
        # ~/.bashrc
        #
        # Starting X11 on console login automatically when login to the first terminal tty1.
        # This is useful when you want to use X and avoid type 'startx' on the termnial.
        # But if you login to terminal tty2 ~ tty7, this script won't start X for you.
        if [[ $(tty) == /dev/tty1 ]]; then
            exec startx
        fi
        ```

    14. If you have a messed desktop setting, you can executing the following commands to have a default setting:
        1. \#  rm -r ~/.cache/sessions
        2. \# rm -r ~/.config/xfce*
        3. \#  rm -r ~/.config/Thunar
45. [deprecated, replaced by method in fstab]

    When you get into the xfce desktop, you may found many unnecessary disk icons on the desktop or thunar sidebar. It's annoying. Use `udev, udisks` utility.
    
    1. \# nano -w /etc/udev/rules.d/99-hide-disks.rules
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
43. Re-compiling current kernel when modifying kernel configurations.
    1. \# mount /boot
    1. \# mount /boot/efi
    1. \# cd /usr/src/linux
    2. \# make menuconfig
       1. You don't need to copy and convert the old kernel config file as specified on [Kernel/Upgrade](https://wiki.gentoo.org/wiki/Kernel/Upgrade) since we just re-compile the current working kernel and share the kernel source. So we share the basic `.config` file in `/usr/src/linux/.config`.
       2. Just make some changes to the old config file.
    3. [optional] \# make clean
       1. Whenever the kernel sources or `.config` is changed, or when you are re-compiling a previously compiled kernel, run `make clean`. Otherwise the compiling process might fail.
    3. \# make modules_prepare
    1. \# echo 3 > /proc/sys/vm/drop_caches
    3. \# make -j3
    4. \# Backup /lib/modules/\`uname -r\`/ contents. Why? Since we are re-compiling the current kernel, *make modules_install* will override old modules. If the newly-compiled kernel and modules thereof are error-prone or even cannot boot, then you cannot boot into old kernel since its modules (some are crucial) are overriden.
    4. \# make modules_install
    5. \# make install
       1. This is add `.old` to backup current kernel and copy the new kernel to */boot*.
       2. You can mannually finish this by renaming and copying kernel files.
    5. \# cp /boot/initramfs-genkernel-x86_64-version-gentoo /boot/initramfs-genkernel-x86_64-version-gentoo.old
       1. *initramfs* won't be renamed by `make install`. 
    6. \# genkernel --install initramfs, re-install `initramfs`.
    7. \# grub2-mkconfig -o /boot/grub/grub.cfg
    8. \# emerge -av @module-rebuild
    8. \# reboot
    9. If you need to compile a different kernel version, refer to the step below _Upgrade kernel_.
45. ALSA - sound.
    1. \# emerge --search alsa, check whether `media-libs/alsa-lib` and `media-libs/alsa-utils` are installed or not. If not, `emerge -av media-libs/alsa-lib` to install `ALSA` support. By default, the `alsa` USE flag is enabled in profile, so these packages will be emerged by default.
    2. [optional] # rc-update add alsasound boot
    3. \# speaker-test -t wav -c 2, test the speaker.
45. Applications:
    1. Firefox < 43

       ```
       www-client/firefox gstreamer
       media-plugins/gst-plugins-libav -libav
       # emerge -av firefox
       # emerge -av1 gst-plugins-vaapi
       # emerge -av www-plugins/adobe-flash
       ```

       1. Enable `gstreamer` USE flag for Firefox to support more video codecs (like H264).
       2. Disable `-libav` USE flag for *gst-plugins-libav* package to uses *ffmpeg* instead of *libav* for video codecs.
       3. *gst-plugins-vaapi* supports *hardware acceleration* to reduce CPU overhead. However, Firefox failed to make use of *gst-plugins-vaapi*. Refer to [894372](https://bugzilla.mozilla.org/show_bug.cgi?id=894372), [56326](https://bugzilla.mozilla.org/show_bug.cgi?id=563206) and [1207429](https://bugzilla.mozilla.org/show_bug.cgi?id=1207429).
       3. From Firefox 43 on, it uses *ffmpeg* instead of *gstreamer*. To make use of *vaapi*, make sure *ffmpeg* and *hwaccel* USE is enabled. BTW, *ffmpeg* should enable *vaapi* USE too.
       3. Now HTML5 H264 support is OK. But for *Media Source Extensions*, wee need to turn on *media.fragmented-mp4.exposed*, *media.fragmented-mp4.ffmpeg.enabled*, *media.mediasource.enabled*, *media.mediasource.mp4.enabled* and *media.mediasource.webm.enabled* in *about:config*, while disabling *media.fragmented-mp4.use-blank-decoder*.
       4. Run `$ flash-player-properties` or from application menu to set flash player.
       4. Add *FoxyProxy Standard*, *uBlock Origin*, *NoScript* (and/or *RefControl*), *DownThemAll*, *user agent switcher* etc. plugins.
       5. Remove unecessary default whitelist of NoScript plugin.
       5. *privacy.trackingprotection.enabled* to TRUE.
       5. [Harden Firefox security](https://vikingvpn.com/cybersecurity-wiki/browser-security/guide-hardening-mozilla-firefox-for-privacy-and-security) and [disable useragent](http://www.howtogeek.com/113439/how-to-change-your-browsers-user-agent-without-installing-any-extensions/). Like *general.useragent.vendor/override*.
    2. Weechat for IRC.
    2. *xfce-extra/xfce4-screenshooter* for capture sreen image.
    2. fcitx install. Refer to [Install (Gentoo)](https://fcitx-im.org/wiki/Install_(Gentoo)).
       1. \# echo "app-i18n/fcitx gtk gtk3" >> /etc/portage/package.use/fcitx
       2. \# emerge -av fcitx
       2. According to fcitx wiki, the following lines should be added to `~/.xinitrc`:

          ```
          export GTK_IM_MODULE=fcitx
          export QT_IM_MODULE=xim
          export XMODIFIERS=@im=fcitx
          ```

          But this will conflicts with `--with-ck-launch`. The solution is to remove the first line related to `dbus`. Details refer to steps below.
       3. **IMPORTANT**: these three lines should be put **AHEAD** of `exec startxfce4 --with-ck-launch`. Commands after `exec` won't be executed! Refer to [xfce4安装fcitx不能激活！很简单的一个原因！](https://bbs.archlinuxcn.org/viewtopic.php?pid=13921).
       4. \# emerge -av fcitx-configtool fcitx-sunpinyin or fcitx-googlepinyin

          It's optional to install *fcitx-configtool* if you are OK to configure Fcitx on command line manually.
    3. mpv

       By default, stable *mpv* is really old (0.9.2). We can accpet the recent 0.17.0 or live 9999 instead.

       1. Add `vaapi` to */etc/portage/make.conf*.
       1. *mpv* depends on *ffmpeg* as dependency. But some features need to tune *ffmpeg* USE flags instead of *mpv*'s. For example, `libv4l` and `gnutls`.
       1. If *mpv* does not play videos, try to
          1. \# emerge -av1 mpv
          2. <s>Bump to a *~arch* of *xf86-video-intel* / *media-libs/mesa*</s>.
          3. Reboot.
       2. Previously, I use `mplayer` which is inactive of development. Now `mpv` is a good choice and has built-in simple GUI based on *lua* language. Under *xfce4*, you might need *reboot* to let *mpv* work.
       3. The default *lua* USE flag of *mpv* brings along a *youtube-dl* hook script. `mpv url` (like live broadcast) will automatically invoke *youtube-dl* to download video cache while palying. Before that, install *youtube-dl*. And make sure *ffmpeg* is installed with `network` USE flag. If the url is *https*, then add `gnutls` or `openssl` USE flag to *ffmpeg*. Ether `gnutls` or `openssl` is fine and you don't need both.
       4. Though *youtube-dl* support many sites, it does play well with Chinese ones. We can try `mpv $(youtube-dl --get-url http://v.youku.com/v_show/id_XMTU2Mjc4MTc4NA==.html)`.

          If install *youtube-dl* in Python3 virtual environment:

          ```bash
          $ ln -sf /absolute/path/to/py3venv/youtube-dl ~/.config/mpv/youtue-dl
          ```

          If *youtube-dl* sucks, could try *livestreamer*.
       5. There is another tool tuned for Chinese sites, that is *you-get* written with Python3. It's brilliant like this `you-get -p mpv url`. I install this package in Python3 virtual environment (*pyvenv*). Of course, *you-get* can be used to download video directly from sites like youtube-dl.
       6. If the url is blocked, we should first enable *proxychains* or *torify*.
       7. If want to make use of *hardware decoding*, both *mpv* and *ffmpeg* should enable *vaapi* USE flag.

          It's better to add `vaapi` USE flag to *make.conf*.
       8. Watching speed.

          When playing url videos, *mpv* might throtle/stuck a lot waiting for cache. This is due to ISP throtling; video provider streaming limit; network bandwidth etc.

          1. Set `--cache=500000`; press SPACE to pause for a while (say 10 minutes); then SPACE to play.
          2. Run *youtube-dl url* and *mpv partial-downloaded-file* separately (say in two terminals). If does play the partial file, try `-f best` argument which tell to download audio and video into a single file (without merging support). If downloading audio and video separately, there would be no audio or video when playing.
          3. Tunning playback speed.
       9. We can use Firefox addons *Watch with MPV* or *Open With* to watch video by invoking *mpv*. But *Watch with MPV* cannot handle GFW issue. Also *mpv*'s default *youtube-dl* may not handle Chinese sites. *Open With* addon is capable of customizing multiple personal applications:

          ```
          /usr/bin/proxychains /usr/bin/mpv  --
          or
          ~/workspace/virtualenv3/bin/you-get -p /usr/bin/mpv --
          ```

          The last two hypens means arguments afterwards are input files/urls. Since using the *Open With* addon, save more CPU and memory.
       10. More refer to *intel vga bug* below.
       11. Details on [*youtube-dl*](https://github.com/rg3/youtube-dl) and [*you-get*](https://github.com/soimort/you-get), refer to their Github pages.
    18. ffmpeg
        1. `ffmpeg` is emerged by some other packages, one of which might be `mplayer` or `mpv`.
        2. However, the default installation does not support `v4l` (mainly the webcamera). If want to support `v4l`, add USE flags `v4l` and/or `libv4l`.
        3. By default *mpv* use *software decoding*. To use *hardware decoding* of Intel GPU, enable *vaapi* for *ffmpeg*. When playing, add `--hwdec=auto` or `--hwdec=vaapi` argument. Of course, it can be added to configuration file (like *~/.config/mpv/mpv.conf*).
        4. Add `-libav` to *make.conf* in favor of system-wide *ffmpeg*.
        5. *ffmpeg* has native RTMP implementation. But if that sucks, enable `librtmp` USE to replace the native ones.

        Manually enabled USE flags are:

        >media-video/ffmpeg openssl vaapi

        Remove `vaapi` if it's already enabled in *make.conf* or *media-video/mpv*.

        ```
        # emerge -av1 ffmpeg
        ```

    4. \# emerge -av guayadeque, make sure the `minimal` USE flag is enabled to install a very minimal build (disables, for example, plugins, fonts, most drivers, non-critical features). Then emerge plugins on demand.

       Since it is *minimal*, when playing songs, guayadeque reminds:

       >Your GStreamer installation is missing a plug-in.

       This is due to lack of essential *gstreamer* plugins, namely:

       >emerge -av1 gst-plugins-bad:0.10 gst-plugins-good:0.10 gst-plugins-ugly:0.10 gst-plugins-base:0.10 gst-plugins-mad:0.10

       Pay attention to `-1` *oneshot* option to explicitly draw in dependencies.

       `guayadeque` depends on `0.10` slot `gstreamer` plugins, instead of those `1.0` slot plugins. The official ebuild should be updated to include those five `0.10` slots even `minimal` USE flag is enabled.

       More useful plugins: `gst-plugins-flac:0.10` `flac` track; `gst-plugins-soup` for online `radio`; `gst-plugins-ffmpeg` for `ape` track.

       This link [guayadeque missing gstreamer plugin](https://forums.gentoo.org/viewtopic-p-7663344.html) recommends `emerge gst-plugins-meta:0.10`. However, it will draw in 21 `gst-plugins-*`. Actually, minimal `guayadeque` does not need so much plugins, but this command is convenient. A more convinient way is to remove `minimal` USE flag. These two convenient methods would emerge many unuseful pakcages.

       If you cannot find a plugin to play specific music format, have a look at `equery g gst-plugins-meta:0.10`.

       **Restart guayadeque** to make those plugins work.
    4. emacs:
       1. \# echo "app-editors/emacs xft libxml2 gnutls athena Xaw3d -gtk -gtk3 -motif" > /etc/portage/package.use/emacs, `xft` is to support Chinese display. `libxml2` is to support builtin `shr`, which in return support `eww` web browser and Gnus viewing HTML. `gnutls` supports Gnus imap connection. `athena Xaw3d -gtk -gtk3 -motif` is to solve *daemon mode* bug [reddit](https://www.reddit.com/r/emacs/comments/2ans0z/have_you_encountered_that_gtk_bug_in_daemon_mode/?ref=share&ref_source=link) and [wiki](https://wiki.gentoo.org/wiki/GNU_Emacs).
       2. \# emerge -av emacs
       3. \# emerge -av media-fonts/font-adobe-75dpi media-fonts/font-adobe-100dpi

          Chinese input with fcitx. First, you need to set `LC_CTYPE=zh_CN.utf8`. Second, change the fcitx input method trigger to `WIN+I` instead of `CTRL+SPACE`. Up to now, in terminal `enamcs -nw` can input Chinese character. But the Window Emacs will not. The solution is to emerge two fonts: `media-fonts/font-adobe-100dpi` and `media-fonts/font-adobe-75dpi`. You can search with Google the following Ebuild message for Emacs:

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

    4. About setting default system-wide editor, refer to [gentoo over lvm luks](http://jimgray.tk/2015/08/15/gentoo-over-lvm-luks/) and [emacs configuration](http://jimgray.tk/2014/09/14/emacs/installation/).
    5. Nano. We can tune some configs of Nano editor */etc/nanorc*: *set autoindent*, *set backup*, *set tabsize 4* etc. If need to totally disable an option, use *unset <option>*.
    6. WPS office.
       1. overlay support
           1. Refer to _New portage plug-in sync system_ below. If `portageq --version > 2.2.16`, install `layman >= 2.3.0`. The code below is for old layman version.
           1. \# echo "app-portage/layman git subversion" > /etc/portage/package.use/layman
           2. \# emerge -av layman
           2. \# echo "source /var/lib/layman/make.conf" >> /etc/portage/make.conf
           3. \# layman -f -a gentoo-zh
           4. \# layman -S

           >The previous version (<= 9.1.0.4953\_alpha18) located in `gentoo-zh` overlay relies on a bundle of customized and pre-built `QT` packages. Use command `qlist wps-office | grep -i qt` or `equery files wps-office | grep -i qt` to list the bundled `QT` libs in wps-office installation directory (/opt/kingsoft/wps-office/).  Since version `9.1.0.4953_alpha18-r1` wps-office was published through official Gentoo portage as well. The coming new versions no longer use those prebuilt `QT` libs. Instead, they will draw in *qtwebkit*, *qtscript*, *qttranslate*, *qtcore*, etc as system-wide packages.

           >But I prefer a Gentoo system without `QT` things since most of my packages are `GTK`-based, especially the `xfce4` desktop environment. So I will not install or update to the new versions.

           >Current method is to mask those higher version in `portage.mask/` directory: `>app-office/wps-office-9.1.0.4953_alpha18::gentoo`.
       2. WPS 32-bit `abi_x86_32` support. You should activate abi\_x86_32 use flag for packages on which WPS office depends.
           1. emerge -pv wps-office. It will reminds you what packages needs `abi_x86_32` support. Just answer 'Y' to the question and run command `dispatch-conf` to update configuration file. Of course you can update those files manually. You can also cange `pv` to `-av`.
           2. package.use/wps-office: check WPS overlay [wps-office-9.1.0.4945_alpha16_p3.ebuild](https://github.com/microcai/gentoo-zh/blob/master/app-office/wps-office/wps-office-9.1.0.4945_alpha16_p3.ebuild). You'd better use dispatch-conf to finish this work.
           3. package.license/wps-office: app-office/wps-office WPS-EULA
           4. package.accept\_keywords/wps-office: =app-office/wps-office-9.1.0.4945\_alpha16_p3 ~amd64

               >To use a specific software version from the testing branch but don't want portage to use the testing branch for subsequent versions, add in the version in the package.accept_keywords location. In this case use the = operator. It is also possible to enter a version range using the <=, <, > or >= operators. In any case, if version information is added, an operator must be used. Without version information, an operator cannot be used. Refer to [mixing branches](https://wiki.gentoo.org/wiki/Handbook:AMD64/Portage/Branches).
           5. emerge -av wps-office
           6. **fonts support** refer to [Fontconfig](/2015/04/13/fontconfig/)
    7. \# emerge -avt xfce4-mixer
    8. \# emerge -av mupdf

        The other PDF viewer may draw in a lot of GTK or QT dependencies consuming many disk space.
    9. \# emerge -av dev-vcs/git
       1. $ git config --global user.name "Jim Green"
       2. $ git config --global user.email "username@users.noreply.github.com"
       3. $ git config --global core.editor emacs
       4. \# emerge -av jekyll, it will install the rubygems, nodejs etc dependencies.
       5. $ git clone xxx

       Refer to [git config](http://jimgray.tk/2015/07/19/git-config/).
    10. \# emerge -av wgetpaste
    11. \# emerge -av net-misc/dropbox [optional] xfce-extra/thunar-dropbox
       1. Xfce4 and Dropbox does not get along well. There is no application menu for Dropbox.
       2. The system `LANG` or `LC_CTYPE` cannot be `zh_CN.GB18030`, otherswise dropbox does not launch with errors like _Gdk Critical...failed_.
           1. This can be overcome by setting the dropbox language to `english`.
       3. _$_ dropbox start, Gentoo and Windows share the Dropbox location in /media/Misc/Dropbox. When firstly run this command, you need to configure dropbox as default setting (Dropbox folder in /home/zachary/Dropbox). But then exit dropbox immediately. And remove /home/zachary/Dropbox.
           1. _$_ rmdir /home/zachary/Dropbox
           2. _$_ ln -s /media/Misc/Dropbox /home/zachary/Dropbox
       4. _$_ dropbox &, run dropbox in the backgroud.
       5. We can create a ~/bin/dropbox.sh or /usr/local/bin/dropbox.sh script:

           >\#!/bin/bash
           >
           >/opt/bin/dropbox &

           Remember to `chmod +x ~/bin/dropbox.sh`, just run `~/bin/dropbox.sh`.
       6. If blocked by GFW, set the corresponding proxy address and port.
    12. \# emerge -avt app-portage/gentoolkit
    13. \# emerge -avt app-portage/eix
       1. [deprecated for new poratge > 2.2.16] <s># emacs -nw /etc/eix-sync.conf:</s>

	  ```
	  # *

	  !!exec >/var/log/eix-sync.log ; chown portage: /var/log/eix-sync.log || true
	  ```

       2. \# eix-sync
    14. Archive
        1. \# emerge -av file-roller
        2. \# emerge -av thunar-archive-plugin

           Steps below might be deprecated depending on related package version

           1. Up to now, this is a bug in that `thunar-archive-plugin` cannot find a suitable archive manager. This is due a filename convention difference. The solution:
           2. \# cd /usr/libexec/thunar-archive-plugin/
           3. \# ln -s file-roller.tap org.gnome.FileRoller.tap
           4. After that, `thunar-archive-plugin` can find `file-roller` correctly. Refer to [thunar archive plugin cannot integrate with file-roller](https://forums.gentoo.org/viewtopic-t-1006838.html?sid=bce8eeef9eab8d916c59b01cef493bb4) and [doesn't work anymore with recent file-roller](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=746504).
        3. \# echo "app-arch/unzip natspec" > /etc/portage/package.use/unzip, this command is to let `unzip` support `GBK` Chinese filenames.
            1. 使用 “natspec” USE Flag重新编译unzip（zip文件中没有保存压缩时使用的编码，故需将unzip打上编码探测的补丁）
        4. [optionl, 7zip can extract zip format] # emerge -av unzip, up to now, Chinese `zip` file can be extracted correctly by `file-roller`.
        5. \# emerge -av p7zip, three commands `7z`, `7za` and `7zr` can be used to extract files.
            1. If `p7zip` is installed, `file-roller` takes `7z` or `7za` to handle `zip` file which cannot handle Chinese filenames.
            2. Solution: rename `/usr/bin/7z` and `/usr/bin/7za` to something else. 若需要安装p7zip，则安装完成后，移除或重命名/usr/bin下除7zr外7z*文件（fileroller在7z或者7za存在的情况下会优先使用。而7z和7za解压zip文件会出现文件名乱码，暂不知如何解决。故删除7z和7za，仅保留7zr以支持7z格式的压缩和解压。)
        6. Use `7z` format instead of `zip` since `7z` support unicode compression especially for filenames.
    15. [deprecated, use Alt+F2 app finder instead] <s># emerge -av xfce4-verve-plugin
        1. Add this plugin to panel.
        2. Under `Settings Editor` --> `xfce4-kerboard-shortcuts` --> `commands` --> `custom`, set a shortcut for this plugin: `Super + R`.</s>
    16. [deprecated] <s># emerge -av xfce4-wavelan-plugin</s>

        **this plugin cannot be added to panel currently**, cannot be used.
    17. kwplayer - Linx version of 酷我音乐
        1. \# emerge -av kwplayer

            Now play MV but not MP3 songs. After detailed search, I found that mplayer used `mad` to decode MP3. So:
        2. \# emerge -av1 gst-plugins-mad
 
            Bingo! This is due to the author did not test this package under Gentoo. So he did not incur the `gst-plugins-mad` dependcy specially for Gentoo. We need to install by ourself.
        3. The default configuration/log is located in `~/.config/kuwo/` and downloaded files are under `~/.cache/kuwo/`. Change the `song` and `mv` directories to `~/Music/Songs` and `~/Music/MVs` through GUI preferences menu. We cannot change the lyrics `lrc` location.
    19. bluetooth - bluez obexfs

        **Apple iOS does NOT support Obex protocol**. So don't test this part with your iphone.

        In the kernel config step, we have enabled several relevant modules to support bluetooth devices. Up to now, if we don't use bluetooth at all, there is no need to emerge `bluez` or `obexfs`. We can just edit the corresponding configuration file to disable bluetooth devices:

        >/sys/devices/platform/thinkpad\_acpi/bluetooth_enable

        >/proc/acpi/ibm/bluetooth

        1. \# emerge -av bluez, the current version is 5. `bluez` package is bluetooth protocol stack for Linux with several useful tools like `hciconfig`, `bluetoothctl`, etc. It also installs the `/etc/init.d/bluetooth` service daemon.

            >\# hciconfig -a

            >\# hciconfig hci0 up
        2. If `hciconfig` reminds error like:

                Can't init device hci0: Operation not possible due to RF-kill.

            >\# emerge -av net-wireless/rfkill

            >\# rfkill list

            >\# rfkill block 0

            >\# rfkill unblock 0

            `rfkill` not only supports bluetooth devices, but also other radio frequency ones. `rfkill` is a small userspace tool to query the state of the rfkill switches, buttons and subsystem interfaces. Some devices come with a hard switch that lets you kill different types of RF radios: 802.11 / Bluetooth / NFC / UWB / WAN / WIMAX / FM. Some times these buttons may kill more than one RF type. The Linux kernel rfkill subsystem exposes these hardware buttons and lets userspace query its status and set its status through a `/dev/rfkill`.
        3. Up to now, we can turn on/off bluetooth device by `rfkill` or `hciconfig`. Next is to pair bluetooth device. Firstly, we need to launch the bluetooth service `/etc/init.d/bluetooth`. If bluetooth is a common service, we can add it to default runlevel. To pair with and connect to other bluetooth device like your cell phone, use another command `bluetoothctl` from bluez 5.

            >\# rc-update add bluetooth default

            >\# rc-service bluetooth start

            >$ bluetoothctl

            >$ [bluetooth]# help

            Details on `bluetoothctl` pleae read the references.
        4. `bluez` commands are only for bluetooth connection. Now we need to transfer file between cell phone and PC by command lines:

            >\# emerge -av obexftp obexfs

            >\# obexfs -b MAC\_address\_of_device /media/Obex

            >\# fusermount -u /media/Obex

            We can combine `fuse` and `obexfs` together:

            >\# mount -t fuse "obexfs#-bMAC\_address\_of_device -B6" /media/Obex

            `obexfs` depends on `obexftp`. How to use `obexftp` along refer to references.
        5. `obexfs`, `obexftp`, `bluetoothctl` are command line tools - not efficient. Use GUI applicatibbon instead like `blueman` based on GTK+. Remember to add USE flag `thunar` for blueman, which will add a right click menu *Send To -> Bluetooth Device*. Refer to [arch wiki blueman](https://wiki.archlinux.org/index.php/Blueman).
            1. $ mkdir $HOME/Bluetooth
            2. \# ect /usr/local/bin/obex_thunar.sh

               ```bash
               #!/bin/bash
               fusermount -u $HOME/Bluetooth
               obexfs -b $1 $HOME/Bluetooth
               thunar $HOME/Bluetooth
               ```

            3. \# chmod +x /usr/local/bin/obex_thunar.sh
            4. The last step is to change the line in Blueman tray icon > Local Services > Transfer > Advanced to `obex_thunar.sh %d`.
        6. No matter blueman or bluetoothctl command is used, the basic procedure:
            1. \# rc-service bluetooth start, start bluetooth daemon.
            2. \# modprobe btusb, load bluetooth related modules.
            3. $ bluetoothctl OR blueman-applet OR by Bluetooth manager from App menu as a normal user account.
        7. Actually most of the time, we don't use bluetooth at all. So deactivate the bluetooth service and relevant modules at boot saves boot time and memory. Refer to *Module blacklist/install*.
        8. [gentoo bluetooth wiki](https://wiki.gentoo.org/wiki/Bluetooth); [archwiki bluetooth](https://wiki.archlinux.org/index.php/Bluetooth); [how to setup bluetooth](http://www.thinkwiki.org/wiki/How_to_setup_Bluetooth); [Linux下访问蓝牙设备的几种办法](http://blog.simophin.net/?p=537).
    22. TeXLive-2014

        Refer to [texlive gentoo](http://jimgray.tk/2015/08/29/texlive-gentoo/).
46. Configuration consistently.
    1. Package can be pulled or emerged into system. There is a big difference. The 1st won't add packages (pulled in by USE flag or dependency) to *@world* group, while the second do. The 2nd method implies that you *explicitly* emerged that specific package. Of course, you could use `--oneshot` option when emerging to avoid adding package to *world* group. More read [the 2nd step](http://jimgray.tk/2015/08/29/texlive-gentoo/) and *gst-plugins-* installation in this post.
    1. You use `1` *oneshot* when you want recompile or update a package (like *gst-plugins-*) that is a dependency of another package (like *kwplayer*) in a world file. So when you remove the second package depclean will be able to remove first one. When updating packages in *world* group, its dependencies won't get updated. If necessary, you need to update those dependencies by *oneshot* option.
    1. Try your best to avoid *package.accept_keywords*. *Tesiting* packages and other *stable* packages might cause conflicts in terms of *perl*, *ruby*, *python* etc.
    1. Update a single package.

       ```bash
       # emerge -avtuDN --with-bdeps=y pkg-name
       ```

    2. Slot conflict.

       When applying a @world update, it reminds the following slot conflicts:

       ```bash
       !!! Multiple package instances within a single package slot have been pulled
       !!! into the dependency graph, resulting in a slot conflict:

       dev-libs/openssl:0

	 (dev-libs/openssl-1.0.1p:0/0::gentoo, ebuild scheduled for merge) pulled in by
	   dev-libs/openssl:0=[-bindist] required by (net-libs/nodejs-0.12.6:0/0::gentoo, ebuild scheduled for merge)
			       ^^^^^^^^                                                                                                                  

	 (dev-libs/openssl-1.0.1p:0/0::gentoo, installed) pulled in by
	   >=dev-libs/openssl-0.9.6d:0[bindist=] required by (net-misc/openssh-6.9_p1-r2:0/0::gentoo, installed)
       ```

       `openssh` and `openssl` require the *same* `bindist` USE flag setting - *both* enabled or disabled. However, the scheduled package `nodejs` requires `openssl` disable `bindist` USE flag. So the solution is to disable `bindist` for `openssh`, `openssl` and `nodejs`.
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
       /dev/sda7 /media/WLShare ntfs-3g noauto,nls=utf8,locale=zh_CN.utf8,uid=zachary,gid=users,users,dmask=022,fmask=133 0 0
       ```

       1. We should create the corresponding directory under `/media/` NOT under `/mnt/`. The reason can be found here [What is the difference between mounting in fstab and by mounting in file manager](http://unix.stackexchange.com/questions/169571/what-is-the-difference-between-mounting-in-fstab-and-by-mounting-in-file-manager).
       2. Pay attention to `nls` support which help displaying Chinese filenames correctly.
       3. But when you create a new Chinese filename in Thunar and copy it to NTFS partition, errors same as above step appear. If you change the mount option in `/etc/fstab` to `en_US.utf8`, you can handle Chinese filenames between Thunar and ntfs partition smoothly which will eventually conflicts with the above step. So you can; creating new Chinese filenames in virtual terminal.
       4. The first line /dev/sda4 is the Windows partition, this will hide it from Thunar sidebar.
       5. I think the most important thing is: the `locale` in `fstab` should be the same as the one in system `LC_CTYPE`. Also as an English system supporting Chinese, `zh_CN.UTF-8` (or `zh_CN.utf8`) is better than `zh_CN.GB2312` or `zh_CN.GB18030`. The later ones are for pure Chinese systems. If set to `zh_CN.GB18030` or `zh_CN.GB2312`, the Chinese folder names cannot be displayed in Thunar's address bar.
       6. Refer to [Mounting a Local Microsoft Windows Partition on Linux Systems](http://docs.oracle.com/cd/E19253-01/819-0918/localization-13).
       7. By default, /boot not auto-mounted for security reason. Similarly, the EFI System Partition is not auto-mounted to `/boot/efi` as well at startup.
       8. `locale` is no longer needed. `nls=utf8` or `utf8` is not needed anymore since *ntfs-3g* defaults to *utf8*.
       9. Add `noatime` to improve performance.
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
       3. \# mkdir /etc/portage/repos.conf
       4. \# cp /usr/share/portage/config/repos.conf /etc/portage/repos.conf/gentoo.conf
          1. This default setting is enough for the official portage sync. The argument `sync-uri` can be changed to other mirrors like *rsync://rsync.cn.gentoo.org/gentoo-portage*.
          2. We can now set *sync-type = git* instead of *rsync*.

          ```
          [DEFAULT]
          main-repo = gentoo

          [gentoo]
          location = /usr/portage
          sync-type = git
          #sync-uri = https://github.com/gentoo-mirror/gentoo
          sync-uri = git://github.com/gentoo-mirror/gentoo.git
          auto-sync = yes

          # for daily squashfs snapshots
          #sync-type = squashdelta
          #sync-uri = mirror://gentoo/../snapshots/squashfs
          ```

          *sync-uri* can be either *https://* or *git://*. The trailing *.git* does not make any difference.

          3. Before switching to *git* sync. We should remove the old portage snapshot.

             ```bash
             \# mv /usr/portage/distfiles ~/
             \# mv /usr/portage/packages ~/
             \# rm -rf /usr/portage/*
             ```

          4. \# emaint sync -r gentoo
          5. \# mv ~/distfiles /usr/portage
          6. \# mv ~/packages /usr/portage
       5. Install new >=layman-2.3.0 supporting the new portage plug-in sync system.
          1. Add `sync-plugin-portage` USE flag to `/etc/package.use/layman` file.
          2. \# emerge -avt \>=app-portage/layman-2.3.0, the new layman package will create `/etc/portage/repos.conf/layman.conf` automatically.
          3. \# rm /var/lib/layman/make.conf, delete is the old-style layman config file.
       6. Edit `etc/portage/make.conf` and commnet out the lines `source /var/lib/layman/make.conf` and `SYNC="rsync://rsync.cn.gentoo.org/gentoo-portage"`.
       7. Sync operations have been moved from `emerge` to `emaint`. `emerge --sync` calls `emaint sync` module with the default `--auto` option. The `--auto` option performs a sync on only those repositories (both official portage and overlays) with the *auto-sync* NOT set to `no` or `false`. If absent, then default to `yes` / `true`. This means the original `emerge --sync` acts like `emaint sync` with default argument `--auto` or `-a`.
       8. As always `eix-sync` can update both overlays and portage while the new sync system will add overlays to `/etc/portage/repos.conf/layman.conf` as well. So when `eix-sync` is called, the new procedure is likely: `layman -S; emerge --sync`. But the new `emerge --sync` will also update overlays in `/etc/portage/repos.conf/layman.conf`.

          ```
          NOTE: As a result of the default auto-sync = True/Yes setting, commands 
          like "eix-sync", "esync -l", "emerge --sync && layman -S" will cause 
          many repositories to be synced multiple times in a row. Please edit 
          your configs or scripts to adjust for the new operation.
          ```

          9. To erase the duplicate updates incurred by `eix-sync` in new sync system, just remove `/etc/eix-sync.conf` or comment out `*` therein.
       10. Choose one of the follwing command for daily operation:
           1. \# emaint sync -a
           2. \# emerge --sync
           3. \# eix-sync

           Although they all update portage and overlays. However, *eix-sync* will call *eix-update* (for *eix* query) and *eix-diff* (showing what has changed) as well. So for daily management and eix operation, you'd better use *eix-sync*.
       11. [sys-apps/portage-2.2.16 发布，支持多种同步方式](http://www.gentoo-cn.info/article/new-portage-plug-in-sync-system/); [Gentoo的portage已支持直接更新第三方源（overlay）](http://phpcj.org/portage-emerge-overlay-on-gentoo/).
    6. Touchpad configuration. After X and Xfce4 installation, parts of Touchpad does not work. The primary method of configuration for the touchpad is through an Xorg server configuration file. After installation of `x11-drivers/xf86-input-synaptics`, a default configuration file is located at `/usr/share/X11/xorg.conf.d/50-synaptics.conf`. Users can copy this file to `/etc/X11/xorg.conf.d/` and edit it to configure the various driver options available. 
       1. \# emacs /etc/X11/xorg.conf.d/50-synaptics.conf

          ```
          Section "InputClass"
              Identifier "touchpad"
              Driver "synaptics"
              MatchIsTouchpad "on"
                  Option "TapButton1" "1"
                  Option "TapButton2" "2"
                  Option "TapButton3" "3"
                  Option "VertEdgeScroll" "on"
                  Option "VertTwoFingerScroll" "on"
                  Option "HorizEdgeScroll" "on"
                  Option "HorizTwoFingerScroll" "on"
                  Option "CircularScrolling" "on"
                  Option "CircScrollTrigger" "2"
                  Option "EmulateTwoFingerMinZ" "40"
                  Option "EmulateTwoFingerMinW" "8"
                  Option "CoastingSpeed" "0"
                  Option "FingerLow" "35"
                  Option "FingerHigh" "40"
          EndSection
          ```

       3. You can also set a temporary config at command line, which is not persisitent. Refer to command `synclient`.  
    7. Brightness key: Fn + Home/End.

       The thinkpad brightness key does not work even though I enabled the revelant `ThinkPad ACPI Laptop Extras, THINKPAD_ACPI`. The final solution is add a special kernel boot parameter **acpi_osi="!Windows 2012"**. This is to use the *standard ACPI* (advanced configuration and power management interfance) system instead of *thinkpad-acpi*.

       If want to make this change permanent, then modify the the `/etc/default/grub` template and update `/boot/grub/grub.cfg`.

       ```bash
       # cp /etc/default/grub /etc/default/grub_backup
       # ect /etc/default/grub, find the kernel parameter GRUB_CMDLINE_LINUX_DEFAULT="". By default it is empty and commented out. Insert the following value:
           acpi_osi=\"!Windows 2012\"
       or,
           acpi_osi='!Windows 2012'
       Get:
           GRUB_CMDLINE_LINUX_DEFAULT="acpi_osi=\"!Windows 2012\"".
       # mount /boot /boot/efi
       # grub2-mkconfig -o /boot/grub/grub.cfg
       ```

       The brightness config value is located in `/sys/class/backlight/`.

       Refer to [Thinkpad T430 brightness keys broken after firmware upgrade](https://bbs.archlinux.org/viewtopic.php?id=157600); [Brightness Key Levels T430](https://bbs.archlinux.org/viewtopic.php?id=158775); [arch wiki backlight](https://wiki.archlinux.org/index.php/Backlight); [Backlight keys stopped working, unless acpi_osi="!Windows 2012"](https://bugzilla.kernel.org/show_bug.cgi?id=51231).
    8. Module blacklist/install. Some modules are rarely used. It's better to deactivate it at boot to save time and memory. For example, to deactivate bluetooth-related modules.
       1. _$_ lsmod, to locate what modules are for bluetooth:

          ```
          btusb
            btbcm
            btintel
              bluetooth
          ```

          `modinfo | grep -i depends` help clarify the module dependencies. Here, modole `btusb` is the root module.
        2. So need to deactivate those 4 modules at boot.

           ```
           # /etc/modprobe.d/bluetooth.conf
           # disable bluetooth modules at boot since most of the time, don't need it at all.
           # Don't forget to remove `/etc/init.d/bluetooth` from *boot* or *default* runlevel if ever added.
           blacklist btusb
           #install btbcm /bin/true
           #install btintel /bin/true
           #install bluetooth /bin/true
           blacklist btbcm
           blacklist btintel
           blacklist bluetooth
           ```

        3. `blacklist mod-name` and `install mod-name /bin/true (or false)` methods are slightly different:

           For blacklisted modules, they will be loaded if another non-blacklisted module depends on it, or if it is loaded manually. For example, *bluetooth* is blacklisted. And now we need to pair cell phone with PC by bluetooth protocol. So we need to manullay `modprobe btusb`. *bluetooth* module will load as a dependency. 

           However, to ensure the modules (bluetooth service) are never inserted, even if they are needed by other modules you load or by manually modprobed, use `install` instead. To install a module as `true` or `false` does not make much difference. But `false` will remind you error message when you manually modprobe the module or a root module tries to load it. Refer to [Modprobe is better disabled by using /bin/true (not/bin/false](https://github.com/OpenSCAP/scap-security-guide/issues/539).

           If you want to **totally disable** the a service, `blacklist` or `install` root module; `install` all dependency modules.
 
           If you might need a service temporarily from time to time, `blacklist` root and all dependency modules.
        5. Refer to [arch wiki blacklist](https://wiki.archlinux.org/index.php/Kernel_modules#Blacklisting); [changes to module blacklisting](https://www.archlinux.org/news/changes-to-module-blacklisting/).
        6. Similarly, some other rarely used modules can also be deactivated from startup:

           Webcamera modules:

           ```
           # /etc/modprob.d/webcamear.conf
           # deactivate webcamera since rarely used.
           blacklist uvcvideo
           #install videobuf2_core /bin/false
           #install videobuf2_vmalloc /bin/false
           #install v4l2_common /bin/false
           #install videobuf2_memops /bin/false
           #install videodev /bin/false
           blacklist videobuf2_core
           blacklist videobuf2_vmalloc
           blacklist v4l2_common
           blacklist videobuf2_memops
           blacklist videodev
           ```

           thinkpad_acpi module:

           ```
           # /etc/modprobe.d/thinkpad_acpi.conf
           # thinkpad_acpi module does offer any useful function support.
           blacklist thinkpad_acpi
           ```

    9. Intel Microcde

       ```
       # emerge -av microcode-ctl
       # rc-update add microcode_ctl boot
       ```

       Refer to [intel microcode](https://wiki.gentoo.org/wiki/Intel_microcode). Updates to CPU microcode have to be re-applied each time the computer is booted, because the memory updated is volatile (despite the term *firmware* also being used for microcode).

       According to the reference, you should run `dmesg | grep -i microcode` to check whether CPU microcode is updated or not. If not, I think there is not need to add *microcode-ctl* to boot level until *microcode-ctl* package is updated.

       CPU的微代码更新支持,建议选中.CPU的微代码更新就像是给CPU打补丁,用于纠正CPU的行为.更新微代码的常规方法是升级BIOS,但是也可以在Linux启动后更新.比如在Gentoo下,可以使用"emerge microcode-ctl"安装microcode-ctl服务,再把这个服务加入boot运行级即可在每次开机时自动更新CPU微代码.
    10. Fingerprint. Fingerprint login is a bad idea since your fingerprint is left anywhere anytime around, like on bottles, cups, books etc. It easy for hackers to get a copy of it. So don't use it!!!

        Here, I just have a try and test the function. That's all of it. BTW, my current system is locked by [lvm luks lvm](/2015/09/10/lvm-luks-lvm/). So it's relatively safe.

        ```bash
        # lsusb | grep -i upek
        Bus 001 Device 004: ID 147e:2016 Upek Biometric Touchchip/Touchstrip Fingerprint Sensor
        ```
        My laptop fingerprint device *Upek 1473:2016*. Fingerprint don't need special device driver.

        1. \# emerge -av sys-auth/fprintd
        2. \# ect /etc/pam.d/system-local-login, add *auth sufficient pam_fprintd.so* to the beginning of the file.

           ```
           auth		sufficient	pam_fprintd.so
           auth		include		system-login
           account		include		system-login
           password		include		system-login
           session		include		system-login
           ```

           Among the */etc/pam.d/* files, *system-auth* is the most important for authentication. For example, if your need to *login*, then you need *authentication* first. So *system-login* file contains a line *auth include system-auth*.
       3. We can edit other files like */etc/pam.d/polkit-1* for GNOME polkit authentication. Add *auth sufficient pam_fprintd.so* to */etc/pam.d/xscreensaver* will help unlock screensaver.
       4. $ fprintd-enroll, wipe finger over the fingerprint reader for 5 times. Later on, we can *fprintd-delete* to remove the enrolled fingerprints.
       5. Reboot and input username, then it reminds *wipe your finger ...*.
       6. Don't enroll fingerprint for *root* account. If the fingerprint authention failed (3 times), it fall back to normal password login automatically.
       7. Refer to [configuring fprint PAM for all authentications [solved]]( https://forums.gentoo.org/viewtopic-p-6952448.html); [arch fprint](https://wiki.archlinux.org/index.php/Fprint); [how to enable fingerprint](http://www.thinkwiki.org/wiki/How_to_enable_integrated_fingerbbprint_reader_with_fprint).
    10. Intel VGA bug

        My Intel VGA is HD3000 series. Refer to [Hangs on Sandy Bridge](https://forums.gentoo.org/viewtopic-p-7319152.html), [freedesktop 54226](https://bugs.freedesktop.org/show_bug.cgi?id=54226), [archwiki SNA](https://wiki.archlinux.org/index.php/intel_graphics#SNA_issues) and [gentoo intel](https://wiki.gentoo.org/wiki/Intel).

        ```
        # less /var/log/messages

        May 13 15:47:12 myhost kernel: [drm] stuck on blitter ring
        May 13 15:47:12 myhost kernel: [drm] GPU HANG: ecode 6:2:0x00fffff7, in X [5230], reason: Ring hung, action: reset
        May 13 15:47:12 myhost kernel: [drm] GPU hangs can indicate a bug anywhere in the entire gfx stack, including userspace.
        May 13 15:47:12 myhost kernel: [drm] Please file a _new_ bug report on bugs.freedesktop.org against DRI -> DRM/Intel
        May 13 15:47:12 myhost kernel: [drm] drm/i915 developers can then reassign to the right component if it's not a kernel issue.
        May 13 15:47:12 myhost kernel: [drm] The gpu crash dump is required to analyze gpu hangs, so please always attach it.
        May 13 15:47:12 myhost kernel: [drm] GPU crash dump saved to /sys/class/drm/card0/error
        May 13 15:47:12 myhost kernel: drm/i915: Resetting chip after gpu hang
        May 13 15:47:12 myhost kernel: [drm:i915_reset] *ERROR* Failed to reset chip: -110
        ```

        Basically create a Xorg configuration file (i.e. */etc/X11/xorg.conf.d/20-intel.conf*):

        ```
        Section "Device"
        	Identifier      "intel"
        	Driver          "intel"
        	Option          "AccelMethod"   "sna"
        	Option          "TearFree"      "true"
        EndSection
        ```

        Though, *sna* is newser and faster than *uxa*. Must enable *TearFree*.

        After reboot, the X looks much better. The *stuck on* kernel error does not occur up to now.

    10. swapfile

        ```bash
        # fallocate -l 1G /mnt/1GB-swapfile
        # chmod 600 /mnt/1GB-swapfile
        # mkswap /mnt/1GB-swapfile
        # swapon /mnt/1GB-swapfile
        # swapon -s
        # ect /etc/fstab
        /mnt/1GB-swapfile none swap defaults 0 0
        ```

        Refer to [swap file creation](https://wiki.archlinux.org/index.php/Swap#Swap_file_creation).
    11. OpenRC Log

        Boot process messages are really useful for system bug tracking. By default, boot messages are not logged.

        ```bash
        # ect /etc/rc.conf
        rc_logger="YES"
        ```

        The default log file path is */var/log/rc.log*. You can change it by *rc_log_path* variable.
47. Upgrade kernel to **unstable 4.0.0**

    >Before updating to newest kernel version, you'd best update system @world. Refer to *Update the system*.

    1. \# echo "=sys-kernel/gentoo-sources-4.0.0 ~amd64" > /etc/portage/package.accept_keywords/gentoo-sources
       1. `eix` helps find out which unstable package version is located in portage mirror.
    2. \# eix-sync
    3. \# emerge -av gentoo-sources
    4. \# eselect kernel list
    5. \# eselect kernel set 2, this is update the `/usr/src/linx` symbol link pointing to the new 4.0.0 kernel.
    5. \# mount /boot
    5. \# mount /dev/sda2 /boot/efi
    6. \# cd /usr/src/linux
    7. \# cp ../linux-3.18.11-gentoo/.config ./linux/, copy the old kernel config to the new kernel sources directory
    8. \# make silentoldconfig, choose all the new settings to default ones. It only asks user new kernel options incurred in new kernel version.
       1. \# make olddefconfig, to convert the old config to fit new kernel version, while setting new kernel options to default values without user confirmation.
       2. \# make oldconfig, similar to `make silentoldconfig` excpet asking you the same kernel options between two kernel versions as well.
    8. \# make modules_prepare
    1. \# echo 3 > /proc/sys/vm/drop_caches
    9. \# make -j3
    10. \# make modules_install
    11. \# make install
    12. \# genkernel --install initramfs

        If Gentoo depends on LUKS and LVM for system mount points and booting, refer to [LVM LUKS LVM](http://jimgray.tk/2015/09/10/lvm-luks-lvm/).
    13. \# grub2-mkconfig -o /boot/grub/grub.cfg
    14. \# emerge -av @module-rebuild

        This should be ran **before reboot**.

        Any external kernel modules, such as binary kernel modules, need to be rebuilt for each new kernel. For example, Virtualbox will bring along its external module by package *app-emulation/virtualbox-modules*. After getting into system with new kernel and load 'vboxdrv' module, you might get errors:

        ```
        modprobe: FATAL: Module vboxdrv not found.
        ```

        What's worse, you might fail to boot since OpenRC cannot load the external modules.

        `emerge -av @module-rebuild` is to re-install all external modules (*app-emulation/virtualbox-modules* inclusive). More read [VirtualBox](http://jimgray.tk/2015/08/21/virtualbox/).

        There is another command from wiki `make modules_prepare` is used when the kernel not built yet or cleaned. For example, you need to compile the external module first, just before the kernel building, then execute this command before `make -j3`.
    15. \# reboot

47. e-sources-4.1.1 kernel

    Except the official default kernel source, there are plenty of sources maintained by other authors like the `e-sources` in `gentoo-zh` overlay. `e-sources` offer many extra features, of which the most important is the `cjktty` patch enabling Chinese character display in virtual terminal.

    To compile `e-sources`, the procedure is all the same as that for `gentoo-sources`. The only difference is to enable a few extra kernel options of `cjktty` patch. Refer to *e-sources / cjktty patch specific options*.
    2. <s>`e-sources` does not add `symlink` USE flag, so edit `/etc/portage/package.use/e-sources` add a line:

       ```
       # 'symlink' will update the 'linux' symbolic automatically whening emerging e-sources.
       sys-kernel/e-sources symlink
       ```</s>

       Use `eselect kernel set xx` after sources emerge instead.
    3. `e-sources-4.1.1` draws in `aufs` and `tuxonice` USE flags by default, which in return draws in `aufs-util` and `aufs-headers` packages. Currently, there is no need. Remove this USE flag for `slot 4.1` in `overlay gentoo-zh`. Add a line:

       ```
       # remove 'aufs' USE flag since it will draw in 'aufs-util' and 'aufs-headers' package. Current system does not need 'aufs' at all.
       sys-kernel/e-sources:4.1::gentoo-zh -aufs -tuxonice
       ```

    4. During the `make` process, it reminds warning:

       ```
       drivers/tty/vt/vt.c: In function ‘vc_do_resize’:
       drivers/tty/vt/vt.c:890:18: warning: ‘old_rows’ may be used uninitialized in this function [-Wmaybe-uninitialized]
         old_screen_size = old_rows * old_row_size;
        		 ^
       drivers/tty/vt/vt.c:890:18: warning: ‘old_row_size’ may be used uninitialized in this function [-Wmaybe-uninitialized]
       ```

       We can look into the `/usr/src/e-sources/drivers/tty/vt/vt.c` at line 890, we do find that issue. Anyway currently the kernel works fine.
48. Refer to [Version specifier](https://wiki.gentoo.org/wiki/Version_specifier) for specifying versions of packages as used when interacting with Portage via emerge or `/etc/portage`. These are also known as `DEPEND atoms` in Portage documentation.
48. Apply `cjktty` patch for kernel `4.0.5`.

    I want to display Chinese characters in *tty*, which can be achieved by `cjktty` patch.
    1. Get the patch for kernel `4.0.5`.

       This patch is now maintained by *microncai*. It is included in the *e-sources* package. Search `gentoo e-sources` in *Google*, and get [/gentoo-zh/sys-kernel/e-sources](http://data.gpo.zugaina.org/gentoo-zh/sys-kernel/e-sources/).

       We can get this patch from the *files/4.0/* sub-directory as [4.0.5-cjktty.patch](/assets/4.0.5-cjktty.patch). 

       If you cannot get the patch file, we can extract it manually [ git diff to get cjktty.patch](http://fangxiang.tk/2015/09/18/git/diff/patch).
    2. Check patch compatibility with `patch --dry-run` option.

       ```
       # cd /usr/src/linux-4.0.5/
       # patch --dry-run -p1 < /path/to/3.18.14-utf8.diff
       ```

       We get a *hunk* failure reminds. After check the patch file, locate the failure:

       ```
       @@ -2691,6 +2710,19 @@ static u16 *fbcon_screen_pos(struct vc_data *vc, int offset)
               unsigned long p;
               int line;

       +       if (offset < 0) {
       +               offset = -offset - 1;
       +               if (vc->vc_num != fg_console || !softback_lines)
       +                       return (u16 *)(vc->vc_origin + offset + (vc->vc_screenbuf_size));
       +               line = offset / vc->vc_size_row;
       +               if (line >= softback_lines)
       +                       return (u16 *) (vc->vc_origin + offset - softback_lines * vc->vc_size_row + (vc->vc_screenbuf_size));
       +               p = softback_curr + offset;
       +               if (p >= softback_end)
       +                       p += softback_buf - softback_end;
       +               return (u16 *) (p + (fbcon_softback_size));
       +       }
       +
               if (vc->vc_num != fg_console || !softback_lines)
                       return (u16 *) (vc->vc_origin + offset);
               line = offset / vc->vc_size_row;
       ```

       The issue lies at the line 4. You might think it a blank line. **IT IS A FAKE BLANK LINE**. It is composed of *one space* and *one tab* that is similar to *one +* and *one tab* for a code line that had been removed. The author *microcai* forget this remove the extra whitespaces before generating the patch file.

       Remove those two whitespaces in patch file to get a real blank line or add corresponding whitespaces in kernel souce file.
    3. Apply the patch.

       ```
       # cd /usr/src/linux-4.0.5
       # patch -p1 < /path/to/3.18.14-utf8.diff
       ```

    4. Similarly, we can choose other useful patches as well.

       Finally, I chose four patches:

       ```
       3.18.14-utf8.diff
       change-the-number-of-tty-devices.patch
       lower-undefined-mode-timeout.patch
       support-micmute-led.patch
       ```

    5. Config and build kernel as usual.
48. Remove old kernels

    `eclean-kernel` only remove files built when compiling kernels like libs, modules, etc. The kernel sources itself in `/usr/src/` will be managed by `portage` instead. So `eclean-kernel` don't remove sources itself! If you want to remove sources as well, use `emerge -ac gentoo-sources-version` and update `package.accept_keyword` and `package.use` files if need.

    1. \# emerge -av app-admin/eclean-kernel
    2. \# eclean-kernel -n 4 -p, the option `-p` is to pretend removal, just showing which kernels will be removed.
    3. \# eclean-kernel -n 4 -avt
       1. Option `-avt` is very important, it will ask for your confirmation for each kernel to be removed by an interactive way. You can just remove the kernels you don't need. You can also adjust the number from `4` to something else to pop out the desired kernel for removal.
       2. At last, I removed the kernel as  `3.18.11-gentoo.old`
    4. `eclean-kernel` will update the grub menu automatically.
    5. **special note**: the kernel is manually compiled by `make, make modules_install, make install, ...` following the basic procedures of official handbook. But `initramfs` is built by `genkernel` resulting in file names different from conventional format. You can `ls -l /boot` finding that `initramfs`'s file name format is different from that of `vmlinuz, System.map`. Tough the file name format won't cause problems since `grub2-mkconfig` is smart enough to generate the correct boot parameters. However, `eclean-kernel` will remind you removing `-genkernel-x86_64-4.0.0-gentoo: vmlinuz does not exist` which is an undesired action. Details refer to [Bug 464576](https://bugs.gentoo.org/464576)
49. Ref:
    1. [Gentoo on a ThinkPad X200](http://vminko.org/gentoo_manuals/thinkpad_x200)
    2. [thinkpad t440s gentoo](https://wiki.gentoo.org/wiki/Lenovo_ThinkPad_T440s)
    3. [Installing Gentoo on a ThinkPad X220](http://www.thinkwiki.org/wiki/Installing_Gentoo_on_a_ThinkPad_X220)
