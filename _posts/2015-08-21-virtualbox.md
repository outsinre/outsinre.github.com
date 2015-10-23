---
layout: post
title: VirtualBox
---
This post indroduces installing *VirtualBox* in *Gentoo host*, and then create a *Windows XP 32-bit* virtual machine.

1. *VirtualBox* depends on QT as GUI. But I would like a XFCE4 desktop without any QT packages. QT related USE flags are disabled in Gentoo *make.conf*.

    So I will emerge *VirtualBox* without GUI. To manage VMs, resort to CLI.
2. \# echo "app-emulation/virtualbox extensions" > /etc/portage/package.use/virtualbox

    *extension* USE flag will draw in binary *VirtualBox Extension Pack* which is not open source package *app-emulation/virtualbox-extpack-oracle*.

    Why do I need this extension binary package? Mostly, it supports:

    1. Virtual USB 2.0 (EHCI) device;
    2. VirtualBox Remote Desktop Protocol (VRDP) support.
    Refer to [Installing VirtualBox and extension packs](https://www.virtualbox.org/manual/ch01.html#intro-installing).

    *Windows XP 32bit* VM won't even start without QT GUI support. But VRDP helps! VRDP is a replacement of QT GUI!

    1. First create VM with *VirtualBox* CLI;
    2. Enable VRDP support for VM;
    3. Connect to VM by VRDP.
3. \# emerge -av app-emulation/virtualbox

    It reminds accepting PUEL licence.

    Refer to [VirtualBox wiki](https://wiki.gentoo.org/wiki/VirtualBox).
4. \# gpasswd -a username vboxusers

    Add current *username* to *vboxusers* group.

    Logout and re-login to have this command take effect.
5. \# modprobe vboxdrv

    Some optional modules: vboxnetadp vboxnetflt vboxpci. If run *VirtualBox* frequently, add these modules to "*/etc/conf.d/modules*".

    Up to now everyting related to *VirtualBox* itself is prepared. Next is to create VM through CLI.
6. VBoxManage - create VM as *normal* user account

    1. $ VBoxManage list ostypes

        To find the supported OSes. I am going to install *Windows XP 32-bit*. The corresponding *--ostype* is *WindowsXP*.
    2. $ VBoxManage createvm --name WinXP --ostype WindowsXP --basefolder ${HOME}/.config/VirtualBox/Machines --register

        *--name* should be enclosed by double quotes if containing white spaces.

        *--basefoler* is to specify the VM related files location. If not set, it will default to *${HOME}/.VirtualBox/Machines*, which violate the new configuration file location. Refer to [XDG Base Directory Specification](http://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html).

        *--register* to register the VM instantly, or run *VBoxManage registervm* afterwards. Attention: *registervm* only accepts *full* path.
    3. $ VBoxManage modifyvm WinXP --memory 512 --acpi on --nic1 nat --audio alsa --audiocontroller ac97 --vrde on --vrdeaddress 127.0.0.1 --vrdeport 5000,5010-5012 --clipboard bidirectional

        *--vrde on* is to enable VRDP support thus I can connect to the VM GUI by RDP client.

        *--vrdeaddress* set to 127.0.0.1 loopback address. If unset, it defaults to 0.0.0.0 which means other hosts on the network can connect to this virtual machine too. Refer to [127.0.0.1 vs 0.0.0.0](http://fangxiang.tk/2015/09/14/0000-127001-localhost/).

        *--draganddrop* option is useful if you need it. However, it is vunerable to security issue.
    4. $ VBoxManage storagectl WinXP --name "IDE Controller" --add ide --controller PIIX4

        Set disk controller for VM. Don't use SATA related controller for *WindowsXP*.
    5. $ VBoxManage storageattach WinXP --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium "/media/Misc/VMs/WinXP/WinXP.vdi"

        *WinXP.vdi* is copied from some guy in QQ group. So I don't need a installation ISO to install VM from scratch. For ISO from scratch, refer those references listed.
    6. $ VBoxManage storageattach WinXP --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium /usr/share/virtualbox/VBoxGuestAdditions.iso

        *VBoxGuestAddiontions.iso* support file sharing, mouse switch etc between *host* and *guest*. We will install this tool after entering *Windows XP 32-bit* VM.

        Up to now, the VM is created! But *VirtualBox* does not have QT GUI. We need RDP client, i.e. FreeRDP.
7. \# emerge -av net-misc/freerdp
8. Magic

    ```bash
    $ VBoxHeadless --startvm WinXP --vrdeproperty "TCP/Ports=5001,5010-5012"
    ```
    *VBoxHeadless* is a tool the launch VM as a server mode instead of traditional GUI mode. If you guy a VPS, say from Linode, your VPS VM mostly runs as a similar mode (maybe through web protocol for your control in brower).

    The RDP TCP port is set to an optional list *5001,5010-5012*. If not set, the default is *3389*. Usually, *5001* is not occupied in system, so most of the time, it is chosen as the port to listen for RDP connection.

    Now the VM is started, but not showing up! We need to use FreeRDP to get X.
9. $ xfreerdp +clipboard /v:127.0.0.1:5001

    Attention: the server IP is the IP address of *host*, NOT IP of *guest*. Since I connect to VM locally, so it *127.0.0.1*.

    *xfreerdp* can add many parameters except *clipboard*, like window size, audio, video etc.

    Now get into *Windows XP 32-bit*. If cannot see the Windows start menu, just enter 'Ctrl + Alt + Enter'.

    You might found there are two mouse pointers, one for *host* while another for *guest*. Also the screen resolution is not correctly set. To solve issues like this, to install *VBoxGuestAdditions*. Open file explorer, installer is located in partition *(D:) VirtualBox Guest Additions* -> *VBoxWindowsAdditions.exe*. Before issues got solved, you can use keyboard shortcuts and TAB, ENTER etc keys.

    During *VBoxGuestAddtions* installation, there is a option *Direct3D* which should NOT be enabled.
10. $ VBoxManage controlvm WinXP savestate/acpipowerbutton/poweroff/pause

    *pause*: temporarily puts a virtual machine on hold, without changing its state for good. The VM window will be painted in gray to indicate that the VM is currently paused. 

    *poweroff*: has the same effect on a virtual machine as pulling the power cable on a real computer. Again, the state of the VM is not saved beforehand, and data may be lost. (This is equivalent to selecting the "Close" item in the "Machine" menu of the GUI or pressing the window's close button, and then selecting "Power off the machine" in the dialog.)

    *savestate*: will save the current state of the VM to disk and then stop the VM. (This is equivalent to selecting the "Close" item in the "Machine" menu of the GUI or pressing the window's close button, and then selecting "Save the machine state" in the dialog.) We can think *savestate* as suspend to disk (*hibernate*).

    *acpipowerbuttion*: just like you press the PC's power buttion to shutdown directly without saving.

    1. If some jobs in VM not yet finished (like editing a file), use *savestate*;
    2. If every jobs completed, use *acpipoweroff* (as long as ACPI is enabled for VM) or *poweroff*;
    3. If still want to leave the VM running remotely, just close the RDP window.
11. $ VBoxManage storageattach WinXP --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium emptydrive

    Since *VBoxGuestAddtions* is installed. So unmount this ISO file. Other ISO files can also be mounted like step *6.6*.
12. Folder share

    First, set the folder that will be shared in *host*. Then in *guest*, mount the shared folder.
    1. $ VBoxManage sharedfolder add WinXP --name "WLshare" --hostpath "/media/misc/VMs/WLshare"
    2. Open Windows Explorer and look for it under "My Networking Places" -> "Entire Network" -> "VirtualBox Shared Folders" -> "\\\Vboxsvr". By right-clicking on a shared folder and selecting "Map network drive" from the menu that pops up, you can assign a drive letter to that shared folder.

        If don't assign a drive letter, each time to access the shared, we have to find it under "\\\Vboxsvr".
12. Launch scripts.
    1. \# ect /usr/local/sbin/vboxmodule

        ```bash
        #!/bin/bash
        for m in vbox{drv,netadp,netflt}; do modprobe $m; done
        echo 'VirtualBox modules loaded!'
        ```
    2. \# ect ${HOME}/bin/vboxWinXP

        ```bash
        #!/bin/bash
        # Bash Menu Script Example

        PS3='Please enter your choice on VM WinXP: '
        options=("startvm" "rdp" "poweroff" "acpipowerbutton" "savestate" "quit")
        select opt in "${options[@]}"
        do
            case $opt in
                "startvm")
                    echo "you choose to launch WinXP"
                    VBoxHeadless --startvm WinXP --vrdeproperty "TCP/Ports=5001,5010-5012"
                    break
                    ;;
                "rdp")
                    echo "you choose to connect WinXP"
                    xfreerdp +clipboard /sound /f /v:127.0.0.1:5001
                    break
                    ;;
                "acpipowerbutton")
                    echo "you chose to acpipowerbuttion WinXP"
                    VBoxManage controlvm WinXP poweroff
                    break
                    ;;
                "savestate")
                    echo "you chose to save WinXPstate"
                    VBoxManage controlvm WinXP savestate
                    break
                    ;;
                "poweroff")
                    echo "you chose to poweroff WinXP"
                    VBoxManage controlvm WinXP poweroff
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
    5. My Computer -> Properties -> Advanced -> Performance -> Settings -> Adjust for best performance.
13. Modules rebuild for new kernel. Read the first reference on *Kernel driver not installed* section.

    After booting with new kernel (by kernel upgrading), you could no longer load modules like *vobxdrv*.

    ```bash
    # modprobe vboxdrv
    modprobe: FATAL: Module vboxdrv not found.
    ```
    The solution is to reinstall *VirtualBox* external modules `emerge -av app-emulation/virtualbox-modules`. More read [Upgrade kernel to unstable 4.0.0](http://www.fangxiang.tk/2015/03/25/gentoo-installation/).
14. References
    1. [gentoo wiki](https://wiki.gentoo.org/wiki/VirtualBox)
    2. http://baige5117.github.io/blog/install_virtualbox_in_gentoo.html
    3. https://github.com/rustymyers/scripts/blob/master/shell/createVBoxVM.sh
    4. http://serverfault.com/questions/171665/how-to-attach-a-virtual-hard-disk-using-vboxmanage
    5. http://www.virtualbox.org/manual/ch07.html#idp46785385220400
    6. https://www.howtoforge.com/vboxheadless-running-virtual-machines-with-virtualbox-4.3-on-a-headless-ubuntu-14.04-lts-server
    7. https://forums.virtualbox.org/viewtopic.php?f=7&t=18012
    8. http://askaralikhan.blogspot.com/2011/01/virtualbox-40-creating-virtual-machine.html
