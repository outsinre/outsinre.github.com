---
layout: post
title: VirtualBox
---

This post indroduces installing VirtualBox in Gentoo host, and then create a Windows XP 32-bit virtual machine.

# Installation

1. VirtualBox depends on QT as GUI. But I would like a XFCE4 desktop without any QT packages. QT related USE flags are disabled in Gentoo *make.conf*.

   So I will emerge VirtualBox without GUI. To manage VMs, resort to CLI.
2. USEs

   ```bash
   root@tux / # echo "app-emulation/virtualbox headless additions extensions" > /etc/portage/package.use/virtualbox
   ```

   1. *headless* builds VirtualBox without any graphic frontend.
   2. *attitions* makes interaction between host and guest easier, which draws in *app-emulation/virtualbox-additions*.

      Since VirtualBox 5, this USE is removed. Emerge *app-emulation/virtualbox-additions* manually.
   3. *extension* pulls in proprietary binary VirtualBox Extension Pack *app-emulation/virtualbox-extpack-oracle*.

      Since VirtualBox 5, this USE is removed.  Emerge *app-emulation/virtualbox-extpack-oracle* manually. Mostly, it supports:

      1. Virtual USB 2.0 (EHCI) and 3.0 (xHCI) device;
      2. VirtualBox Remote Desktop Protocol (VRDP) support.

         Actually *vnc* USE is an alternative to VRDP but sucks.
      3. Host webcam passthrough. 
      4. Refer to [Installing VirtualBox and extension packs](https://www.virtualbox.org/manual/ch01.html#intro-installing).

3. Build

   ```bash
   root@tux / # emerge -avt app-emulation/virtualbox
   root@tux / # emerge -avt app-emulation/virtualbox-additions (>=5)
   root@tux / # emerge -avt app-emulation/virtualbox-extpack-oracle (>=5)
   ```

   It reminds accepting PUEL licence. Refer to [VirtualBox wiki](https://wiki.gentoo.org/wiki/VirtualBox).

   If fail to emerge VirtualBox, then probably you should bump to a newer version. Lastest Linux kernel usually requires lastest VirtualBox.
4. Group - vboxusers

   ```bash
   root@tux / # gpasswd -a username vboxusers
   ```

   Add current *username* to *vboxusers* group. Check *newgrp* in Wireshark post.
5. Kernel modules

   ```bash
   root@tux / # modinfo vboxdrv vboxnetadp vboxnetflt vboxpci
   root@tux / # modprobe -v vboxdrv vboxnetadp vboxnetflt vboxpci
   ```

   1. If you launch VirtualBox frequently, add these modules to */etc/conf.d/modules* for automatic loading on boot.
   2. *vboxdrv* is the core module and must always be present. *vboxnetadp* and *vboxnetflt* are required for any VM networking beyond the default NAT mode (i.e. host-only, bridged etc.).

   When booting into a new Linux kernel (i.e kernel upgrading), you could no longer load modules like *vobxdrv*.

   ```bash
   # modprobe vboxdrv
   modprobe: FATAL: Module vboxdrv not found.
   ```

   Read the first reference on *Kernel driver not installed* section. The solution is to rebuild VirtualBox external modules `emerge -avt1 app-emulation/virtualbox-modules` or `emerge -avt1 @module-rebuild`. More details in [Upgrade kernel to unstable 4.0.0](/2015/03/25/gentoo-installation/).
6. Up to now everyting related to VirtualBox itself is prepared. Next is to create VM through CLI.

# Scheme

VM won't even start without QT GUI support. But VRDP helps! VRDP is a replacement of QT GUI!

1. First create VM with VirtualBox CLI;
2. Enable VRDP support for VM;
3. Connect to VM by VRDP.

Notice:

1. The less snapshots we create, the better is the performance.
2. Put *.vdi* file on host home partition for better performance.

# Windows XP 32-bit

[VBoxManage createvm](https://www.virtualbox.org/manual/ch07.html#idm3213):

```bash
user@tux ~ $ VBoxManage list ostypes
user@tux ~ $ VBoxManage createvm --name WinXP32 --ostype WindowsXP --register --basefolder ~/Documents/VirtualBox/Machines
user@tux ~ $ VboxManage list vms
user@tux ~ $ VBoxManage registervm ~/Documents/VirtualBox/Machines/WinXP32/WinXP32.vbox (opt); vboxmanage list vms
user@tux ~ $ VBoxManage modifyvm WinXP32 --memory 384 --acpi on --nic1 nat --nictype1 Am79C973 --audio alsa --audiocontroller ac97 --usb on --usbehci on --vrde on --vrdeaddress 127.0.0.1 --vrdeport 5001,5010-5012 --clipboard bidirectional
user@tux ~ $ VBoxManage createmedium --filename ~/Documents/VirtualBox/Machines/WinXP32/WinXP32.vdi --size 5000 (omitted)
user@tux ~ $ VBoxManage storagectl WinXP32 --name "IDE Controller" --add ide --controller PIIX4
user@tux ~ $ VBoxManage storageattach WinXP32 --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium ~/Documents/VirtualBox/Machines/WinXP32/WinXP32.vdi
user@tux ~ $ VBoxManage storageattach WinXP32 --storagectl "IDE Controller" --port 0 --device 1 --type hdd --medium /path/to/ISO (omitted)
user@tux ~ $ VBoxManage storageattach WinXP32 --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium /usr/share/virtualbox/VBoxGuestAdditions.iso (or --device 1 --medium additions)
user@tux ~ $ VBoxManage sharedfolder add WinXP32 --name WLshare --hostpath /media/Misc/VMs/WLshare
```

Here, I re-use an existing *.vdi* instead of guest installation ISO, thus avoiding guest OS installing process. Apparently, *createmedium* prodecure is omitted.

1. ostypes

   List supported OSes. I am going to install *Windows XP 32-bit*. The value is *WindowsXP*.
2. createvm
   + `--name` should be enclosed by double quotes in case of white spaces.
   + `--register` to register the VM instantly, or run *VBoxManage registervm* afterwards.
   + `--basefoler` can be used to specify this VM's files location (i.e. set to NTFS partition). If not set, it will default to *${HOME}/.VirtualBox/Machines* which is different from VirtualBox confiuration file.
3. registervm

   **Bug**: `--register` of *createvm* failed to register the VM though it reports negtive success. Possible solutions:

   1. Specifically add `--basefolder` to *createvm --register*.
   2. Pass *full* path of *.vbox* file to *registervm* on a separate shell command.
4. modifyvm
   + `--nictype1 Am79C973` sets virtual Ethernet hardware to AMD PCNet FAST III (Am79C973). VirtualBox 5 now use Intel PRO/1000 T Server (82543GC) as default, which requires extra drivers installed.
   + `--audio alsa` could be *pulse* instead.
   + `--vrde on` is to enable VRDP support thus I can remotely connect to the VM by RDP client.
   + ` --usb on --usbehci on` enables USB 1.0 and 2.0. To enable 3.0, use `--usbxhci on`.
   + `--vrdeaddress` set to loopback address. If unset, it binds to all host network interfaces accepting both IPv4 and IPv6. Refer to [127.0.0.1 vs 0.0.0.0](/2015/09/14/0000-127001-localhost/).
   + `--draganddrop` option is useful if you need it. However, it is vunerable to security issue.
5. storagectl

   Set disk controller for VM. Don't use SATA related controller for *WindowsXP*.
6. storageattach VDI

   *WinXP32.vdi* is copied from some guy in QQ group, getting rid of installing VM OS from scratch.
7. storageattach

   *VBoxGuestAddiontions.iso* supports file sharing, mouse switch etc between *host* and *guest*. WE will install this toolbox after entering *WindowsXP* VM.
8. sharedfolder

   Create a folder for share on host and mount it on guest. After entering VM, open Windows Explorer and look for it under "My Networking Places", "Entire Network", "VirtualBox Shared Folders", "\\\Vboxsvr". By right-clicking on a shared folder and selecting "Map network drive" from the menu that pops up, you can assign a drive letter to that shared folder. If don't assign a drive letter, each time to access the shared, we have to find it under "\\\Vboxsvr".

   If you cannot locate the shared folder under Windows Explorer/Network, check if Network Discovery is turned on in Control Panel (default for Home networking type). You can also try CMD:

   ```
   net use z: \\vboxsvr\WLshare
   # or
   net use z: \\vboxsrv\WLshare
   ```

9. Up to now, the VM is prepared!

   We need RDP client.


# RDP / VRDP / VRDE

VirtualBox Remote Display Protocol (VRDP) is a backwards-compatible extension to Microsoft's Remote Desktop Protocol (RDP). VirtualBox Remote Desktop Extension (VRDE) is an Oracle VRDP implementation. Apart from TCP/Ports and TCP/Address, we have:

+ `--vrdemulticon on` allows multiple simultaneous connections to the same VM, withou which we can resort to `--vrdereusecon on` that can cut off the current client for sake of a new one.
+ `--vrdeauthtype external` requires username and password of the host before connection.

Here is FreeRDP setup details:

1. Up to now, the VM is created! But VirtualBox does not have QT GUI. We need RDP client, i.e. FreeRDP.

   ```bash
   root@tux / # emerge -avt net-misc/freerdp
   ```

2. startvm

   ```bash
   user@tux ~ $ VBoxManage startvm "WinXP32" --type headless
   # or
   user@tux ~ $ VBoxHeadless --startvm WinXP32
   ```

   *VBoxHeadless* is a tool the launch VM as a server mode without GUI. If you buy a VPS, the OS mostly runs in a similar mode (maybe VNC browser).

   Now the VM is started, but not showing up! We need to use FreeRDP to get X.
3. FreeRDP

   The RDP TCP port is set to an optional list *5001,5010-5012* (default *3389*).

   ```bash
   $ xfreerdp +clipboard /w:1024 /h:576 /v:127.0.0.1:5001
   ```

    1. The server IP is *vrdeaddress*, NOT IP of *guest*. Since I connect to VM locally, so it *127.0.0.1*. *xfreerdp* can add many other parameters, like window size, audio, video etc.
    2. 'Ctrl + Alt + Enter' to toggle full screen.
    3. You might found there are two mouse pointers, one for *host* while another for *guest*. Also the screen resolution is not correctly set. Install *VBoxGuestAdditions* we've attached the Vm. Open file explorer, installer is located in partition *(D:) VirtualBox Guest Additions*, *VBoxWindowsAdditions.exe*. Disable *Direct3D* component on a low end VM.
4. Shutdown

    ```bash
    $ VBoxManage controlvm WinXP32 savestate/acpipowerbutton/poweroff/pause
    ```

    1. *pause*: temporarily puts a virtual machine on hold, without changing its state for good. The VM window will be painted in gray to indicate that the VM is currently paused. 
    2. *poweroff*: has the same effect on a virtual machine as pulling the power cable on a real computer. Again, the state of the VM is not saved beforehand, and data may be lost. (This is equivalent to selecting the "Close" item in the "Machine" menu of the GUI or pressing the window's close button, and then selecting "Power off the machine" in the dialog.)
    3. *savestate*: saves the current state of the VM to disk and then stop the VM. (This is equivalent to selecting the "Close" item in the "Machine" menu of the GUI or pressing the window's close button, and then selecting "Save the machine state" in the dialog.) We can think *savestate* as suspend to disk (*hibernate*).
    4. *acpipowerbuttion*: simulates long press the PC's power buttion to shutdown directly without saving.

    Which to use, depens on:

    1. If jobs in VM is not yet finished (like editing a file), use *savestate*;
    2. If every jobs completed, use *acpipoweroff* (as long as ACPI is enabled for VM) or *poweroff*;
    3. If still want to leave the VM running remotely, just close the RDP window.
5. Remove guest addtions ISO

    ```bash
    user@tux ~ $ VBoxManage storageattach WinXP32 --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium emptydrive / none
    ```

    Detach ISO file after *VBoxGuestAddtions* is installed.

# Scripts

1. */usr/local/sbin/vboxmodule*

   ```bash
   #!/bin/bash
   for m in vbox{drv,netadp,netflt,pci}; do modprobe -v $m; done
   echo 'VirtualBox modules loaded!'
   ```

2. *${HOME}/bin/winXP32*

   ```bash
   #!/bin/bash
   # Bash Menu Script Example

   PS3='Please enter your choice on VM WinXP32: '
   options=("startvm" "rdp" "poweroff" "acpipowerbutton" "savestate" "quit")
   select opt in "${options[@]}"
   do
       case $opt in
	   "startvm")
	       echo "you choose to launch WinXP32"
	       VBoxHeadless --startvm WinXP32
	       break
	       ;;
	   "rdp")
	       echo "you choose to connect WinXP32"
	       xfreerdp +clipboard /sound /f /v:127.0.0.1:5001
	       break
	       ;;
	   "acpipowerbutton")
	       echo "you chose to acpipowerbuttion WinXP32"
	       VBoxManage controlvm WinXP32 poweroff
	       break
	       ;;
	   "savestate")
	       echo "you chose to save WinXP32state"
	       VBoxManage controlvm WinXP32 savestate
	       break
	       ;;
	   "poweroff")
	       echo "you chose to poweroff WinXP32"
	       VBoxManage controlvm WinXP32 poweroff
	       break
	       ;;
	   "quit")
	       break
	       ;;
	   *) echo 'invalid option'
	      ;;
       esac
   ```

   Pay attention `PS3` and `case` *bash* usage.

# Configurations.

After getting into the XP system, we can still tune some of the configurations including:

1. If English XP, then set *non-Unicode* language to *Chinese* and set the *Regional Options* values.
2. Disable *soundman* etc. with *msconfig*.
3. Set *Wireless Zero Configuration*, *Windows Audio*, *Windows Themes* etc. services to *Disabled* or *Manual*. Refer to [windows xp终极优化](http://blog.sciencenet.cn/blog-76534-508692.html).
4. If installed '迅雷', then set *XLServicePlatform* to *Manual*.
5. My Computer, Properties, Advanced, Performance, Settings, Adjust for best performance.

# USB

Attach plugged in USB device to VM. First make sure USB 1.0/2.0/3.0 is enabled for VM. Then:

```bash
user@tux ~ $ vboxmanage list usbhost
user@tux ~ $ vboxmanage controlvm WinXP32 usbattach USB-UUID
user@tux ~ $ vboxmanage list usbhost
user@tux ~ $ vboxmanage controlvm WinXP32 usbdetach USB-UUID
```

1. Find the USB device UUID by checking the Manufacture and Product field. Make sure the Current State field is Available.
2. Attach the USB to VM.
3. The Current State field becomes Captured (by VM).
4. Detach from VM.
5. Get UUID from *vboxmanage list usbhost* instead of *blkid*.

We can also make this attachment permanent by creating a *usbfilter*.

# Host-only networking

Take Android-x86 for example:

```bash
user@tux ~ $ VBoxManage hostonlyif create/remove vboxnet0
user@tux ~ $ ip link
user@tux ~ $ VBoxManage list hostonlyifs
user@tux ~ $ VBoxManage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1 ('--dhcp' is not supported currently)
user@tux ~ $ VBoxManage modifyvm Android51 --nic2 hostonly --hostonlyadapter2 vboxnet0
user@tux ~ $ VBoxManage dhcpserver add/modify --ifname vboxnet0 (--netname HostInterfaceNetworking-vboxnet0) --ip 192.168.56.1 --netmask 255.255.255.0 --lowerip 192.168.56.100 --upperip 192.168.56.110 --enable
user@tux ~ $ VBoxManage list dhcpservers
user@tux ~ $ ip address
# boot Android51
user@tux ~ # netcfg eth1 dhcp
```

1. WES7x86 fails to set *default gateway* as *192.168.56.1*. Fix it manually! If the new adapter (Local Area Connection 2) is *Public network*, host cannot connect to (i.e. *ping*) WES7x86. Switch to *Home network*.
2. Check *iptables* and/or firewall.

   ```bash
   user@tux ~ # iptables -I INPUT/OUTPUT 4 -s 192.168.56.0/24 -j ACCEPT
   ```

4. Apart from NAT/bridged networking, we can use *iptables redirect* to let *vboxnet0* traffic go outside.
5. Especially, guest VM can share host's proxy as long as it's listening on *local network* (i.e. *0.0.0.0*). You may want to prohibit LAN devices connection to proxy.

   ```bash
   user@tux ~ # iptables -I INPUT 4 -d 192.168.0.0/24 -i wlan0 -p tcp -m tcp --dport 1080 -j DROP
   ```

   Alternatively, *iptables redirect vboxnet0* traffic to *127.0.0.1* like:

   ```
   user@tux ~ # iptables -t nat -A PREROUTING [ -s 192.168.56.101 ] -d 192.168.56.100 -i vboxnet0 -p tcp -m tcp --dport 1080:1081 -j DNAT --to-destination 127.0.0.1
   user@tux ~ # sysctl -w net.ipv4.conf.vboxnet0.route_localnet=0 (runtime)
   # or 
   user@tux ~ # sysctl -p /etc/sysctl.d/15-route_localnet.conf
   net.ipv4.conf.vboxnet0.route_localnet = 1 (better)
   net.ipv4.conf.all.route_localnet = 1 (dangerous)
   ```

   By default, kernel [refuses to route](https://security.stackexchange.com/a/137603) source or destination loopback addresses (i.e. 127.0.0.1). *vboxnet0* interface would not be created before VirtualBox launches. So *15-route_localnet.conf* does not apply accross boot.

# Delete VM

```bash
user@tux ~ $ vboxmanage list vms
user@tux ~ $ vboxmanage unregistervm WinXP32 --delete
```

Amost everything related to the VM is deleted, especially the virtual disk image file (*.vdi*), snapshots, saved state files, VM xml file etc.

# Windows Embedded Standard 7 x86

```bash
~ $ VBoxManage list ostypes (Windows7)
~ $ VBoxManage createvm --name WES7x86 --ostype Windows7 --register --basefolder ~/Documents/VirtualBox/Machines
~ $ VBoxManage registervm ~/Documents/VirtualBox/Machines/WES7x86/WES7x86.vbox (opt)
~ $ VBoxManage modifyvm WES7x86 --memory 700 --audio alsa --audiocontroller hda --acpi on --vrde on --vrdeproperty "TCP/Ports=5001,5010-5012" --vrdeproperty "TCP/Address=127.0.0.1" --clipboard bidirectional (--draganddrop bidirectional --usb on --usbehci on, --nic1 bridged --bridgeadapter1 wlp3s0, --nic1 nat, --boot1 dvd --boot2 disk --boot3 none --boot4 none)
~ $ VBoxManage createmedium --filename ~/Documents/VirtualBox/Machines/WES7x86/WES7x86.vdi --size 7000 (createhd)
~ $ VBoxManage storagectl WES7x86 --name "SATA Controller" --add sata --controller IntelAHCI --portcount 3 (SATA and Intel AHCI)
~ $ VBoxManage storageattach WES7x86 --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium ~/Documents/VirtualBox/Machines/WES7x86/WES7x86.vdi
~ $ VBoxManage storageattach WES7x86 --storagectl "SATA Controller" --port 1 --device 0 --type dvddrive --medium /media/Misc/WLshare/en_windows_embedded_standard_7_runtime_x86_dvd_521803.iso
~ $ VBoxManage storageattach WES7x86 --storagectl "SATA Controller" --port 2 --device 0 --type dvddrive --medium /usr/share/virtualbox/VBoxGuestAdditions.iso
~ $ VBoxManage sharedfolder add WES7x86 --name WLshare --hostpath /media/Misc/WLshare
# Installation processing ...
~ $ VBoxManage storageattach WES7x86 --storagectl "SATA Controller" --port 1 --device 0 --type dvddrive --medium emptydrive
~ $ VBoxManage storageattach WES7x86 --storagectl "SATA Controller" --port 2 --device 0 --type dvddrive --medium none
```

For details on choosing template and components during installation, read the Windows post. Since the ISO image is till attached to SATA Controller, booting will be directed to installation process again. Either update VM boot order or F12 at early phase.

# Android-x86

```bash
~ $ VBoxManage createvm --name Android51 --ostype Linux26 --register --basefolder /media/Misc/VirtualBox/Machines
~ $ VBoxManage modifyvm Android51 --memory 700 --acpi on --mouse usbtablet --usb on --usbehci on --vrde on --vrdeproperty "TCP/Ports=5001,5010-5012" --vrdeproperty "TCP/Address=127.0.0.1" --clipboard bidirectional
~ $ VBoxManage createmedium --filename /media/Misc/VirtualBox/Machines/Android51/Android51.vdi --size 4000
~ $ VBoxManage storagectl Android51  --name "IDE Controller" --add ide --controller PIIX4
~ $ VBoxManage storageattach Android51 --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium /media/Misc/VirtualBox/Machines/Android51/Android51.vdi
~ $ VBoxManage storageattach Android51 --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium ~/Downloads/cm-x86-13.0-r1.iso
~ $ VBoxManage list vms
```

1. Installation - Install Android-x86 to harddisk.
2. Create/Modify partitions.
3. Do you want to use GPT? No!
4. New - Primary - Bootable - Write - *yes* - Quit.
5. *sda1* unkown VBOX HARDDISK.
6. *ext3/ext4*.
7. Format *sda1* to *ext4*? Yes.
8. Do you want to install boot loader GRUB? Yes.
9. Do you want to install */system* directory as read-write? Yes.
1. Run Android-x86.

>If you choose GPT at 3rd step, then install EFI and GRUB2 instead of GRUB.

First boot takes several minutes to initialize system preparation.

# Arch Linux x86_64

```bash
~ $ VBoxManage list ostypes (ArchLinux_64)
~ $ VBoxManage createvm --name archlinux_64 --ostype ArchLinux_64 --register --basefolder /media/Misc/VirtualBox/Machines
~ $ VBoxManage modifyvm archlinux_64 --memory 512 --acpi on --vrde on --vrdeproperty "TCP/Ports=5001,5010-5012" --vrdeproperty "TCP/Address=127.0.0.1" --clipboard bidirectional
~ $ VBoxManage createmedium --filename /media/Misc/VirtualBox/Machines/archlinux_64/archlinux_64.vdi --size 4000
~ $ VBoxManage modifymedium /media/Misc/VirtualBox/Machines/archlinux_64/archlinux_64.vdi --resize 5000
~ $ VBoxManage storagectl archlinux_64 --name "SATA Controller" --add sata --controller IntelAHCI --portcount 3
~ $ VBoxManage storageattach archlinux_64 --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium /media/Misc/VirtualBox/Machines/archlinux_64/archlinux_64.vdi
~ $ VBoxManage storageattach archlinux_64 --storagectl "SATA Controller" --port 1 --device 0 --type dvddrive --medium ~/Downloads/archlinux-2017.11.01-x86_64.iso
~ $ VBoxManage sharedfolder add archlinux_64 --name WLshare --hostpath /media/Misc/WLshare
# starvm
~ $ VBoxManage startvm archlinux_64 --type headless
```

1. The VDI file is [resized](https://forums.virtualbox.org/viewtopic.php?f=35&t=50661) after creation. Then follow the [installation guide](https://wiki.archlinux.org/index.php/Installation_guide).
2. VBoxGuestAdditions in Linux guest requires extra effors. Details refer to Arch Linux post.

## [VirtualBox Guest Additions](https://www.virtualbox.org/manual/ch04.html#idm2096)

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

## VirtualBox sharedfolder

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

# Expand VDI size

Occasionally, the guest OS warns running out of disk space when you should [resize](https://forums.virtualbox.org/viewtopic.php?f=35&t=50661) the VM disk and parition thereof.

1. Firstly, make sure the guest does **not** have snapshots. If there exist any, delete all of them.
2. Resizing does not work if the guest VM resides on fix-sized VDI.
3. Currently, only expansion is supported. You cannot decrease any VDI or partitions.

Here, I will show an example of expanding *archlinux_64* VM above. On the host:

```bash
user@tux ~ $ VBoxManage list hdds
user@tux ~ $ VBoxManage modifymedium /media/Misc/VirtualBox/Machines/archlinux_64/archlinux_64.vdi --resize 10000
user@tux ~ $ VBoxManage list hdds
```

The VDI is increasing from 5GiB to 10GiB. Next, we should let the VM known the expansion. Basically, we attach to the VM a live booting media containing *parted* or *gparted* tool. GParted live CD and Arch Linux ISO are such examples. On the host:

```bash
user@tux ~ $ VBoxManage showvminfo archlinux_64
user@tux ~ $ VBoxManage storageattach archlinux_64 --storagectl "SATA Controller" --port 1 --device 0 --type dvddrive --medium ~/Downloads/archlinux-2018.01.01-x86_64.iso
user@tux ~ $ VBoxManage showvminfo archlinux_64
```

Please make sure boot the VM on the attached ISO instead of VDI. In the live system, suppose */dev/sda* represents VDI:

```bash
[root@archiso / #] fdisk/blkid/lsblk/findmnt
[root@archiso / #] parted -a optimal /dev/sda
(parted) help
(parted) print free
The backup GPT table is not at the end of the disk, as it should be. This might mean that another operating system believes the disk is smaller. Fix, by moving the backup to the end (and removing the old backup)? Fix/Ignore/Cancel?
(parted) Fix
(parted) print free
(parted) resizepart 1 100%
```

*parted* detects the VDI expanded and reminds *Fix*. The key is *resizepart* (or *resize* depending on *parted* version) epxanding (100%) the *root* partition to include newly added space. That's all!

1. Some posts write *resizepart* can be done on guest OS directly as long as swap partition/file is turned off.
2. For Windows guest, use enclosed *disk management* to finish the job.

# Troublshooting.

1. Re-install VirtualBox. `emerge -av1 $(qlist -IC virtualbox)`.
2. Check *VBoxSVC.log* and *VBox.log*.
3. Avoid quotes on bash symbols `~` and `$`.

   ```
   # /etc/udev/rules.d

   # Correct /dev/{vboxdrv,vboxdrvu,vboxnetctl} permissions to 0600
   KERNEL=="vboxdrv", NAME="vboxdrv", OWNER="root", GROUP="vboxusers", MODE="0660"
   KERNEL=="vboxdrvu", NAME="vboxdrvu", OWNER="root", GROUP="vboxusers", MODE="0660"
   KERNEL=="vboxnetctl", NAME="vboxnetctl", OWNER="root",GROUP="vboxusers", MODE="0660"
   ```

   To load the new rule, execute `udevadm trigger`.
4. Avoid 64-bit guest OS. 64-bit OS occupies one third more resources (disk, memory) than the 32-bit version. What was worse, it seems *VBoxGuestAddtions-amd64.exe* makes no difference on the guest (double mouse, weird resolution etc.).
6. WES7x86 no sound. Change `--audiocontroller` to [Intel *hda* instead of default *ac97*](https://www.virtualbox.org/manual/ch03.html#ftn.idm1540).
7. Suppose you have unticked an feature (i.e. IE 8), and want to add it back after installation, do the following:
   1. Find the feature package *cabinet* file from ISO, like *DS/Packages/FeaturePack/x86~winemb-ie-explorer~~~~6.1.7600.16385~1.0/WinEmb-IE-Explorer.cab*.
   2. Use *Pkgmr* or [DISM](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/dism-operating-system-package-servicing-command-line-options) command line to install the cabinet file. *dism* is far more powerful than what you think. For example, you can use it to check a cabinet file information.

   ```
   # Do it in an elevated command prompt.
   DISM /?
   Pkgmgr /?
   DISM /Online /Add-Package /PackagePath:"C:\Users\Brink\Desktop\WinEmb-IE-Explorer.cab"
   Pkgmgr /ip /m:C:\Users\Brink\Desktop\WinEmb-IE-Explorer.cab
   # restart system as requested
   ```

8. Windows XP `--port 2 --device 0` refuses to attach extra ISO:

   >VBoxManage: error: No drive attached to device slot 0 on port 2 of controller 'IDE Controller'

   Windows XP guest uses IDE PIIX4 disk drive that supports at most 2 ports. Each port supports two devices , namely (0, 0), (0, 1), (1, 0) and (1, 1). If you have a strong reason to use port 2, then create SATA disk drive like Win 7 guest.

   Especially, each port of SATA disk drive only supports one device, namely (0, 0), (1, 0), (2, 0) etc.

# References

1. [gentoo wiki](https://wiki.gentoo.org/wiki/VirtualBox)
2. [install_virtualbox_in_gentoo](http://baige5117.github.io/blog/install_virtualbox_in_gentoo.html)
3. [createVBoxVM.sh](https://github.com/rustymyers/scripts/blob/master/shell/createVBoxVM.sh)
4. [how-to-attach-a-virtual-hard-disk-using-vboxmanage](http://serverfault.com/questions/171665/how-to-attach-a-virtual-hard-disk-using-vboxmanage)
5. [Remote virtual machines](http://www.virtualbox.org/manual/ch07.html#idp46785385220400)
6. [headless vm](https://www.howtoforge.com/vboxheadless-running-virtual-machines-with-virtualbox-4.3-on-a-headless-ubuntu-14.04-lts-server)
7. [guestadditions in headless](https://forums.virtualbox.org/viewtopic.php?f=7&t=18012)
8. [vbox create headless vm](http://askaralikhan.blogspot.com/2011/01/virtualbox-40-creating-virtual-machine.html)
9. [VBoxManage createvm manully](https://www.virtualbox.org/manual/ch07.html#idm3213)
