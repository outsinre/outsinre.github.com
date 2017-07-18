---
layout: post
title: VirtualBox
---

This post indroduces installing VirtualBox in Gentoo host, and then create a Windows XP 32-bit virtual machine.

1. VirtualBox depends on QT as GUI. But I would like a XFCE4 desktop without any QT packages. QT related USE flags are disabled in Gentoo *make.conf*.

   So I will emerge VirtualBox without GUI. To manage VMs, resort to CLI.
2. USEs

   ```bash
   # echo "app-emulation/virtualbox headless additions extensions" > /etc/portage/package.use/virtualbox
   ```

   1. *headless* builds VirtualBox without any graphic frontend.
   2. *attitions* makes interaction between host and guest easier, which draws in *app-emulation/virtualbox-additions*.

      Since VirtualBox 5, this USE is removed. Emerge *app-emulation/virtualbox-additions* manually.
   3. *extension* pulls in proprietary binary VirtualBox Extension Pack *app-emulation/virtualbox-extpack-oracle*.

      Since VirtualBox 5, this USE is removed.  Emerge *app-emulation/virtualbox-extpack-oracle* manually. Mostly, it supports:

      1. Virtual USB 2.0 (EHCI) device;
      2. VirtualBox Remote Desktop Protocol (VRDP) support.

	 Actually *vnc* USE is an alternative to VRDP but sucks.
      3. Refer to [Installing VirtualBox and extension packs](https://www.virtualbox.org/manual/ch01.html#intro-installing).

3. Scheme

   Windows XP 32-bit VM won't even start without QT GUI support. But VRDP helps! VRDP is a replacement of QT GUI!

   1. First create VM with VirtualBox CLI;
   2. Enable VRDP support for VM;
   3. Connect to VM by VRDP.

4. Install

   ```bash
   # emerge -avt app-emulation/virtualbox
   # emerge -avt app-emulation/virtualbox-additions (>=5)
   # emerge -avt app-emulation/virtualbox-extpack-oracle (>=5)
   ```

   It reminds accepting PUEL licence. Refer to [VirtualBox wiki](https://wiki.gentoo.org/wiki/VirtualBox).

   If fail to emerge VirtualBox, then probably you should bump to a newer version. Lastest Linux kernel usually requires lastest VirtualBox.
5. Group - vboxusers

   ```bash
   # gpasswd -a username vboxusers
   ```

   Add current *username* to *vboxusers* group.

   Logout and re-login to have this command take effect.
6. Kernel modules

   ```bash
   # modprobe vboxdrv
   ```

   Some optional modules: *vboxnetadp, vboxnetflt, vboxpci*. If run VirtualBox frequently, add these modules to */etc/conf.d/modules* for automatic loading on boot.

   Up to now everyting related to VirtualBox itself is prepared. Next is to create VM through CLI.
7. VBoxManage - create VM under *normal* user account

   ```bash
   $ VBoxManage list ostypes
   $ VBoxManage createvm --name WinXP32 --ostype WindowsXP --register
   $ VBoxManage registervm ~/.VirtualBox/Machines/WinXP32/WinXP32.vbox
   $ VBoxManage modifyvm WinXP32 --memory 384 --acpi on --nic1 nat --nictype1 Am79C973 --audio alsa --audiocontroller ac97 --usb on --usbehci on --vrde on --vrdeaddress 127.0.0.1 --vrdeport 5000,5010-5012 --clipboard bidirectional
   $ VBoxManage storagectl WinXP32 --name "IDE Controller" --add ide --controller PIIX4
   $ VBoxManage storageattach WinXP32 --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium "/media/Misc/VMs//WinXP32.vdi"
   $ VBoxManage storageattach WinXP32 --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium /usr/share/virtualbox/VBoxGuestAdditions.iso
   $ VBoxManage sharedfolder add WinXP32 --name "WLshare" --hostpath "/media/Misc/VMs/WLshare"
   ```

   1. ostypes

      List supported OSes. I am going to install *Windows XP 32-bit*. The corresponding `--ostype` is *WindowsXP*.
   2. createvm

      `--name` should be enclosed by double quotes in case of white spaces.

      `--register` to register the VM instantly, or run *VBoxManage registervm* afterwards.

      `--basefoler` can be used to specify this VM's files location (i.e. set to NTFS partition). If not set, it will default to *${HOME}/.VirtualBox/Machines* which is different from VirtualBox confiuration file.

   3. $ VBoxManage registervm ~/.VirtualBox/Machines/WinXP32/WinXP32.vbox

      *registervm* only accepts *full* path of *.vbox* file. If you are using an existing virutal machine disk files (instead of fresh installation), *registervm* after the *.vdi* file is specified.
   4. modifyvm

      `--nictype1 Am79C973` sets virtual Ethernet hardware to AMD PCNet FAST III (Am79C973). VirtualBox 5 now use Intel PRO/1000 T Server (82543GC) as default, which requires extra drivers installed.

      `--vrde on` is to enable VRDP support thus I can remotely connect to the VM by RDP client.

      ` --usb on --usbehci on` enables USB 1.0 and 2.0. To enable 3.0, use `--usbxhci on`.

      `--vrdeaddress` set to 127.0.0.1 loopback address. If unset, it defaults to 0.0.0.0 listening on all host network interfaces. Refer to [127.0.0.1 vs 0.0.0.0](/2015/09/14/0000-127001-localhost/).

      `--draganddrop` option is useful if you need it. However, it is vunerable to security issue.
   5. storagectl

      Set disk controller for VM. Don't use SATA related controller for *WindowsXP*.
   6. storageattach VDI

      *WinXP32.vdi* is copied from some guy in QQ group. So I don't need a installation ISO to install VM from scratch. For ISO from scratch, refer to references listed.
   7. storageattach Additions

      *VBoxGuestAddiontions.iso* supports file sharing, mouse switch etc between *host* and *guest*. WE will install this toolbox after entering *WindowsXP* VM.

      Up to now, the VM is created! But VirtualBox does not have QT GUI. We need RDP client, i.e. FreeRDP.
   8. sharedfolder

      Create a folder for share on host and mount it on guest. After entering VM, open Windows Explorer and look for it under "My Networking Places", "Entire Network", "VirtualBox Shared Folders", "\\\Vboxsvr". By right-clicking on a shared folder and selecting "Map network drive" from the menu that pops up, you can assign a drive letter to that shared folder. If don't assign a drive letter, each time to access the shared, we have to find it under "\\\Vboxsvr".
7. Up to now, the VM is created! But VirtualBox does not have QT GUI. We need RDP client, i.e. FreeRDP.

   ```bash
   # emerge -avt net-misc/freerdp
   ```

8. startvm

   ```bash
   $ VBoxHeadless --startvm WinXP32 --vrdeproperty "TCP/Ports=5001,5010-5012"
   ```

   *VBoxHeadless* is a tool the launch VM as a server mode instead of traditional GUI mode. If you guy a VPS, say from Linode, your VPS VM mostly runs as a similar mode (maybe through web protocol for your control in brower).

   The RDP TCP port is set to an optional list *5001,5010-5012*. If not set, the default is *3389*. Usually, *5001* is not occupied in system, so most of the time, it is chosen as the port to listen for RDP connection.

   Now the VM is started, but not showing up! We need to use FreeRDP to get X.
9. RDP magic

   ```bash
   $ xfreerdp +clipboard /w:1024 /h:576 /v:127.0.0.1:5001 >/dev/null 2>&1 &
   ```

    1. The server IP is the IP address of *host*, NOT IP of *guest*. Since I connect to VM locally, so it *127.0.0.1*. *xfreerdp* can add many other parameters, like window size, audio, video etc.
    2. 'Ctrl + Alt + Enter' to toggle full screen.
    3. You might found there are two mouse pointers, one for *host* while another for *guest*. Also the screen resolution is not correctly set. To solve issues like this, to install *VBoxGuestAdditions*. Open file explorer, installer is located in partition *(D:) VirtualBox Guest Additions*, *VBoxWindowsAdditions.exe*. Before issues got solved, you can use keyboard shortcuts and TAB, ENTER etc keys.
    4. Remember to install the guest addtions and mount shared folder. During *VBoxGuestAddtions* installation, there is a option *Direct3D* which should NOT be enabled.
10. Forcefully poweroff

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
11. Remove guest addtions ISO

    ```bash
    $ VBoxManage storageattach WinXP32 --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium emptydrive
    ```

    Detach ISO file after *VBoxGuestAddtions* is installed.
12. Launch scripts.
    1. */usr/local/sbin/vboxmodule*

       ```bash
       #!/bin/bash
       for m in vbox{drv,netadp,netflt}; do modprobe $m; done
       echo 'VirtualBox modules loaded!'
       ```

    2. *${HOME}/bin/vboxWinXP32*

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
                   VBoxHeadless --startvm WinXP32 --vrdeproperty "TCP/Ports=5001,5010-5012"
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
13. Configurations.

    After getting into the XP system, we can still tune some of the configurations including:

    1. If English XP, then set *non-Unicode* language to *Chinese* and set the *Regional Options* values.
    2. Disable *soundman* etc. on startup.
    3. Set *Wireless Zero Configuration*, *Windows Audio*, *Windows Themes* etc. services to *Disabled* or *Manual*. Refer to [windows xp终极优化](http://blog.sciencenet.cn/blog-76534-508692.html).
    4. If installed 迅雷, then set *XLServicePlatform* to *Manual*.
    5. My Computer, Properties, Advanced, Performance, Settings, Adjust for best performance.
13. Modules rebuild for new kernel. Read the first reference on *Kernel driver not installed* section.

    BEFORE booting with new kernel (by kernel upgrading), you could no longer load modules like *vobxdrv*.

    ```bash
    # modprobe vboxdrv
    modprobe: FATAL: Module vboxdrv not found.
    ```
    
    The solution is to reinstall VirtualBox external modules `emerge -avt1 app-emulation/virtualbox-modules` or `emerge -avt1 @module-rebuild`. More read [Upgrade kernel to unstable 4.0.0](/2015/03/25/gentoo-installation/).
14. USB

   Attach plugged in USB device to VM. First make sure USB 1.0/2.0/3.0 is enabled for VM. Then:

   ```bash
   $ vboxmanage list usbhost
   $ vboxmanage controlvm WinXP32 usbattach USB-UUID
   $ vboxmanage list usbhost
   $ vboxmanage controlvm WinXP32 usbdetach USB-UUID
   ```

   1. Find the USB device UUID by checking the Manufacture and Product field. Make sure the Current State field is Available.
   2. Attach the USB to VM.
   3. The Current State field becomes Captured (by VM).
   4. Detach from VM.
   5. *Attention*: get UUID from *vboxmanage list usbhost* instead of *blkid*.

   We can also make this attachment permanent by creating a *usbfilter*.
15. References
    1. [gentoo wiki](https://wiki.gentoo.org/wiki/VirtualBox)
    2. [install_virtualbox_in_gentoo](http://baige5117.github.io/blog/install_virtualbox_in_gentoo.html)
    3. [createVBoxVM.sh](https://github.com/rustymyers/scripts/blob/master/shell/createVBoxVM.sh)
    4. [how-to-attach-a-virtual-hard-disk-using-vboxmanage](http://serverfault.com/questions/171665/how-to-attach-a-virtual-hard-disk-using-vboxmanage)
    5. [Remote virtual machines](http://www.virtualbox.org/manual/ch07.html#idp46785385220400)
    6. [headless vm](https://www.howtoforge.com/vboxheadless-running-virtual-machines-with-virtualbox-4.3-on-a-headless-ubuntu-14.04-lts-server)
    7. [guestadditions in headless](https://forums.virtualbox.org/viewtopic.php?f=7&t=18012)
    8. [vbox create headless vm](http://askaralikhan.blogspot.com/2011/01/virtualbox-40-creating-virtual-machine.html)
