---
style: post
title: VirtualBox
---
This post indroduces installing *VirtualBox* in *Gentoo host*, and then create a *Windows XP 32-bit* virtual machine.

1. *VirtualBox* depends on QT as GUI. But I would like a XFCE4 desktop without any QT packages. QT related USE flags are disabled in Gentoo *make.conf*.

    So I will emerge *VirtualBox* without GUI. To manage VMs, resort to CLI.
2. # echo "app-emulation/virtualbox extensions" > /etc/portage/package.use/virtualbox

    *extension* USE flag will draw in binary *VirtualBox Extension Pack* which is not open source package *app-emulation/virtualbox-extpack-oracle*.

    Why do I need this extension binary package? Mostly, it supports:

    1. Virtual USB 2.0 (EHCI) device;
    2. VirtualBox Remote Desktop Protocol (VRDP) support.
    Refer to [Installing VirtualBox and extension packs](https://www.virtualbox.org/manual/ch01.html#intro-installing).

    *Windows XP 32bit* VM won't even start without QT GUI support. But VRDP helps! VRDP is a replacement of QT GUI!

    1. First create VM with *VirtualBox* CLI;
    2. Enable VRDP support for VM;
    3. Connect to VM by VRDP.
3. # emerge -av app-emulation/virtualbox

    It reminds accepting PUEL licence.

    Refer to [VirtualBox wiki](https://wiki.gentoo.org/wiki/VirtualBox).
4. # gpasswd -a <user> vboxusers

    Add current user to *vboxusers* group.

    Logout and re-login to have this command take effect.
5. # modprobe vboxdrv

    Some optional modules: vboxnetadp vboxnetflt vboxpci. If run *VirtualBox* frequently, add these modules to "*/etc/conf.d/modules*".

    Up to now everyting related to *VirtualBox* itself is prepared. Next is to create VM through CLI.
6. VBoxManage - create VM as *normal* user account

    1. $ VBoxManage list ostypes

        To find the supported OSes. I am going to install *Windows XP 32-bit*. The corresponding *--ostype* is *WindowsXP*.
    2. $ VBoxManage createvm --name WinXP --ostype WindowsXP --basefolder ${HOME}/.config/VirtualBox/Machines --register

        *--name* should be enclosed by double quotes if containing white spaces.
	
        *--basefoler* is to specify the VM related files location. If not set, it will default to *${HOME}/.VirtualBox/Machines*, which violate the new configuration file location. Refer to [XDG Base Directory Specification](http://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html).

        *--register* to register the VM instantly, or run *VBoxManage registervm* afterwards. Attention: *registervm* only accepts *full* path.
    3. $ VBoxManage modifyvm WinXP --memory 1024 --acpi on --nic1 nat --vrde on

        *--vrde on* is to enable VRDP support thus I can connect to the VM GUI by RDP client.
    4. $ VBoxManage storagectl WinXP --name "IDE Controller" --add ide --controller PIIX4

        Set disk controller for VM. Don't use SATA related controller for *WindowsXP*.
    5. $ VBoxManage storageattach WinXP --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium "/media/Misc/VMs/WinXP/WinXP.vdi"

        *WinXP.vdi* is copied from some guy in QQ group. So I don't need a installation ISO to install VM from cratch. For ISO from cratch, refer those references listed.
    6. $ VBoxManage storageattach WinXP --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium /usr/share/virtualbox/VBoxGuestAdditions.iso

        *VBoxGuestAddiontions.iso* support file sharing etc between *host* and *guest*. We will install this tool after entering *Windows XP 32-bit* VM.

        Up to now, the VM is created! But *VirtualBox* does not have QT GUI. We need RDP client, i.e. FreeRDP.
7. # emerge -av net-misc/freerdp
8. Magic

    ```bash
    \$ VBoxHeadless --startvm WinXP --vrdeproperty "TCP/Ports=5001"
    ```
    *VBoxHeadless* is a tool the launch VM as a server mode instead of traditional GUI mode. If you guy a VPS, say from Linode, you probably control VPS VM through similiar tool.

    The RDP TCP port is set to *5001*. If not set, the default is *3389*.

    Now the VM is started, but not showing up! We need to use FreeRDP to get X.
9. $ xfreerdp +clipboard /v:127.0.0.1:5001

    Attention: the server IP is the IP address of *host*, NOT IP of *guest*. Since I connect to VM locally, so it *127.0.0.1*.

    *xfreerdp* can add many parameters except *clipboard*, like window size, audio, video etc.

    Now get into *Windows XP 32-bit*. The very first thing is to install "VBoxGuestAdditions*. Open file explorer, click on the DVD device and find the installer there *xfreerdp +clipboard /v:127.0.0.1:5001
10. $ VBoxManage controlvm WinXP savestate/acpipowerbutton/poweroff/pause

    *pause*: temporarily puts a virtual machine on hold, without changing its state for good. The VM window will be painted in gray to indicate that the VM is currently paused. 

    *poweroff*: has the same effect on a virtual machine as pulling the power cable on a real computer. Again, the state of the VM is not saved beforehand, and data may be lost. (This is equivalent to selecting the "Close" item in the "Machine" menu of the GUI or pressing the window's close button, and then selecting "Power off the machine" in the dialog.)

    *savestate*: will save the current state of the VM to disk and then stop the VM. (This is equivalent to selecting the "Close" item in the "Machine" menu of the GUI or pressing the window's close button, and then selecting "Save the machine state" in the dialog.) If the VM is *savestate*, then you cannot *modifyvm* any more. The trick is to start it and *poweroff*.
11. $ VBoxManage storageattach WinXP --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium emptydrive

    Since *VBoxGuestAddtions* is installed. So unmount this ISO file.
12. Launch VM again.
13. Refs:
    1. https://wiki.gentoo.org/wiki/VirtualBox#Configuration
    2. http://baige5117.github.io/blog/install_virtualbox_in_gentoo.html
    3. https://github.com/rustymyers/scripts/blob/master/shell/createVBoxVM.sh
    4. http://serverfault.com/questions/171665/how-to-attach-a-virtual-hard-disk-using-vboxmanage
    5. http://www.virtualbox.org/manual/ch07.html#idp46785385220400
    6. https://www.howtoforge.com/vboxheadless-running-virtual-machines-with-virtualbox-4.3-on-a-headless-ubuntu-14.04-lts-server
    7. https://forums.virtualbox.org/viewtopic.php?f=7&t=18012
    8. http://askaralikhan.blogspot.com/2011/01/virtualbox-40-creating-virtual-machine.html
