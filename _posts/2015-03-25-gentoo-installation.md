---
layout: post
title: Gentoo Installation
---

> Gentoo installation along with Windows 8.1 and Ubuntu 14.04 with UEFI booting.

# Tips

1. Follow [Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64).
2. To *emerge* packages, you'd better read ArchWiki.
3. Ref:
   1. [Gentoo on a ThinkPad X200](http://vminko.org/gentoo_manuals/thinkpad_x200)
   2. [thinkpad t440s gentoo](https://wiki.gentoo.org/wiki/Lenovo_ThinkPad_T440s)
   3. [Installing Gentoo on a ThinkPad X220](http://www.thinkwiki.org/wiki/Installing_Gentoo_on_a_ThinkPad_X220)

# LiveDVD USB stick

1. Download the LiveDVD like *livedvd-amd64-multilib-20140826.iso* instead of Minimal installation CD like *install-amd64-minimal-20150319.iso*.
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
   1. The very first thing is to connect to Wi-Fi or Ethernet through networkmanager.
   2. Use shortcut F12 to Open/Retract Yakuake/Guake terminal in KDE destop.
   3. Refer to [Gentoo Ten LiveDVD Frequently Asked Questions](https://www.gentoo.org/proj/en/pr/releases/10.0/faq.xml).
4. `sudo su -` to *root*.

   You can use `passwd USERNAME` to change the password of *gentoo*. As *root*, you can change any account passworld by `passwd username`. All the password thing within the LiveDVD environment is volatile.

   The command prompt is *livecd ~ #* different from the handbook one - *root #*.

# Disk preparation

1. `parted -a optimal /dev/sda` or `fdisk /dev/sda`. Erase */dev/sda10* (NTFS partition) to install Gentoo on.

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
2. Newly created */dev/sda10* will be the *boot* partition while */dev/sda12* the *root* partition.

   We don't create Swap (*sda7*) or EFI (*sda2*) partition. Just share them with Ubuntu and Windows. You can also create a separate home partition, say *sda13*.
3. Format the new partition. You are suggested to format boot partition as *ext2*.

   ```bash
   # mkfs.ext2 /dev/sda10
   # mkfs.ext4 /dev/sda12
   # mkfs.ext4 /dev/sda13
   ```

4. *swap* partition needs *activate*d

   ```bass
   # mkswap /dev/sda7 (opt)
   # swapon /dev/sda7
   ```

   If you prefer an independent *swap* partition, *mkswap* to format it firstly.
5. Mount partitions.

   ```bash
   # mount /dev/sda12 /mnt/gentoo
   # mkdir -p /mnt/gentoo/boot/efi && mount /dev/sda10 /mnt/gentoo/boot && mount /dev/sda2 /mnt/gentoo/boot/efi
   # mkdir /mnt/gentoo/home && mount /dev/sda13 /mnt/gentoo/home
   ```

# Install *stage3*

1. Check date and time using `date` command. Set by `date MMDDhhmm` if it's incorrect.
2. Downloading and verify *stage3* tarball. Usually, it's downloaded beforehand along with LiveDVD ISO.
3. Now unpack the downloaded stage onto the system.

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

1. `CFLAGS` == `CXXFLAGS`

   Check your CPU architecture to set `-march=` parameter. Refer to [Intel](https://wiki.gentoo.org/wiki/Safe_CFLAGS#Intel). `grep -m1 -A3 "vendor_id" /proc/cpuinfo` offers CPU architecure. That link also teach you how to precisely detect `-march=` parameter by touching, compiling and comparing two *.gcc* files.

   1. CFALGS="-march=sandybridge -O2 -pipe"
   2. CXXFLAGS="${CFLAGS}"

2. The `MAKEOPTS` variable defines how many parallel compilations should occur when emerging packages.

   The recommended value is the number of logical processors in the CPU plus 1. But this rule is outdated. I have 4GB memory and *swap/swapfile* enabled, *emerge* will make use of *swap* havily. *swap* naturally slow down application performance though support more parallel tasks. So turn down to 2 or 3 to reduce usage of *swap* file.

   1. Add a line `MAKEOPTS="-j3"`.
   2. The boot screen shows several penguins, that is the number of logical hardware cores. If kernel `X86_SYSFB` and `FB_SIMPLE` were turned off, you could not see boot penguins.
   3. This value does not apply to *kernel compiling*. We explicitly specify *make -j3*.
   4. Refer to [MAKEOPTS](https://wiki.gentoo.org/wiki/MAKEOPTS).

3. `CPU_FLAGS_X86`: The 'USE' flags corresponding to the CPU instruction sets and other features specific to the *x86/amd64* architecture are *being* moved into a separate USE flag group called *CPU\_FLAGS_X86*.
   1. Refer to [cpu\_flags_x86 instruction](https://www.gentoo.org/support/news-items/2015-01-28-cpu_flags_x86-introduction.html).
   2. Emerge *app-portage/cpuinfo2cpuflags* and run in LiveDVD. Edit 'make.conf' to set *CPU\_FLAGS_X86*.
   4. Up to now, some packages in *portage* and overlays are not yet migrating those flags from 'USE' to 'CPU\_FLAGS\_X86'. So:
   5. Remove the old CPU specific flags from 'USE'. Then add *${CPU\_FLAGS_X86}* to 'USE'.

# Chrooting

1. `cp -L /etc/resolv.conf /mnt/gentoo/etc/`,  to ensure that networking after chrooting.
2. Mounting the necessary filesystems.

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

1. Selecting mirrors
    
   ```bash
   # mirrorselect -s3 -b10 -o -D >> /mnt/gentoo/etc/portage/make.conf (opt)
   ```

   Choose the 3 fastest mirrors for package sources downloading. This will takes around 10 minutes. So usually, we just manually choose 3 physically close servers.
2. [deprecated] <s>Portage sync</s>

   ```bash
   # mirrorselect -i -r -o >> /mnt/gentoo/etc/portage/make.conf
   ```

   Selects the *rsync* server to use when synchronizing portage tree. It is recommended to choose a *rotation link*, such as *rsync.us.gentoo.org*, rather than choosing a single mirror (i.e. *ftp*). This helps spread out the load and provides a fail-safe in case a specific mirror is offline.

3. The new plug-in [sync system](https://wiki.gentoo.org/wiki/Project:Portage/Sync) (*>=sys-apps/portage-2.2.16*)

   The new *plug-in sync system* currently supports *rsync, git, svn, websync, webrsync, cvs, laymansync* synchronizing types. And configurations reside now under */etc/portage/repos.conf/*.

   After installing *stage3*:

   ```bash
   # mkdir /mnt/gentoo/etc/portage/repos.conf
   # cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
   ```

# Portage tree

1. Portage snapshot

   ```bash
   # emerge-webrsync
   ```

   *webrsync* type is used to create a complete Portage tree. It might complain about a missing */usr/portage/* location. This is to be expected and nothing to worry about - it will be created on demand.
2. Profile

   ```bash
   # eselect profile list
   # eselect profile set 3
   ```

   Choose the *desktop* profile instead of *desktop/gnome* or *desktop/kde*. We will install XFCE later on.
3. USE flags

   Try `emerge --info | grep ^USE` to check USEs set by profile and *make.conf*. Refer to [Xfce/Guide](https://wiki.gentoo.org/wiki/Xfce/Guide#The_basics).

   ```
   USE="${CPU_FLAGS_X86} -bindist -qt4 -libav vaapi"
   ```

   By default, *bindist* is enabled by *stage3*'s default *make.conf*. If we don't plan to distribute packages, remove it. *bindist* may cause conflicts, i.e. between *openssh* and *openssl*.

# Localization

1. [Time](https://unix4lyfe.org/time/)
   1. Prime Meridian was (arbitrarily) chosen to pass through the Royal Observatory in Greenwich.
   1. UTC (previously called Greenwich Mean Time GMT): time at zero degrees longitude (the Prime Meridian) is called Coordinated Universal Time.
   1. Unix time: Measured as the number of seconds since epoch (the beginning of 1970 in UTC). Unix time is not affected by time zones or daylight saving.
   2. Local Time (UTC + Timezone) is what we use daily.

      Setting operating Software Time equal correctly to Local Time is our goal.
   3. Hardware Time resides on BIOS, which can be set manually and updated by operating system. Both Windows and Linux sets Software Time based on Hardware Time.

      It's regarded as *permanent placeholder* to set Software Time.
   4. Windows treats Hardware Time as Local Time and sets Software Time identically to Hardware Time without any translation on boot. Timezone is to synchronize time with time server. Upon shutdown, Windows writes back Software Time to Hardware Time.
   5. Linux treats Hardware Time as UTC and adds Timezone (i.e. +8) to Hardware Time on boot. Similarly, Software Time is written back to Hardware Time.
   6. Suppose Windows Software Time and Hardware Time are correctly synched to Local Time, Linux (dual boot installation afterwards) Software Time will be Hardware Time + Timezone = Windows Software Time + Timezone = Local Time + 2xTimezone. So usually Linux Software Time is ahead of Localtion for Timezone.

      If we shutdown Linux and boot Windows, Windows Software Time will be ahead of Local Time too. Each time we switch back and forth, Software Time will increase linearly by Timezone step.
   7. We must tell Linux that Hardware Time is Local Time instead of UTC *before* Software Timezone is configured.

2. hwclock

   Set *clock=local* in */etc/conf.d/hwclock*. We can find other options concerning reading/updating Hardware Time.

   Afterwards, set timezone information.

3. Timezone

   This step should be after */etc/conf.d/hwclock* update. Otherwise the Linux Software Time is usually ahead of local time by 8 hours, thus resulting in portage tree time stamp issues.

   ```bash
   # ls /usr/share/zoneinfo
   # echo "Asia/Shanghai" > /etc/timezone
   # emerge --config sys-libs/timezone-data
   # date
   ```

4. [locale](https://www.ibm.com/developerworks/cn/linux/l-cn-linuxglb/)

   The *locale* format is like *xx_YY.ZZ*, where *xx* and *YY* denote lanugage code and country code respectively. *ZZ* stands for charset (encoding/decoding). *xx* and *YY* mainly affects GUI (DE, app menus etc.), while *ZZ* takes care of encoding/decoding.

   *eselect locale set* defines all locale settings at once by *LANG* varaible while allowing further individual customization via the *LC_\** sub-options (i.e. *LC_CTYPE*).

   ```bash
   # cat /usr/share/i18n/SUPPORTED \| grep zh_CN >> /etc/locale.gen
   # uncomment en_US.UTF-8 UTF-8
   # locale-gen
   # locale -a
   # eselect locale list
   ```

   In the 3rd step, if reminded to run "*. /etc/profile*" to reload the variable, just remember `export PS1="(chroot) $PS1"`.

   Use *xx_YY.UTF-8* (some applications does NOT recognize *xx_YY.utf8*) instead of the illegal format *xx_YY.UTF8*. How to achieve this? Use the *free form* of Gentoo *eselect*.

   ```
   # eselect locale set en_US.UTF-8
   # locale
   ```

   *LC_\** sub-options can be further set in */etc/env.d/02locale*:

   ```
   LANG="en_US.UTF-8"
   LC_COLLATE="C"
   LC_CTYPE="zh_CN.UTF-8"
   ```

   If you don't have privileged access, *export* them in shell RC file like *.bashrc*.

   Remember to

   ```bash
   # env-update && source /etc/profile && export PS1="(chroot) $PS1"
   # locale
   ```

5. Chinese

   Why English display is not a concern? You may say that English is the mostly used accross the world and hence mostly used by programmers and the applications thereof. Yep, that's right but superficial. The underlying core is that ASCII table (English characters) is covered by nearly all encoding schemes present, GB2312/GBK/GB18030 included! No matter which encoding scheme is chosen, English is always correctly displayed.

   To be specific, language display is divided into two aspects:

   1. GUI (i.e. menu, popup box, botton, log etc.).
   2. File content which is what we refer to without explicit explanation (i.e. a HTML page).

   Let's talk about GUI first. You may be confused. Most applications GUI use ASCII characters (i.e. popup error dialog), thus correctly displayed no matter what system locale is set. But applications written for Chinese like QQ use Chinese GUI by default. What if the author could not be bothered to offer English GUI? Then we have to tune system locale to Chinese.

   What if you would like more Chinese on your system GUI, which looks confortable? That's where *nls* and *l10n_\** (*linguas_\** will be deprecated) USEs play a role. Many applications supports either *nls* or *l10n*, which is the part users can control. The localized/translated GUI messages is stored in */usr/share/locale/<locale>/LC_MESSAGES/<package>.mo* files. *nls* installs all possible locale messages, while *l10n_\** only installs the locale message specified by USE. The system locale must be *LANG=zh_CN.ZZ* to let installed locale messages displayed, where ZZ charset must covers zh_CN characters (GB2312 GBK GB18030). If you only set sub-option *LC_CTYPE=zh_CN.ZZ*, then GUI sticks to English.

   When it comes to file content display/updating, it depends on the context. Client browser decodes HTM page by the *charset* tag. Emacs is smtart at detecting file encoding. Mousepad is stupid and always decode by system locale. And will ask for user confirmation upon failure. However, for Unicode-based encoding, it's easy to guess the encoding through BOM. Modern applications is smart at detection. If the application fails to detect file encoding, it may default to *ZZ* part of locale, to which many comand line tools belongs. 

   Let's go on details of *ZZ* part. For Chinese, it should be one of GB2312, GBK, GB18030 and UTF-8. Apart from the fallback decoding role, it mainly determine encoding of user generated contents (i.e. default encoding of new file).

   Remember that:

   1. Chinese characters are encoded and stored. Upon display, the encoding scheme should be found (by GTK/QT, text editor, browser, even by user involvement).
   2. The advantage of setting *ZZ* to UTF-8 is that it's widely used and reduces mojibak.
   3. UTF-8 requires more disk space (50% higher) for CJK characters.

   It's not over yet. Up to now, only character encoding is detected. In order to display the character, relevant font should be matched (i.e. by Fontconfig). Translation between the character encoding and *glyph* is done through font's intermediate *charmap/cmap/bmp* files. *cmap* is  a table mapping character encoding to glyph's *internal index*. Each font may include multiple *cmap* files corresponding to different character encodings (i.e. GBK, UTF-8). Then font engine (i.e. FreeType2) renders the returned *glyph*.

   Attention, *cmap* itself is encoded as well, mostly by Unicode (namely UTF-16/UCS-2, NOT UTF-8).

   Details on XFT, refer to [fontconfig](/2015/04/13/fontconfig/).

# Kernel building

1. Kernel sources tree

   ```bash
   # emerge -avt sys-kernel/gentoo-sources
   # eselect kernel list/set
   # ls -l /usr/src && cd /usr/src/linux;
   # git apply [--whitespace=warn] [--numstat] [--check] < /path/to/cjktty.patch
   or
   # patch [--dry-run] -p1 < /path/to/cjktty.patch (opt)
   ```

   For the first time we install kernel sources, */etc/src/linux* symlink is created automatically.

   If possible, apply kernel patches like *cjktty.patch*.
2. Tips
   1. You'd best have a *.config* backup to start with.
   2. If possible, try *sys-apps/pciutils, sys-apps/usbutils, and sys-apps/hwinfo* in LiveDVD. Don't emerge those packages in *chroot*, it's not necessary.
   3. During kernel config, search the kernel options on page [Linux-3.10-x86_64 内核配置选项简介](http://www.jinbuguo.com/kernel/longterm-3_10-options.html) and refer to LiveDVD's configuration.
   4. Try `lspci -n` and paste it's output to [device driver check page](http://kmuto.jp/debian/hcl); that site gives kernel options that must be enabled.
   5. We can refer to the LiveDVD's kernel config directly.
   6. Search with `/`. When search "snd-hda-intel", replace the hypen with dash.

       Usually there are several search outputs numerated (1, 2, 3 ...). Press the number to enter the kernel option.
   7. *exit* or two successive ESCs to get back.
   8. When confronted with issues related to kernel options, we can choose M instead of Y which might be a solution.
3. *.config* backup

   Suppose we have an old *.config* backup:

   ```bash
   # cd /usr/src/linux
   # cp /path/to/.config-backup .config && chmod -x .config
   # make help
   # make silentoldconfig
   ```

   1. If the *.config* backup belongs to the same kernel version, we omit *make silentoldconfig*.
   2. *oldconfig* asks for both new and old options.
   3. *silentoldconfig* only asks for NEW options while preserving old ones. Press ENTER to choose default setting.
   4. *olddefconfig* is similar but sets NEW options to default without confirmation.

   We might tune kernel options by graphical interface *make menuconfig* below.
4. Menuconfig

   ```bash
   # cd /usr/src/linux
   # make menuconfig
   ```

   1. `i915 e100e snd-hda-intel iTCO-wdt ahci i2c-i801 iwlwifi sdhci-pci`.

       Basic dirvers that needs activated.

   1. `IKCONFIG` should be Y instead of M. This allows you to inspect (by `./scripts/extract-ikconfig /path/to/vmlinuz`)the configuration of the kernel while it is running, without having to worry whether you've changed or cleaned the source directory after it was built. `IKCONFIG_PROC` stores kernel config at */proc/config.gz*.
   2. `NR_CPUS` set to 4 since laptop has 2 cores and 4 threading. The smaller this value is, the smaller the kernel image is and the less memory is consumed by kernel.
   2. `MICROCDE` and `MICROCDE_INTEL` are Y by default. Refer to *microcode-ctl* below.
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
   1. Set `FAT_DEFAULT_CODEPAGE` to 936 (without prefix *cp*), `FAT_DEFAULT_IOCHARSET` uses default *iso8859-1* (this value should include the prefix *cp* such as *cp936*) but turn on it's sub-option `FAT_DEFAULT_UTF8`.

      Enable `NLS_CODEPAGE_936` to Y or M. Current *initramfs* reads LUKS key from VFAT USB partition on boot, MUST set to be Y, otherwise it would fail to find (mount) the key device.

      Turn off `NLS_CODEPAGE_437` and `NLS_ASCII` since we either use UTF-8 or GBK system. Keep `NLS_ISO8859_1` since it's the default value of `FAT_DEFAULT_IOCHARSET`.
   2. Dm-crypt. *Device mapper support* = `BLK_DEV_DM` is set Y by default. *Crypt target support* = `DM_CRYPT` must be M or Y. *XTS support* = `CRYPTO_XTS` and *AES cipher algorithms (x86_64)* = `CRYPTO_AES_X86_64` optionally set to M (recommended). Refer to [Dm-crypt](https://wiki.gentoo.org/wiki/Dm-crypt). Refer to *Cryptsetup* step below. `CRYPTO_SERPENT` and `CRYPTO_SHA512` must be Y instead of M if relevant LUKS algorithms are used. Refer to [gentoo over lvm luks](2015/08/15/gentoo-over-lvm-luks/).
   3. Iptables. `NETFILTER_ADVANCED` & `XT_MATCH_OWNER` (*-m owner*). `IP_NF_TARGET_REDIRECT` (-j REDIRECT) which will enable `NF_NAT_REDIRECT` (in return, `NETFILTER_XT_TARGET_REDIRECT` be enabled as well). We only need `NETFILTER_XT_TARGET_REDIRECT` for iptables REDIRECT, so to simplify kernel, just enable it alone. If necessary, turn on `IP6_NF_NAT` (`NF_NAT_IPV6` as dependency) to enable ip6tables *nat* table. Turn on `NETFILTER_XT_MATCH_MULTIPORT` for *-m multiport*
   4. System log: `SECURITY_DMESG_RESTRICT` to Y. Refer to [Restrict unprivileged access to kernel syslog](https://lwn.net/Articles/414813/).

      More refer to *syslog-ng* below.
   5. Filesystem. Set `UDF_FS` as M to mount ISO file.
   6. SquashFS. `CONFIG_SQUASHFS` as M; `CONFIG_SQUASHFS_XATTR` and `CONFIG_SQUASHFS_XZ` as Y. (*sys-fs/squashfs-tools*).
   7. IO scheduler - BFQ: `IOSCHED_BFQ`, `BLK_CGROUP` and `BFQ_GROUP_IOSCHED` as Y.
   8. Magic SysRq key `CONFIG_MAGIC_SYSRQ` as Y. `CONFIG_MAGIC_SYSRQ_DEFAULT_ENABLE` configure default value of */proc/sys/kernel/sysrq*.
   9. [WireGuard](https://wiki.gentoo.org/wiki/User:Maffblaster/Drafts/WireGuard): `NET_FOU` (`NET_UDP_TUNNEL`) and `PADATA`.
   1. (opt) Thinkpad-related. *ThinkPad ACPI Laptop Extras* = `THINKPAD_ACPI` set to M.
   2. [e-sources / cjktty patch specific options].`FONTS` and `FONT_8x16` set to Y. And *console 16x16 CJK font ( cover BMP )* = `FONT_16x16_CJK` which is *cjktty* patch. These options are for Chinese characters display in TTY (Ctrl + Alt + Fn).
   3. Reference links: [Linux-3.10-x86_64 内核配置选项简介](http://www.jinbuguo.com/kernel/longterm-3_10-options.html); [Linux Kernel in a Nutshell](http://www.kroah.com/lkn/); [kernel-seeds](http://kernel-seeds.org/); [device driver check page](http://kmuto.jp/debian/hcl); [How do you get hardware info and select drivers to be kept in a kernel compiled from source](http://unix.stackexchange.com/a/97813); and [Working with Kernel Seeds](http://kernel-seeds.org/working.html).

5. Compiling

   ```bash
   # cd /usr/src/linux
   # mount /dev/sda10 /boot; mount /dev/sda2 /boot/efi
   # make clean/mrproper/distclean; echo 3 > /proc/sys/vm/drop_caches
   # make modules_prepare
   # make -j3 && make modules_install && make install
   # genkernel --install initramfs
   # grub-mkconfig -o /boot/grub/grub.cfg
   # emerge -avt @module-rebuild
   # reboot
   ```

   1. Make sure we are in kernel sources directory.
   2. This is usually done before chrooting.
   3. Clean previous building leftovers.
      1. *clean* removes most generated files but keep the *.config* and enough build support to build external modules (such that we overlook *make modules_prepare*).
      2. *mrproper* removes all generated files + *.config* + various backup files.
      3. *distclean* = *mrproper* + remove editor backup and patch files.
      4. *drop_caches* is set to 3.

      They are chosen if we re-compile kernel sources in case of compiling error. What's worse, a successful re-compiling generates unworkable kernel binaries.

      Please be noted that *mrproper* and *disclean* will remove backup files, especially the *.config*.
   4. *make modules_prepare* is somewhat complicated.

      External kernel modules (i.e. self-written codes; VirtualBox binary kernel modules) are built against kernel tree (*/usr/src/linux/*). That's because external modules building need support from kernel sources support (i.e. kernel sources' head file).

      So the kernel tree should be prepared to support external kernel modules building, which can be achieve by several ways:

      1. If the kernel tree is brand new (i.e. just unpacked) and we will build external kernel modules before kernel building, we should prepare by *make modules_prepare* under */usr/src/linux/*.
      2. If the kernel have already been built (i.e. *make -j3*), it's already prepared. Need to prepare.
   5. Compiling.
      1. Compiles the kernel. Keep `-j3` since MAKEOPTS in *make.conf* does not apply to kernel compiling.
      2. Install kernel modules into */lib/modules/\`uname -r\`*. If we are re-compiling the kernel, old modules will be overriden. Ether backup the old modules or install modules to a new location (by `INSTALL_MOD_PATH`) when we want the old kernel be bootable (i.e. we remove a module from kernel).
      3. Install kernel binaries *System.map, config, initramfs, vmlinuz* into */boot* and rename old identical version kernel binaries on demand. We can copy and even rename those binaries manually as long as filenames are kept consistent.
   6. For LVM/LUKS containers, emerge *genkernel* with *cryptsetup* USE.

      Extra arguments `--lvm --luks --gpg --busybox` should be supplied upon generates *initramfs*. Refer to [Gentoo rootfs over LVM encrypted in LUKS container](/2015/08/15/gentoo-over-lvm-luks/) and [lvm luks lvm](/2015/09/10/lvm-luks-lvm/).

      Like modules, *genkernel* does not backup *initramfs* automatically when re-compiling the kernel. We are responsible for bakcuping manually, especially when the *genkernel* arguments are different.
   7. Append current kernel to Grub menu.
   8. Although we are on the old kernel, *@module-rebuild* knows how to re-install external kernel modules for the new kernel as long as */usr/src/linux* symlink pointing the new kernel source tree (see *eselect kernel list*).

      If we are re-compiling the kernel, this is not a requirement.
7. Linux firmware

   Some drivers require additional firmware to be installed on the system before they work. This is often the case for network interfaces, especially wireless network interfaces.

   ```bash
   # emerge -avt --fechonly sys-kernel/linux-firmware
   ```

   Firstly just fetch the package sources. If network failed connect after booting into new kernel, emerge it then.

#  System configuration

1. *fstab*

   Creating the */etc/fstab* file. Use backup at best. The default */etc/fstab* file provided by Gentoo is not a valid fstab file but instead more of a template.

   ```
   /dev/sda10   /boot		ext2    defaults,noatime	1 2
   /dev/sda12   /   		ext4    noatime	       0 1
   /dev/sda7    none	  	swap	sw	       0 0
   proc         /proc          proc    nosuid,nodev,noexec,hidepid=2,gid=wheel 0 0
   none         /tmp           tmpfs   nodev,nosuid,noexec     0 0
   ```

   Double-check the */etc/fstab* file, save and quit to continue. Fault *fstab* results in boot failure.
9. *hostname* and *domain*
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
1. Enable OpenRC log

   Edit */etc/rc.conf* and set *rc_logger="YES"*.

1. Set root password

   ```bash
   # passwd
   ```

# [Networking](/2016/03/01/networking/)

1. *dhcpcd*

   We don't follow [Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64)'s *net config*. *net-misc/netifrc* requires support from DHCP, while *net-misc/dhcpcd* can handle network configuration alone.

   ```bash
   # emerge -avt net-misc/dhcpcd
   # rc-update add dhcpcd default
   # emerge -avt net-wireless/wpa_supplicant
   ```

   1. Personal DNS servers can be put into */etc/resolv.conf.head* and/or */etc/resolv.conf.tail*.
   2. In case of the network interface card should be configured with a static IP address, entries can also be manually added to */etc/dhcpcd.conf*.
   3. If need GUI tool, use NetworkManager instead of *wicd* since the later one don't support *nl80211* driver. Also Networkmanager depends on *wpa_supplicant* and *dhcpcd* or *dhcpclient*.
   4. Reference: [Network management using DHCPCD](https://wiki.gentoo.org/wiki/Network_management_using_DHCPCD); [wpa_supplicant](https://wiki.gentoo.org/wiki/Wpa_supplicant); [Handbook:AMD64/Networking/Wireless](https://wiki.gentoo.org/wiki/Handbook:AMD64/Networking/Wireless); [configuration example](http://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf); [wpa_supplicant.conf for sMobileNet in HKUST](http://blog.ust.hk/yang/2012/09/21/wpa_supplicant-conf-for-smobilenet-in-hkust/); [wpa_supplicant.conf](http://www.freebsd.org/cgi/man.cgi?wpa_supplicant.conf).

2. *wpa_supplicant*

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

   *dhcpcd* invokes *wpa_supplicant* through */lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant* hook. From *dhcpcd-6.10.0* onward, *10-wpa_supplicant* hook is no longer there by default. We should copy */usr/share/dhcpcd/hooks/10-wpa_supplicant* to */lib/dhcpcd/dhcpcd-hooks/10-wpa_supplicant*.

   `lspci -k` shows wireless driver in use is *iwlwifi*. However if specify `-D iwlwifi` on command line, *wpa_supplicant* will fail:

   >Unsupported driver 'iwlwifi'

   The actual driver in use was *nl80211*.

   1. If you have installed `net-misc/netifrc` (by default from *stage3*) and created `/etc/ini.d/net.*` and `/etc/conf.d/net` files, refer to [Migration from Gentoo net.* scripts](https://wiki.gentoo.org/wiki/Network_management_using_DHCPCD#Migration_from_Gentoo_net..2A_scripts).
   3. *wpa_supplicant-2.4* might cause authentication problem for PEAP Wifi. Refer to [Downgrade Package && wpa_supplicant && local overlay](/2015/05/11/gentoo-downgrade-package/)

# System tools

1. Portage tools

   ```bash
   # emerge -avt eix; eix-update
   # emerge -avt gentoolkit
   ```

2. System logger.

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
3. Cron daemon

   A cron daemon executes scheduled commands. It is very handy if some command needs to be executed regularly (for instance daily, weekly or monthly).

   ```bash
   # echo "sys-process/cronie anacron" > /etc/portage/package.use/cronie (opt)
   # emerge -avt sys-process/cronie
   # rc-update add cronie default
   ```

   1. *anaron* USE is not must if no rigid shechuled tasks.
   2. *cronie* must be in a runlevel to shedule *logrotate*.
   1. Details on running scheduled tasks based on input from the command *crontab*, refer to [Cronie and Anacron](/2015/07/19/cronie/).

4. More

   ```bash
   # emerge -avt sys-apps/mlocate, file indexing
   # emerge -avt sys-fs/ntfs3g, mounting NTFS filesystem
   # rc-update add sshd default, incoming SSH connection (opt)
   ```

# Bootloader - [Grub2](https://wiki.gentoo.org/wiki/GRUB2)

1. Installation

   ```bash
   # echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
   # emerge -avt grub:2 os-prober
   # mount /dev/sda10 /boot; mount /dev/sda2 /boot/efi
   # grub-install --target=x86_64-efi
   ```

   1. This must be the 1s step. Otherwise it would show *error: /usr/lib/grub/x86_64-efi/modinfo.sh doesn't exist*.
   2. Default to slot 2. If there are other operating system, *os-prober* is suggested.
   3. Make sure *boot* and EFI partition are mounted. Because Gentoo, Ubuntu, Windows share the EFI partition, we mount the shared EFI partion here.
   4. To install Grub and EFI system to media.

2. [deprecated] Manually Grub menu

   As long as the */boot* and */boot/efi* partitions are mounted, *grub-mkconfig* automatically detects other operating systems and writes to */etc/grub.d/30_os_prober* through *sys-boot/os-prober*.

   Chainloading Windows system manully in */etc/grub.d/40_custom*. The traditional `chainloader +1` does not boot Windows based on UEFI.

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

   The next is to replace `$hints_string` and `$fs_uuid`. This is where `os-prober` comes into playing a role.

   ```
   # grub2-probe --target=hints\_string /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi
   # grub2-probe --target=fs\_uuid /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi
   ```

   1. Print `$hints_string`.
   2. Print `fs_uuid`.
   3. Now replace the relevant values.
   4. Refer to [Windows installed in UEFI-GPT Mode menu entry](https://wiki.archlinux.org/index.php/GRUB\#Windows_installed_in_UEFI-GPT_Mode_menu_entry) and [Can GRUB2 share the EFI system partition with Windows?](http://unix.stackexchange.com/q/49165).

3. Generate grub menu

   ```bash
   # grub2-mkconfig -o /boot/grub/grub.cfg
   ```

# Get out of Chroot - Reboot

1. Exit the chrooted environment: `exit` and unmount all mounted partitions:

   ```
   # exit
   # umount -lv /mnt/gentoo/home
   # umount -lv /mnt/gentoo/boot{/efi,}
   # umount -lv /mnt/gentoo/dev{/shm,/pts,}
   # umount -lv /mnt/gentoo{/proc,/sys,}
   # reboot
   ```

   When rebooting, if the LiveDVD usb stick is still plugged onto the computer, the chainload to Ubuntu grub does not work. You find _error: disk hd0,gpt2 not found_. This is because Grub treats the USB stick as _hd0_ while the hard disk as _hd1_. You can unplug the USB, and CTRL+ALT+DEL. Another way is to edit the Grub menu from _hd0_ to _hd1_, then press F10 to boot.

# Finalizing

1. User

   The very first thing after getis to create a regular user account:

   ```
   # useradd -G wheel,video -m myUser
   # passwd myUser
   ```

   Compared to *useradd*, *adduser* has a nice interactive mode.
2. Check wireless network.

   If fail to connect,

   ```bash
   # emerge -avt linux-firmware
   ```

   Remember we have fetched the package sources.

# System upate

```bash
# eix-sync
# emerge -avtuDN --with-bdeps=y @world
# dispatch-conf (opt)
# revdep-rebuild -pv (opt)
# emerge -avt @preserved-rebuild
# emerge -av --depclean
```

1. For *>=portageq-2.2.16*, *emaint sync* replaces original *emerge --sync* to sync Portage tree.
2. *--with-bdeps=y* will calculate build time dependencies for updates. Append this argument occasionally.
3. If configuration needs updated, there will be a corresponding *._cfg\** file.
4. As a tool of *gentoolkit*, *revdep-rebuild* is Gentoo's Reverse Dependency rebuilder. It will scan the installed ebuilds to find broken packages as a result of an upgrade of their dependencies. Those packages will be re-merged. However, it can also happen that a given package does not work with the currently upgraded dependencies, in which case you should upgrade the broken package to a more recent version. *revdep-rebuild* will pass flags to emerge which lets you use the --pretend flag to see what is going to be emerged again before going any further. 
5. `emerge --info \| grep FEATURES` prints *preserve-libs* FEATURE is enabledby default. It tells Portage to preserve libraries when *soname*'s change during upgrade or downgrade, only as necessary to satisfy shared library dependencies of installed consumers/packages. Preserved libraries are automatically removed when there are no remaining consumers, which occurs when consumer packages are rebuilt or uninstalled. Ideally, rebuilds are triggered automatically during updates, in order to satisfy slot-operator dependencies.

   However, before emerge exits during installing updates, if there are remaining preserved libraries because slot-operator dependencies have not been used to trigger automatic rebuilds, then emerge will display a message:

   ```
   !!! existing preserved libs:
   >>> package: sys-libs/libfoo-1
    * - /lib/libfoo.so.1
    *      used by /usr/bin/bar (app-foo/bar-1)
   Use emerge @preserved-rebuild to rebuild packages using these libraries
   ```
6. Clean packages out of dependency tree.

   Cleans the system by removing packages that are  not  associated with  explicitly merged packages. Depclean works by creating the full dependency tree from the @world set, then comparing it to installed packages. Packages installed, but not part of the dependency tree, will be uninstalled by depclean.

   *depclean* is the last stage of system update.

# Chroot - a complete routine

Boot with LiveDVD, then

```bash
# swapon /dev/sda7
# mount /dev/sda12 /mnt/gentoo
# mount /dev/sda10 /mnt/gentoo/boot
# mount /dev/sda2 /mnt/gentoo/boot/efi
# cp -L /etc/resolv.conf /mnt/gentoo/etc
# mount -t proc proc /mnt/gentoo/proc
# mount --rbind /sys /mnt/gentoo/sys
# mount --rbind /dev /mnt/gentoo/dev 
# mount --make-rslave /mnt/gentoo/sys 
# mount --make-rslave /mnt/gentoo/dev
# chroot /mnt/gentoo /bin/bash
# source /etc/profile && export PS1="(chroot) $PS1"
```

# System notes

1. Use *ffmpeg* instead of *libav* including the USEs.
2. Prefer *openssl/ssl* USE (*dev-libs/openssl*) to *gnutls* USE (*net-libs/gnutls*) for SSL/TLS secure connection. Please Google *openssl vs gnutls*. Gnutls seems to suffer library bug.
   1. *dev-libs/openssl*: full-strength general purpose cryptography library (including SSL and TLS).
   2. *net-libs/gnutls*: a TLS 1.2 and SSL 3.0 implementation for the GNU project.
3. Vaapi

   *ffmpeg*, *mesa*, *mpv*, and *firefox* enable *vaapi* USE.
4. Globally USEs:

   ```
   USE="${CPU_FLAGS_X86} -bindist -qt4 -qt5 -libav vaapi"
   ```
   
5. Try to to *--with-bdeps=y* occasionally on system update.

   Updates fundamental dependencies like *git* and *ffmpeg*.
6. Try *--oneshot -1* when re-merging dependencies.

   This option should only be used for packages that are reachable from the @world package set (those that would NOT be removed by --depclean).
7. Use UUID to identify a partition instead of */dev/sdaxy*.
8. Python

   Since python-exec-2.3 / eselect-python-20160207, the preferred method of altering Python configuration is to use the *edit* mode that opens *python-exec.conf* in an editor. The previous *set* mode is deprecated.

   ```bash
   ~ # eselect python list/edit`.
   ```

   For example, remove Python 3.4 after upgrading to 3.5.
1. [Perl update](https://wiki.gentoo.org/wiki/Perl)

   The official way of upgrading Perl, e.g. from Perl 5.22 to Perl 5.24, is upgrading your entire *world* set, and upgrading Perl with it.

   ```bash
   # emerge -avtuDN --with-bdeps=y --backtrack 30 @world
   # perl-cleaner --all
   ```

   1. If 30 does not help, try 100 instead.
   2. Attention: both `--with-bdeps=y` and `--backtrack 30` are required.
2. [upgrading Gcc](https://wiki.gentoo.org/wiki/Upgrading_GCC)

   After upgrading from 4.9 to 5.4, I encountered [librime-1.2.9 bug](https://forums.gentoo.org/viewtopic-t-1049882-start-0.html).

   ```bash
   ~ # gcc-config --list-profiles
   ~ # gcc-config 2
   ~ # env-update && source /etc/profile
   ~ # emerge -av1 sys-devel/libtool
   ~ # gcc --version
   ~ # emerge -avc =sys-devel/gcc-4.9.4
   ~ # revdep-rebuild --library 'libstdc++.so.6' -- --exclude gcc (opt)
   ```

   The last comman triggers over 60 packages rebuild.
3. SysRq
   1. `Alt - SysRq - argument` simutaneously; ThinkPad Fn is not required.
   2. Or press the key one by one: `Alt`, `[Fn] - SysRq`, and *argument*.
   3. The detailed *arguments* can be found at [kernel sysrq](https://www.kernel.org/doc/html/latest/admin-guide/sysrq.html).

       I generally unraw(r), sync(s), umount(u), and/or reboot(b) when my system locks.

# [X Window System](https://en.wikipedia.org/wiki/X_Window_System)

1. X Window System, X11, and X (shortname) are used interchangebly.

   X originated at the Massachusetts Institute of Technology (MIT) in 1984. The protocol has been version 11 (hence "X11") since September 1987.
2. [Xorg](https://wiki.gentoo.org/wiki/Xorg/Configuration) Server.
   1. Mainly *x11-base/xorg-server* (depending on *x11-base/xorg-drivers* and *x11-libs/mesa*).
   2. *vaapi* USE is enabled to utilize hardware acceleration.
   3. The *i965* and *i915* drivers split at system application (*media-libs/mesa*) level. The kernel always enable *i915* instead.

   Avoid

   ```bash
   # echo XSESSION="Xfce4" > /etc/env.d/90xsession
   ```

   since *root* does not launch X. If really need, set in *~/.bash_profile* (detailed below).
3. Display Manager: GDM, LightDM, SDDM etc.

   Most display managers source */etc/xprofile*, *~/.xprofile* and */etc/X11/xinit/xinitrc.d/*.
4. xinit

   We will use *startx* (wrapper of *xinit* binary) to read *~/.xinitrc* (the default is */etc/X11/xinit/xinitrc* or */etc/xdg/xfce4/xinitrc*). One of the main functions of *~/.xinitrc* is to dictate which client (i.e. Xfce4) for the X Window System is invoked with *startx* or *xinit* programs on a per-user basis.
4. Window Manager: Awesome, OpenBox, Xfwm4 etc.
5. Desktop: [Xfce](https://wiki.gentoo.org/wiki/Xfce) and [Xfce/Guide](https://wiki.gentoo.org/wiki/Xfce/HOWTO), KDE, Gnome etc.
   1. Refer to [XFCE_PLUGINS](https://gitweb.gentoo.org/repo/gentoo.git/tree/profiles/desc/xfce_plugins.desc).
   2. Similar to *xfce-extra/xfce4-notifyd*, *x11-themes/gnome-icon-theme* can be explicitly emerged along with *xfce-base/xfce4-meta*. Details refer to Missing icons in *Xfce4 configuration* below.
   3. Add *-qt4 -qt5* to *make.conf*.
6. It's essential to outline the diagram of X system:
   1. login shell -> /etc/profile -> .bash_profile -> .bashrc -> startx/xinit
   2. startx is a front wrapper to xinit program. startx/xinit launch X server (Xorg/X) and X client (Xfce4).
   3. Command: startx/xinit <X client> <X client arguments> -- <X server> <X server arguments>
   4. Without command line options, startx try to parse .xinitrc and .xserver respectively.
   5. Put environment variables for GUI applications into *~/.xinitrc* while for terminal into *~/.bashrc* or *~/.bash_profile*.

# X Configuration

1. Per-user [Theming](/2016/11/23/theme/) locations:
   1. *\*.desktop* file should be placed in *~/.local/share/applications/*.
   2. Themes are placed in *~/.local/share/themes/*.
   3. Icons are placed in *~/.local/share/icons/*.
2. *consolekit* is emerged as dependency during X installation. But *Shutdown*, *Suspend* etc. are greyed after getting into Xfce4.

   ```bash
   # echo "sys-auth/consolekit pm-utils" >> /etc/portage/package.use/consolekit
   # emerge -avt consolekit
   # rc-update add consolekit default
   ```

3. [Xinit](https://wiki.archlinux.org/index.php/Xinitrc) (Dispaly Manager)

   The default OpenRC Xinit conguration is located under */etc/X11/xinit*. Customization:

   First get a copy of the default. The reason of doing this (instead of creating one from scratch) is to preserve some desired default behaviour in the original file, such as sourcing shell scripts from */etc/X11/xinit/xinitrc.d*.
   
   ```bash
   # cp /etc/X11/xinit/xinitrc ~/.xinitrc
   ```

   The default Xinitrc examines Xresources and Xmodmap and prepares Consolekit and Dbus. The last section either tries *twm* and *xterm* or lanuches XSESSION. We can change *exec $command* to *echo $command* and test the script within virtual terminal (console). It looks like:

   >ck-launch-session dbus-launch --sh-syntax --exit-with-session /etc/X11/Sessions/Xfce4

   The default is enough to launch Xfce4 desktop. However we might require extra stuff like:

   ```
   export XMODIFIERS="@im=fcitx"
   export QT_IM_MODULE="fcitx"
   export GTK_IM_MODULE="fcitx"

   # Refer to /etc/X11/xinit/xinitrc
   # or replace 'startxfce4' with 'xfce4-session':
   #exec ck-launch-session dbus-launch --sh-syntax --exit-with-session xfce4-session
   exec $command
   ```

4. *startx*

   ```bash
   # startx -- vt7 -nolisten tcp
   ```

   Arguments after two bashes are passed to Xorg server.

   1. The *xserverrc* file is a shell script responsible for starting up the Xorg server. Both *startx* and *xinit* execute *~/.xserverrc* if it exists, *startx* will use default */etc/X11/xinit/xserverrc* otherwise.
   2. Attention, *-nolisten tcp* is to disallow TCP connection to X server.
   3. *vt7* must be appended, otherwise switches between X and virtual terminal would freeze the whole X seesion to death. Refer to [Intel HD3000 Tearing/Corruption/Glitch](/2016/09/10/intel-graphics/).

   The default */etc/X11/xinit/xserverrc* fails to set the correct virtual terminal to start X server. It depends on `$XDG_VTNR` which is not available on OpenRC [but Systemd](https://www.freedesktop.org/wiki/Software/systemd/writing-display-managers/).

   An alternative is:

   ```bash
   # cp /etc/X11/xinit/xserverrc ~/.xserverrc
   ```
   
   Modify a little bit:

   ```
   #!/bin/sh
   if [ -z "$XDG_VTNR" ]; then
     exec /usr/bin/X -nolisten tcp "$@" vt7
   else
     exec /usr/bin/X -nolisten tcp "$@" vt$XDG_VTNR
   fi
   ```

   If a Display Manager is used, those arguments would be handled correctly without user concern.

3. Automatic Xfce4 on login

   Edit *~/.bash_profile*:

   ```
   # /etc/skel/.bash_profile

   # This file is sourced by bash for login shells.  The following line
   # runs your .bashrc and is recommended by the bash info pages.
   if [[ -f ~/.bashrc ]] ; then
           . ~/.bashrc
   fi

   # If ~/.xinitrc does not exist, startx takes it as a fallback
   XSESSION="Xfce4"

   if shopt -q login_shell; then
           [[ -t 0 && $(tty) == /dev/tty1 && "$USER" == "username" && ! $DISPLAY ]] && exec startx -- vt7 -nolisten tcp
   fi
   ```

   1. If *~/.xserverrc* exists, *vt7 -nolisten tcp* part can be removed.
   2. If you would like to remain logged in when the X session ends, remove *exec*.
4. Clear Xfce configuration

   ```bash
   # rm -r ~/.cache/sessions
   # rm -r ~/.config/xfce*
   # rm -r ~/.config/Thunar
   ```

5. Hide unmounted partitions from user interface (opt)

   When you get into the Xfce4 desktop, you may found many unnecessary disk icons on the desktop or thunar sidebar. It's annoying. Use *udev/udisks* utility.

   Edit */etc/udev/rules.d/80-hide-disks.rules*:

   ```
   ENV{ID_FS_UUID}=="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXXX", ENV{UDISKS_IGNORE}="1"
   or
   KERNEL=="sdaXY", ENV{UDISKS_IGNORE}="1"
   ```

   To load the new rules without reboot:

   ```bash
   # udevadm control --reload
   # udevadm trigger
   ```

   1. XY is the disk partition number you would like to hide. As noted in the reference below, `UDISKS_PRESENTATION_HIDE` is deprecated and replaced by `UDISKS_IGNORE`.
   2. Refer to */lib64/udev/rules.d/80-udisks2.rules*, read section:

      >Devices which should not be display in the user interface
   3. Refer to [udev 99-hide-disks.rules is no longer working](https://superuser.com/q/695791) and [udisks on archwiki](https://wiki.archlinux.org/index.php/Udisks).
6. Touchpad

   If Touchpad does not support *tap click*. As long as *x11-drivers/xf86-input-synaptics* is installed, a default configuration is located at */usr/share/X11/xorg.conf.d/50-synaptics.conf*. Copy it to */etc/X11/xorg.conf.d/* and edit:

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
           Option "MaxTapTime" "125"
   EndSection
   ```

   You can also set a temporary config at command line. Check command line *synclient*.

# New *plug-in sync system* - Layman/Overlay

1. Layman adds, removes, syncs etc. Overlays on system.
2. With the help of Portage new *plug-in sync system*, Overlays can be easily synced by Portage *emaint*. Therefore, Layman is purely used to add/remove Overlays.
3. From *2.3.0* onwards, Layman supports the new *plug-in sync system* naturally. But current stable version is *2.0.0*. Ether accept *2.3.0* or manually write new *plug-in sync* configurations for newly added Overlays.

   ```bash
   # emerge -avt app-portage/layman
   ```

   By default, the *subversion* USE is not enabled, which support *subversion*-based Overlays.

   After emerging *<layman-2.3.0*, don't do any configuration (i.e. *PORTDIR*, *PORTDIR_OVERLAY*) as told on Wiki since we will manually add new *plug-in sync system* configuration.
4. Add/remove an Overlay:

   ```bash
   # layman -a gentoo-zh
   # layman -d gentoo-zh
   ```

5. Manual Overlay configuration (*<layman-2.3.0*)

   Add */etc/portage/repos.conf/gentoo-zh.conf*:

   ```
   [gentoo-zh]
   location = /var/lib/layman/gentoo-zh
   sync-type = git
   sync-uri = git://github.com/microcai/gentoo-zh.git
   auto-sync = yes
   ```

   We choose *git* instead of traditional *rsync*. If needed, official Portage tree *gentoo.conf* can be changed to *git* type as well.
6. Portage tree switch to *git* sync

   ```bash
   # mv /usr/portage/distfiles ~/
   # mv /usr/portage/packages ~/
   # rm -rf /usr/portage/*
   # emaint sync -r gentoo
   # mv ~/distfiles /usr/portage
   # mv ~/packages /usr/portage
   ```

   We delete the Portage tree and *emaint sync -r* will *git clone* locally. Refer to [Portage git sync](http://blog.yjl.im/2015/05/going-gentoo-portage-git-sync.html).
5. (opt) Sync

   ```bash
   # eix-sync, or
   # emaint sync
   ```

# Applications

1. Xfce4 goodies

   xfce4-power-manager; xfce-extra/xfce4-pulseaudio-plugin and media-sound/pavucontrol; xfce4-screenshooter.

   1. Go to Applications > Settings > Keyboard, Application Shortcuts. Add the *xfce4-screenshooter -r* command to use the PrtSc key.
2. Recommendations

   apg, guake, wgetpaste, weechat, wps-office, evince, [TeXLive](/2015/08/29/texlive-gentoo/), remmina/freerdp.

   1. [wps math formula fonts](https://github.com/IamDH4/ttf-wps-fonts) and [fontconfig](/2015/04/13/fontconfig/). Those fonts are essential to display formulas. However, WPS-linux does not have built-in formula creation function due to copyright.

3. Sound - ALSA/[PulseAudio](https://wiki.gentoo.org/wiki/PulseAudio)
   1. PulseAudio depends on (both kernel and package parts) of ALSA - they coexist instead of excluding each other. It serves as a proxy to sound applications using existing kernel sound components like ALSA or OSS.
   2. ALSA includes both kernel sound card drivers and userspace package *media-libs/alsa-lib*, while PulseAudio builds on top of ALSA kernel part while offering userspace package *media-sound/pulseaudio*.
   3. PulseAudio is configured to automatically detect all sound cards and manage them. It takes control of all detected ALSA devices and redirect all audio streams to itself, making the PulseAudio daemon the central configuration point. 

   Check if *alsa-lib* and *alsa-utils* are installed or not. By default, the `alsa` USE flag is enabled in profile, so these packages will be emerged by default.

   ```bash
   # speaker-test -t wav -c 2
   ```

   Enable global *pulseaudio* USE:

   ```bash
   # /etc/portage/make.conf
   ~ # euse -E pulseaudio
   # remove all users from *audio* group
   ~ # gpasswd -d username audio
   ~ # emerge -avtuDN --with-bdeps=y @world
   ```

   1. Enabling *pulseaudio* USE flag will pull in *media-sound/pulseaudio* automatically.
   2. If you plan running *pulseaudio* in the [system-wide mode](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/PerfectSetup/), then the special user *pulse* should still be in the *audio* group in order to have access to the sound card.

   GStreamer support:

   ```bash
   ~ $ gconftool-2 -t string --set /system/gstreamer/0.10/default/audiosink pulsesink
   ~ $ gconftool-2 -t string --set /system/gstreamer/0.10/default/audiosrc pulsesrc
   ```

   >From what we've done, you know *enabling pulseaudio* is quite easy as no kernel options required like ALSA.

4. Firefox

   1. To make use of *vaapi*, make sure *ffmpeg* and *hwaccel* USEs are enabled. BTW, *ffmpeg* should enable *vaapi* USE too.
   2. For Media Source Extensions, turn on *media.fragmented-mp4.exposed*, *media.fragmented-mp4.ffmpeg.enabled*, *media.mediasource.enabled*, *media.mediasource.mp4.enabled* and *media.mediasource.webm.enabled* in *about:config*, while disabling *media.fragmented-mp4.use-blank-decoder*.
   3. Add FoxyProxy Standard, uBlock Origin, NoScript (and/or RefControl), HTTPS Everywhere, DISCONNECT, Open With etc. add-ons. Remove unecessary default whitelist of NoScript plugin.
   4. *privacy.trackingprotection.enabled*, *Network.proxy.socks_remote_dns* to True.
   5. *network.IDN_show_punycode* to be True.
   6. *browser.cache.check_doc_frequency* set to [0/1/2/3](https://superuser.com/a/411826); Ctrl + Shift + I, Gear, Advanced Settings, Disable Cache (when toolbox is open).
   5. [Harden Firefox security](https://vikingvpn.com/cybersecurity-wiki/browser-security/guide-hardening-mozilla-firefox-for-privacy-and-security) and [disable useragent](http://www.howtogeek.com/113439/how-to-change-your-browsers-user-agent-without-installing-any-extensions/). Like *general.useragent.vendor/override*.
   8. [fingerprint test](https://panopticlick.eff.org).

5. [Fcitx](https://wiki.gentoo.org/wiki/Fcitx)

   ```bash
   # echo "app-i18n/fcitx gtk gtk3 -table" >> /etc/portage/package.use/fcitx
   # emerge -av fcitx fcitx-rime fcitx-configtool
   ```
   
   *table* USE will draw several built-in input methods (i.e. Wubi) which fail to meet my requirement. *rime* will be installed instead.
   
   Make sure the following code resides before *exec* of *~/.xinitrc*.

   ```
   export GTK_IM_MODULE=fcitx
   export QT_IM_MODULE=xim
   export XMODIFIERS=@im=fcitx
   ```

   They should be *ahead* of *exec startxfce4* or *xfce4-session*. Commands after *exec* won't be executed! Refer to [xfce4安装fcitx不能激活！很简单的一个原因！](https://bbs.archlinuxcn.org/viewtopic.php?pid=13921).
6. FFmpeg

   ```
   # echo "media-video/ffmpeg vaapi librtmp openssl vpx" >> /etc/portage/package.use/ffmpeg
   # emerge -avt ffmpeg
   ```

   1. *ffmpeg* is emerged by some other packages, one of which might be Firefox or Mpv.
   4. Add *-libav* to *make.conf* in favor of system-wide *ffmpeg*.
   2. *v4l* and *libv4l*USE for the webcamera.
   3. *vaapi* USE for hardware decoding.
   5. *librtmp* USE to replace the native RTMP implementation (bad performance).
   6. *network* (default) and *openssl* to support HTTP/HTTPS streaming.

7. Mpv

   ```bash
   # echo "media-video/mpv vaapi" >> /etc/portage/package.use/mpv
   # echo "=media-video/mpv-9999 **" >> /etc/portage/package.keywords/mpv
   # emerge -avt mpv
   ```

   1. You are recommended to accpet *mpv* live build.
   2. Rely on *ffmpeg* and *youtube-dl*.
   3. Enable *vaapi* USE for *mpv* and *ffmpeg to utilize hardware acceleration.
   4. Re-merge *mpv* if [*ffmpeg* is upgraded](https://wiki.gentoo.org/wiki/Mpv#Broken_playback.2Fcrashes_after_updating_FFmpeg.2FLibav.2Flibass.2Fetc.)
   5. The default *lua* USE flag of *mpv* brings along a *youtube-dl* hook script. *mpv url* revokes *youtube-dl* to stream video playback.

      And make sure *ffmpeg* is installed with *network* USE flag. To stream HTTPS url, enable either *openssl* or *gnutls* USE of *ffmpeg*. Ether *openssl* is fine and you don't need both.

   6. *youtube-dl* is suggested to install under Python3 virtual environment by *pyvenv-3.4* thus receving immediate updates. If *youtube-dl* does not stream Chinese url, try *mpv $(youtube-dl --get-url http://v.youku.com/v_show/id_XMTU2Mjc4MTc4NA==.html)*.

      After installing *youtube-dl* in Python3 virtual environment:

      ```bash
      $ ln -sf /absolute/path/to/py3venv/youtube-dl ~/.config/mpv/youtue-dl
      ```

      If *youtube-dl* sucks, could try *you-get* (for Chinese urls) and *livestreamer* (for live broadcast). If the url is blocked, we should first enable *proxychains* or *torify*.

   7. When streaming, *mpv* might throtle/stuck a lot waiting for cache. This is due to ISP throtling; video provider streaming limit; network bandwidth etc.
      1. Set bigger *--cache* value; press SPACE to pause for fechting more cache.
      2. Run *youtube-dl url* and *mpv partial-downloaded-file* separately (say in two terminals). If does not play the partial file (HASH video and audio are usually downloaded separately), try *-f best* argument which tell to download audio and video into a single file. If downloading audio and video separately, there would be no audio or video when playing.
   8. We can use Firefox addons *Open With* to revoke *mpv*.

      ```
      /usr/bin/proxychains /usr/bin/mpv  --
      or
      ~/workspace/virtualenv3/bin/you-get -p /usr/bin/mpv --
      ```

      The last two hypens means arguments afterwards are input files/urls. Since using the *Open With* addon, save more CPU and memory than Firfox.
   9. Details on [*youtube-dl*](https://github.com/rg3/youtube-dl) and [*you-get*](https://github.com/soimort/you-get), refer to their Github pages.
   9. Some videos with *aac* audio track won't show video (only audio). `-v --no-config` gives:

      >[lavf] Edit lists are not correctly supported (FFmpeg issue).

      Before *ffmpeg* fixes the *edit list* parsing, add `--demuxer-lavf-o=ignore_editlist=1` option.
8. Emacs
   1. Use *athena Xaw3d -gtk -gtk3 -motif*  USEs to replace GTK toolkit if multiple monitors are used. Refer to *daemon mode* bug [reddit](https://www.reddit.com/r/emacs/comments/2ans0z/have_you_encountered_that_gtk_bug_in_daemon_mode/?ref=share&ref_source=link) and [wiki](https://wiki.gentoo.org/wiki/GNU_Emacs).
   2. *xft* for Fontconfig. *libxml2* support *shr* enables *eww* HTML viewer. *gnutls/ssl* supports Gnus IMAP connection.
   3. Chinese input with Fcitx. Change the Fcitx input method trigger to `WIN+I` instead of `CTRL+SPACE`.
   4. (opt) If X frame does not input Chinese, emerge two fonts: *font-adobe-100dpi* and *font-adobe-75dpi*. You can read post-emerge message:

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

   5. *emacs-daemon*

      ```bash
      # emerge -avt emacs-daemon
      # cd /etc/init.d/
      # ln -sv emacs emacs.usernmae
      # rc-update add emacs.username default, (do NOT add */etc/init.d/emacs* to default runlevel directly)
      # rc-service emacs.username start
      ```

   6. XIM

      Since *emacs-daemon* is launched before (OpenRC init) setting variable (XMODIFIERS="@im=fcitx" by *startx* in *~/.xinitrc*), *emacsclient -c* cannot find external Fcitx's XIM server. Possible solutions:

      ```bash
      #/etc/init.d/emacs.username
      export SHELL EMACS EMACS_TIMEOUT EMACS_DEBUG
      export XMODIFIERS=@im=fcitx
      ```

      We set the variable in init *start()* function. Alternatively, set XMODIFIERS through `--env` argument of *start-stop-daemon*.

      ```bash
      start-stop-daemon --start \
        --user "${USER}" --pidfile "${PIDFILE}" --chdir "${home}" \
        --env XMODIFIERS="@im=fcitx" --exec "${EMACS_START}" \
	-- ${EMACS_OPTS}
      ```

      However, direct modification init script is discouraged (suppose you are on a multiple user accounts system). *emacs-daemon* has a wrraper script (*/usr/libexec/emacs/emacs-wrapper.sh*) to start the daemon *with a login shell*. We can set XMODIFIERS in bash startup script (*~/.bashrc* or *~/.bash_profile*).

      A more elegant way is to create a new copy of */etc/conf.d/emacs* and do update there:

      ```bash
      # pushd /etc/conf.d
      # cp emacs emacs.username
      # nano -w emacs.username
      export XMODIFIERS="@im=fcitx"
      ```

      Don't worry! *openrc-run* looks for both */etc/conf.d/emacs* and */etc/conf.d/emacs.username*.
      
      Emacs uses XIM to support external input method instead of GTK/QT toolkit. Details on XMODIFIERS/XIM, refer to Fcitx's official page. Read more at [fcitx not work on *emacsclient --create-frame*](https://stackoverflow.com/a/34852916) and [app-emacs/emacs-daemon should allow for setting up a proper environment](https://bugs.gentoo.org/show_bug.cgi?id=246460).

9. Nano

   After installing Emacs, *nano* will be removed on next *emerge -avc* (check package *virtual/editor*). To keep *nano*:

    ```bash
    # emerge -avt --noreplace nano
    ```

   Turn on a few Nano options in */etc/nanorc*: *set autoindent*, *set backup*, *set tabsize 4* etc. If need to totally disable an option, use *unset <option>*.
1. Archive

   ```bash
   # emerge -av file-roller
   # emerge -av thunar-archive-plugin
   ```

   1. *file-roller* is a front-end GTK interface - Archive Manager, which needs background Archiver support - *zip/unzip*, *bzip2*, *tar*, *7zip*, *unrar* etc.
   2. *thunar-archive-plugin* is a Thunar plugin (right-click menu). If *thunar-archive-plugin* cannot find a suitable archive manager, check [thunar archive plugin cannot integrate with file-roller](https://forums.gentoo.org/viewtopic-t-1006838.html?sid=bce8eeef9eab8d916c59b01cef493bb4) and [doesn't work anymore with recent file-roller](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=746504).
   3. By default, *unzip* cannot extract arhives containing no-ASCII file names (i.e. archives created on Chinese Windows). Enable *natspec* USE that brings in `-O` command line option. Check `-O` by `-h` as it's not present in man page. Attention, *natpsec* requires `LC_CTYPE` be *zh_CN*.
   4. (opt) *p7zip* has a higher data compression ratio.

      *p7zip* brings 3 archive binaries, namely *7z*, *7za* and *7zr*. Only *7zr* can extract non-ascii *zip* files. However *file-roller* will take *7z* and *7za* primarily. Rename */usr/bin/7z* and */usr/bin/7za* to something else.
2. [NetEase-MusicBox](https://github.com/darknessomi/musicbox)

   ```bash
   # emerge -avt media-sound/mpg123
   $ source ~/opt/pyvenv3/bin/activate
   $ pip(3) install NetEase-MusicBox
   $ pip(3) install lxml
   ```

   If encountered error during installing NetEase-MusicBox,

   ```
   running build_ext
   running build_configure
   checking for gcc... gcc
   checking whether the C compiler works... yes
   checking for C compiler default output file name... a.out
   checking for suffix of executables...
   checking whether we are cross compiling... configure: error: in `/tmp/pip-build-y5q1g0ib/pycrypto':
   configure: error: cannot run C compiled programs.
   ```

   The reason is that */tmp* is mounted as *noexec*. To temporarily solve it,

   ```bash
   ~ $ TMPDIR=~/tmp pip(3) install NetEase-MusicBox
   ```

   Read more on [Won't install python 2.7.4 if /tmp is mounted noexec](https://github.com/pyenv/pyenv/issues/13) and [pip install pycrypto fails on centos 6.4](https://bugs.launchpad.net/pycrypto/+bug/1294670/comments/7).

# Sphere

1. Blacklist/install modules

   Prevent rarely used modules from loading on boot. Always refer to command line tool *lsmod*, *modinfo* if unsure. Create */etc/modprobe.d/blacklist.conf* and put unwanted modules there.

   ```
   # /etc/modprobe.d/blacklist.conf
   blacklist btusb
   blacklist btbcm
   blacklist btintel
   blacklist bluetooth
   ```

   We can replace *blacklist* with *install ... true/false* which makes a difference. A blacklisted module will be loaded if another non-blacklisted module depends on it, or if it is loaded manually. For example, *bluetooth* is blacklisted. If we manullay *modprobe btusb*. *bluetooth* module will load as a dependency. Hence, it's necessary to remove related *init service* (if exist) from runlevels. Otherwise *blacklisted* modules were loaded.

   To ensure the modules are never inserted, even if they are needed by other modules or manually *modprobe*, use *install*. To install a module as *true* or *false* does not make much difference. *false* just files an extra error message while *true* silence it. Refer to [Modprobe is better disabled by using */bin/true* (not */bin/false*](https://github.com/OpenSCAP/scap-security-guide/issues/539).

   Refer to [arch wiki blacklist](https://wiki.archlinux.org/index.php/Kernel_modules#Blacklisting); [changes to module blacklisting](https://www.archlinux.org/news/changes-to-module-blacklisting/).

2. Intel Microcde (opt)

   ```
   # emerge -av microcode-ctl
   # rc-update add microcode_ctl boot
   ```

   Refer to [intel microcode](https://wiki.gentoo.org/wiki/Intel_microcode). Updates to CPU microcode have to be re-applied each time the computer is booted, because the memory updated is volatile (despite the term *firmware* also being used for microcode).

   According to the reference, you should run `dmesg | grep -i microcode` to check whether CPU microcode is updated or not. If not, I think there is not need to add *microcode-ctl* to boot level until *microcode-ctl* package is updated.

   It's recommended to update CPU microcode through BIOS update. We can do this under Windows.
3. (opt) swapfile

   ```bash
   # dd if=/dev/zero of=/mnt/1GB-swapfile bs=1M count=1024
   # chmod 600 /mnt/1GB-swapfile
   # mkswap /mnt/1GB-swapfile
   # swapon /mnt/1GB-swapfile
   # swapon -s
   # ect /etc/fstab
   /mnt/1GB-swapfile none swap defaults 0 0
   ```

   >Since OpenRC-0.22, please enable *rc_need="localmount"* in */etc/conf.d/swap*.

   1. Use *dd* to create a file occupying continuing disk space instead of a *sparse file*. Refer to [swap partition vs file for performance?](https://serverfault.com/a/25708).
   2. Refer to [swap file creation](https://wiki.archlinux.org/index.php/Swap#Swap_file_creation).
