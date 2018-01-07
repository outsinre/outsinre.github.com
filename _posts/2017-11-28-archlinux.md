---
layout: post
title: Arch Linux
---

>Arch Linux has dropped support for *i686* platforms. Only *x86_64* now!

# [Installation guide](https://wiki.archlinux.org/index.php/Installation_guide)

Once booting into the live CD (*archlinux-2017.11.01-x86_64.iso*), check network connection and rectify system clock:

```bash
root@archiso ~ # ping www.archlinux.org
root@archiso ~ # timedatectl set-ntp true; timedatectl status
```

## Scheme

1. Grub, BIOS/GPT partitioning.
2. Single *root* partition plus extra *swap file*.

### [Partitioning](https://wiki.archlinux.org/index.php/Partitioning)

Firstly, to identify disk devices:

```bash
root@archiso ~ # fdisk -l
root@archiso ~ # lsblk -f
root@archiso ~ # blkid
root@archiso ~ # findmnt
```

For Grub, BIOS/GPT scheme, we must create the [BIOS boot partition](https://wiki.archlinux.org/index.php/BIOS_boot_partition) to hold Grub *core.img*. Around 1 MiB (2048 sectors) is enough. It can be in any position order (partition number) but has to be on the first 2 TiB of the disk. This partition should be flagged as *bios_grub* for *parted*, *ef02* for *gdisk*, or select *BIOS boot* and partition type *4* for *fdisk*.

Grub, BISO/MBR scheme uses post-MBR gap (after the 512B MBR and before the first partition) to store *core.img*. Usually this gap is 31 KiB. For complex modules in *core.img*, this space is limited. That can be resolved by pushing forward starting sector of the first parititon. For instance, partitioning tool can align at MiB boundaries, thus leaving enough post-MBR gap.

>Attention, the later scheme eliminates the bother to create BIOS boot partition.

```
root@archiso ~ # parted -a optimal /dev/sda
(parted) help
(parted) mktable gpt
(parted) unit s
(parted) print free
(parted) mkpart primary 0% 2047s
(parted) set 1 bios_grub on
(parted) mkpart primary 2048s 100%
(parted) print free
(parted) quit
```

1. Tell *parted* to optimally align parititon boundaries.
2. By default, GPT's free sector starts at 34s (equally *0%*). Make BIOS boot partition be the very first parition which starts at *34s* spanning to *2047s*. *parted* reminds:

   >Warning: The resulting partition is not properly aligned for best performance. Ignore/Cancel?

   Choose *Ignore* as this partition will not be regularly accessed. Performance issues can be disregarded though this is out of GPT alignment specifications.
3. The rest space is assigned to *root* starting at *2048s*.

   On a device with 512B sectors, *parted* wants to align at multiple 1 MiB (i.e. 2048s, 4096s, 6144s). This tells the reason that previous BIOS boot partition ends at 2047s.

#### Swap

There is not any other separate partitions except that a *swap file* will be created. Swap partition bears NO advantages over swap file while could be shared among different systems. On the other hand, we can easily resize and plug/unplug a swap file on-the-fly.

Leave the swap file part after booting into new system.

### File system

There is not need to create file system for BIOS boot parititon. Hence, just create an *ext4* on the single *root* parition:

```bash
root@archiso ~ # mkfs.ext4 /dev/sda2
```

### Mount

```bash
root@archiso ~ # mount /dev/sda2 /mnt
```

As there is only the *root* parition, we would not create other mount points (i.e. *boot*) under */mnt*.

## Mirrors

Edit */etc/pacman.d/mirrorlist* and place geographically closest mirrors on top. Optionally comment out all the other mirrors.

## base packages

```bash
root@archiso ~ # pacstrap /mnt base
```

This will install all packages from *base* group to *root* partition. Around 200 MiB packages will be downloaded and 700 MiB disk space consumed. Ignore the warning on *locale* failure that will be handled after *chroot*.

## fstab

```bash
root@archiso ~ # genfstab -U /mnt >> /mnt/etc/fstab
root@archiso ~ # cat /mnt/etc/fstab
```

`-U` and `-L` use UUID and label respectively. Obviously, there is only one record generated.

## Chroot

```bash
root@archiso ~ # arch-chroot /mnt
```

Pretty simple step.

## Locale

Mainly, we will modify two files */etc/locale.gen* and */etc/locale.conf*.

```bash
# Uncomment the following lines from /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GB18030 GB18030
zh_TW.UTF-8 UTF-8
#
[root@archiso / #] locale-gen
[root@archiso / #] echo LANG=en_US.UTF-8 > /etc/locale.conf
```

Here we set system locale to *en_US.UTF-8* (better for log trace). For per-user locale, leave it after new user account creation.

## Hostname

```bash
# Create /etc/hostname
[root@archiso / #] echo "myhostname" > /etc/hostname
# add a line to /etc/hosts accordingly
127.0.1.1	myhostname.localdomain	myhostname
```

## *root* password

```bash
[root@archiso / #] passwd
```

## Boot loader - Grub

```bash
[root@archiso / #] pacman -S grub intel-ucode
[root@archiso / #] grub-install --target=i386-pc /dev/sda
[root@archiso / #] grub-mkconfig -o /boot/grub/grub.cfg
```

1. Although we install *x86_64* Arch Linux, the `--target` should be *i386-pc* due to BIOS booting scheme.
2. For intel CPU, *intel-ucode* package is installed to update processor *microcode*.

## Reboot

```bash
[root@archiso / #] exit
root@archiso ~ # umount -R /mnt
root@archiso ~ # reboot
```

We'd better unmount all partitions under */mnt* to determine busy partitions and diagnose with *fuser*.

# Post-installation tricks

1. *dhcpcd* is not *enabled* by default.

   ```bash
   [root@host ~]# systemctl enable dhcpcd
   [root@host ~]# systemctl start dhcpcd
   ```

2. 1 GiB Swap file

   ```bash
   [root@host ~]# dd if=/dev/zero of=/swapfile bs=1M count=1024
   [root@host ~]# chmod 600 /swapfile
   [root@host ~]# mkswap /swapfile
   [root@host ~]# swapon /swapfile
   [root@host ~]# swapon --show
   [root@host ~]# free -h
   ```

   Finally, add an entry to */etc/fstab*:

   >/swapfile none swap defaults 0 0

   1. We must use the exact swap file path instead of UUID or Label.
   2. To cease manual operation bother, try *systemd-swap* that automates swap management.
3. New user account

   ```bash
   [root@host ~]# passwd -Sa (list system users)
   [root@host ~]# useradd -m test
   [root@host ~]# passwd test
   ```

4. Default EDITOR

   ```bash
   # ~/.bashrc
   export EDITOR=emt
   ```
   
# Xorg and awesome

```bash
[root@host ~]# pacman -S xorg-server (X server)
[root@host ~]# pacman -S xorg-xinit (startx/xinit)
[root@host ~]# pacman -S awesome (X client - WM)
[root@host ~]# awesome
# E: awesome: main:656: cannot open display (error 5)
```

1. We will use *startx* or *xinit* to launch X server and client awesome. awesome alone would fail as X server is not launched.
2. Video drivers. If this Arch Linux is VirtualBox guest, then install VirtualBox guest additions. Ohterwise:

```bash
[root@host ~]# lspci | grep -e VGA -e 3D (check video card)
[root@host ~]# pacman -Ss xf86-video (search for video drivers)
[root@host ~]# pacman -S xf86-video-intel (take Intel for example)
```

## Configuration

The configuration API varies often across awesome updates. So, repeat these configuration whenever something goes strange, or you want to modify the configuration.

```bash
[user@host ~]$ mkdir -p ~/.config/awesome
[user@host ~]$ cp /etc/xdg/awesome/rc.lua ~/.config/awesome/
```

### Autostart

Desktop (XFCE, KDE etc.) havs autostart *.desktop* files in */etc/xdg/autostart/* or *~/.config/autostart/*. awesome requires manual settings:

Here is an example of VBoxClient-all autostart with awesome. Create *~/.config/awesome/autostart.sh*:

```bash
#!/usr/bin/env bash

function run {
  if ! pgrep $1 ;
  then
    $@&
  fi
}

run VBoxClient-all
```

Make it executable:

```bash
[user@host ~]$ chmod +x ~/.config/awesome/autostart.sh
```

Check *autostart.sh*:

```bash
[user@host ~]$ ~/.config/awesome/autostart.sh
```

You can add any other programs to autostart by appending *run executable --arguments* to the end of *autostart.sh*.

Finally, add the following line to *~/.config/awesome/rc.lua*:

```lua
awful.spawn.with_shell("~/.config/awesome/autostart.sh")
```

## xinit

### xserver

The default */etc/X11/xinit/xserver* does not include `vt$XDG_VTNR` which let adversaries bypass screen lock by switching terminals (Ctrl+Alt+Fx). So create *~/.xserver* as:

```
#!/bin/sh

exec /usr/bin/Xorg -nolisten tcp "$@" vt${XDG_VTNR}
```

You can check `XDG_VNTR` variable afterwards.

### xinitrc

If *~/.xinitrc* is present in a user's home directory, startx and xinit execute it. Otherwise startx will run the default */etc/X11/xinit/xinitrc*. On the onther hand, xinit has its own defaults (check *man 1 xinit*).

xinitrc, by default, launches Twm, xorg-xclock and Xterm (assumen these packages are installed). We want awesome instead:

```bash
[user@host ~]$ cp /etc/X11/xinit/xinitrc ~/.xinitrc
```

Go to the end and replace the defaults with:

```
# ~/.xinitrc

exec awesome
```

1. Commands after *exec* won't be executed in *.xinitrc*. If any other commands are required, put them before *exec* line.
2. If you decide to write a custom *~/.xinitrc* file, then make sure existing *~/.Xresources* are loaded like the default does.

Finally, on terminal type: *startx*.

### Automatic startx

Add the following code to *~/.bash_profile*:

```bash
if shopt -q login_shell; then
        [[ -t 0 && "${XDG_VTNR}" -eq 1 && "$USER" == "username" && ! "$DISPLAY" ]] && exec startx 2>&1 | tee "$HOME"/.startx.log
fi
```

# [VirtualBox Guest Additions](https://www.virtualbox.org/manual/ch04.html#idm2096)

VirtualBox Guest Additions consists of device drivers and system applications that optimize the guest operating system for better performance and usability.

1. Some Linux distributions (i.e. Gentoo, Arch Linux) already come with all or part of the VirtualBox Guest Additions. On such Linux guests just install the corresponding package, for example:

   ```bash
   [root@host ~]# emerge -avt app-emulation/virtualbox-guest-additions (Gentoo)
   [root@host ~]# pacman -S virtualbox-guest-utils/virtualbox-guest-utils-nox (archlinux)
   ```

   This method is always preferred!
2. Alternatively, like Windows guest, we can mount the VBoxGuestAdditions.iso file and invoke the relevant installation script manually.

   Firstly, we should obtain the ISO from host (i.e. *vboxmanage storageattach*), from guest package repository (if available) or more simply from VirtualBox official website.

   ```bash
   [root@host ~]# lsblk -f
   [root@host ~]# mkdir -p /mnt/vbox
   [root@host ~]# mount -o loop /dev/sr1 /mnt/vbox
   [root@host ~]# ls /mnt/vbox
   [root@host ~]# sh ./VBoxLinuxAdditions.run
   ```

   This is an example of gettin VBoxLinuxAdditions.iso from host. To get ISO file from Arch Linux package repository:

   ```bash
   [root@host ~]# pacman -S virtualbox-guest-iso
   [root@host ~]# ls /usr/lib/virtualbox/additions/VBoxGuestAdditions.iso
   ```

3. Lastly but not least, guest additions on guest OS and VirtualBox application on host OS should have matching version, otherwise some guest addtions functionalities (i.e. shared clipboard) *may* stop working. Update of guest additions or VirtualBox application on one OS assumes the counterpart on the other OS.
4. Environment:
   1. Gentoo host: kernel-4.12.5, VirtualBox 5.1.26.
   2. Arch guest: kernel-4.13.12,  VirtualBox 5.2.2.

   Gentoo is relatively conservative on package rolling update compared to Arch Linux. There is a remarkable gap between VirtualBox packages and kernel versions. Latest Linux kernel version always expect newer Virutalbox packages. So installing VBoxGuestAdditions-5.1.26.iso from Gentoo host *may* break things.
5. In this post, I choose the one from Arch Linux repository:

   ```bash
   [root@host ~]# pacman -S virtualbox-guest-utils (X environment)
   # or
   [root@host ~]# pacman -S virtualbox-guest-utils-nox (no X)
   ```

6. Enable *vboxservice*

   ```bash
   [root@host ~]# systemctl enable vboxservice
   ```

   This service loads *vboxguest*, *vboxsf*, and *vboxvideo* kernel modules. It's also responsible for synchronizing system time with host.
7. VBoxClient

   VBoxClient (or the wrapper *VBoxClient-all*) is the core VirtualBox guest additions service. It manages clipboard, seamless window display, etc. Package *virtualbox-guest-utils* installs */etc/xdg/autostart/vboxclient.desktop* that launches VBoxClient-all on logon.

   VBoxClient-all script launches VBoxClient as:

   ```bash
   [user@host ~]$ VBoxClient --clipboard --draganddrop --seamless --display --checkhostversion
   ```

   Check Autostart section above on how to launch VBoxclient alongside with awesome.
8. Notice: you should guest system before VirtualBox guest additions take effect.

# Time

>VirtualBox guest OS relies on VirtualBox guest additions to synchronizes time with host. Hence leave this part after VirtualBox guest additions.

In an operating system, the time (clock) is determined by four parts: time value, time standard, time zone, and Daylight Saving Time (DST) if applicable. Especially, RTC is just a bare value on board without any other information attached. It's the time managment tools that control time standard, time zone and DST.

*timedatectl* is used to control system date and time while *hwclock* is for hwardware time (RTC). The use of *timedatectl* requires an active dbus. Therefore, it may not be possible to use this command under a *chroot* (i.e. during installation). In such cases, you can revert back to the *hwclock* command or wait for booting into the new system.

As stated before, dual boot with Windows should set Linux system to treat RTC as *localtime*. If this installation resides in virtual machine, then leave it default.

Check time:

```bash
[root@host ~ #] hwclock/timedatectl -h
[root@host ~ #] timedatectl status
[root@host ~ #] hwclock --show
```

Here is an example of *timedatectl* output:

```
                      Local time: Thu 2017-09-21 16:08:56 CEST
                  Universal time: Thu 2017-09-21 14:08:56 UTC
                        RTC time: Thu 2017-09-21 14:08:56
                       Time zone: Europe/Warsaw (CEST, +0200)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```

Don't be confused. Both *Local time* and *Universal time* are system time but with different time standards. The third one (RTC) is hardware time value. Obviously, RTC in this example is treated as Universal Time, which can be verified by the last line - RTC in local TZ: no

Set time:

```bash
[root@host ~ #] hwclock [--systohc | --hctosys] [--utc | --localtime]
[root@host ~ #] timedatectl set-local-rtc 0/1
```

### time zone

```bash
[root@host ~ #] ln -sf /usr/share/zoneinfo/Asia/Chongqing /etc/localtime (set system time zone manually)
# or
[root@host ~ #] timedatectl set-timezone Asia/Chongqiong
```

1. From the past experience (i.e. Gentoo installation), it's better to set time first and then time zone.
2. If all that fails, then resort to [ntpd](https://wiki.archlinux.org/index.php/Ntpd).

# pacman

```bash
[root@host ~ #] pacman -Ss pkg      # search for pkg
[root@host ~ #] pacman -Si pkg      # show pkg information
[root@host ~ #] pacman -S pkg1 pkg2 # install pkg1 and pkg2
[root@host ~ #] pacman -Qs pkg      # search for locally installed pkg 
[root@host ~ #] pacman -Qi pkg      # show locally installed pkg information
[root@host ~ #] pacman -R pkg       # remove pkg but leave dependencies alone
[root@host ~ #] pacman -Rs pkg      # remove pkg and orphan dependencies
```

# [Xterm](https://wiki.archlinux.org/index.php/Xterm)

Install *xterm* package. Add the following into *~/.Xresources*:

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

To take these settings into effec immediately:

```bash
[root@host ~ #] xrdb -merge ~/.Xresources
```

## Copy/Paste

X11 has two kinds of buffer (also named as *selection*): PRIMARY and CLIPBOARD. Usually, to copy/paste to/from the CLIPBOARD , we select (highlight) text & press Ctrl-C and Ctrl-V. Selected text will be copied to PRIMARY automatically. To paste from PRIMARY, press the middle mouse button. *Shift-insert* also pastes from PRIMARY, but only support by terminal emulator. Implicitly, PRIMARY buffer lives a short period and will be replaced with new text selection.

>Literally, we also call *paste* as *insert* from buffer.

Buffer/Selection | In/Copy | Out/Paste/Insert
--- | --- | ---
PRIMARY | select | MiddleMouse/Shift-Insert
CLIPBOARD | select & Ctrl-C | Ctrl-V

The actual buffer and key shortcut used are determined by X application. When it comes to Xterm, it select text to PRIMARY by default. Swtich between PRIMARY and CLIPBOARD by Ctrl-MiddbleMouse and tick/untick *Select to Clipboard* or:

```
# ~/.Xresources
XTerm.vt100.selectToClipboard: true/false
```

The first switch method is preferred as it's easier to adjust buffer setting according to associated application. The switch only changes which buffer to use while keeps the default operations (select, middle mouse and/or Shift-Insert).

To enable both PRIMARY and CLIPBOARD buffers, we leave the default while adding copy/paste operations. Thus both buffers are active:

```
# ~/.Xresources
XTerm.vt100.translations: #override \n\
    Ctrl Shift <Key>C: copy-selection(CLIPBOARD) \n\
    Ctrl Shift <Key>V: insert-selection(CLIPBOARD)
```

Notice: each key binding line must be separated by escape sequence *\n\*.

# VirtualBox sharedfolder

1. Make sure *vboxservice* is enabled and started.
2. Add user account to *vboxsf* group: *usermod -aG vboxsf username*.

Add shared folder:

```bash
user@host ~ $ VBoxManage sharedfolder add archlinux --name share_name --hostpath /path/to/host/folder [--automount]
```

Arch Linux guest can mount the shared folder manually (mount -t vboxsf), automatically (vboxmanage --automount), or by *fstab*. Here is an example of *fstab* method:

```
# /etc/fstab
share_name	/mount/point	vboxsf	uid=user,gid=group,rw,dmode=700,fmode=600,noauto,x-systemd.automount 
```

The last two arguments *noauto,x-systemd.automount* avoid service racing on booting (i.e. guest additions are not loaded yet while systemd mounts partitions in *fstab*).

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

We can also set Grub menu resolution by:

```
GRUB_GFXMODE="1366x768x24" (for Grub itself)
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

```bash
[root@host ~ #] pacman -S fcitx fcitx-configtool [ fcitx-gtk3 | fcitx-qt4 ]
```

*fcitx-gtk* and *fcitx-qt* is optional. You only want it when GTK/QT applications cannot input Chinese.

Append *run fcitx-autostart* into *~/.config/awesome/autostart.sh* and relaunch awesome.

Fcitx has built-in Pinyin that is really fast. Open *fcitx-configtool* and add Pinyin to input list.

Finally, Ctrl-Space.

# to-dos

1. gentoo bypass lock screen; xdg_vtnr
2. gentoo linput instead of synaptics?
