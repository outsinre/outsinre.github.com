---
layout: post
title: Partitioning and Booting
---

1. toc
{:toc}

# Firmware and Interface

1. Firmware refers to underlying hardware as 'UEFI Firmware' or 'BIOS Firmware'. Nowadays, almost all PCs bring in UEFI Firmware.
2. At the very beginning of booting, we press ESC/F2/F12 and get into 'Firmware interface' as System BIOS, ROM BIOS or PC BIOS which is also named to BIOS Setting.

# Partitioning table

1. GPT
2. MBR (MSDOS)
3. MAC
4. BSD

# Booting scheme

1. Legacy (BIOS) booting
2. UEFI booting.
3. UEFI Firmware supports either UEFI booting or Legacy booting, which is called Compatibility Support Module (CSM).

# Partitioning tool

To create partition table and partitions

1. parted
2. fdisk
3. diskpart on Windows

Both *fdisk* and *parted* are partitioning utilities. *fdisk* is well known, stable, and recommended for the MBR partition layout while *parted* was one of the first Linux block device management utilities to support GPT partitions.

# Filesystem

'Format' refers to *create* filesystem on a disk parition.

1. NTFS
2. FAT 12/16/vfat/32
4. EXT 2/3/4
5. BTRFS

EFI System Partition (ESP) (including bootable USB stick) must be FAT (FAT32 is recommended) filesystem.

# Bootable USB stick

1. *dd* (rude but robust)

   This make USB stick unacessible to OS file manager due to *boot* and/or *esp* flag.
2. Rufus (reliable and versatile tool)
3. *diskpart* (Windows)
4. Ubuntu Disk Creater
5. Manual copy.

   For UEFI booting, just copy (i.e. *rsync*) ISO contents to FAT32 USB partition.

# Operating System Limitations

1. Usually, MBR and BIOS (MBR + BIOS), and GPT and UEFI (GPT + UEFI) go hand in hand without any compatibility issue. However, that matching is a must for Windows and Mac while Linux is more flexible.
3. For [Dual boot with Windows](https://wiki.archlinux.org/index.php/Windows_and_Arch_dual_boot), turn off Windows 'Fast Reboot' and 'Secure boot' in BIOS setting.

   What's more, then stick to the matching rule unless they are installed separately on two disks.
2. UEFI requires the firmware and operating system loader (or kernel) to be size-matched; for example, a 64-bit UEFI firmware implementation can load only a 64-bit operating system (OS) boot loader or kernel. 

# Summary

The final booting scheme depends on:

1. Firmware: UEFI/BIOS Firmware;
2. BIOS setting: uefi/Legacy/CSM booting;
3. Partitioning: GPT/MBR;
4. Operating System (and boot loader thereof).

# Manually creating Windows UEFI bootable USB stick

This tutorial shows how to manually create a Windows 8.1 bootable USB stick under Linux. **gpt does NOT ask for confirmation on each command like fdisk**. On Windows, we use *diskpart*.

1. Create partition table - GPT

   ```bash
   # parted -a optimal /dev/sdb
   (parted) mktable/mklabel gpt
   (parted) unit s
   (parted) print free
   ```

   *mktable/mklabel* **ERASE**s the whole disk data!
2. Create a *primary* partition

   ```
   (parted) mkpart primary fat32 0% 5GiB
   (parted) print free
   (parted) align-check opt 1
   (parted) name 1 Win81USB
   (parted) print free
   ```

   1. The *fat32* argument is optional at this step. It may be specified to set an appropriate partition ID. *parted* does not format filesystem.
   2. *0%* is better for *start* of the very first partition alignment. Similarly, *100%* is better for *end* of the very last partition alignment.
   3. 5 GiB is enough for Windows 8.1 ISO. The remaining space can be used to store data (i.e. create *ext4* partition).
   4. *align-check* checks partition alignment. *optimal/opt* guarantees optimal disk performance. Try to use *unit s* (sector) when *mkpart* for better disk performance.
3. (opt) Set the partition be *bootable*

   ```
   (parted) set 1 boot on
   or
   (parted) toogle 1 esp
   (parted) print free
   (parted) quit
   ```

   1. Enable either *boot* or *esp* as they imply each other.
   2. As mentioned earlier, *boot* and *esp* flag prevents this partition unacessible to OS file manager.
4. Format FAT32 filesystem

   ```bash
   # mkfs.vfat -F32 /dev/sdb1
   ```

   1. Though we sepcify *fat32* argument in *parted*, we must *explicitly* format the partition.
5. Mount ISO file

   ```bash
   # mount -t udf -o loop,ro,unhide /path/to/file.iso /mnt/iso
   ```

   Make usre *udf_fs* is enabled in kernel to support DVD ISO. The default *iso9660* only supports CD ISO.
6. Mount the USB stick

   ```bash
   # mount /dev/sdb1 /mnt/usbstick
   ```
   
7. Copy ISO files to USB stick

   ```bash
   # cp -rv /mnt/iso/* /mnt/usbstick/
   # -or-
   # rsync -rv /mnt/iso/ /mnt/usbstick/
   ```

8. Flush file system buffers

   ```bash
   # sync
   ```

9. Umount ISO and USB stick

   ```bash
   # umount /mnt/iso
   # umount /mnt/usbstick
   ```

1. Refs
   1. [fedora create windows 8.1 USB stick](https://superuser.com/questions/729087/fedora-create-windows-8-1-bootable-usb)
   2. [howtogeek cd/dvd ISO mount](http://www.howtogeek.com/168137/mount-an-iso-image-in-linux/?PageSpeed=noscript)

## Windows 7 Notes

By default, Windows 7 ISO does not prepare the *bootx64.exe* and relevant directories correctly. Use robust tool like Rufus.

For whatever reason you want to create usb stick by copying or *dd*, please get *bootx64.exe* from an existing Windows 7 OS. Alternatively, extract (i.e. by *7zip*) *\1\Windows\Boot\EFI\bootmgfw.efi* from ISO *\sources\install.wim* and rename it to *bootx64.exe*. From within the USB stick, copy *\efi\microsoft\boot* directory to a upper level, namely *\efi\boot* into which we put *bootx64.exe*. Actually, Rufus just creates *\efi\boot* directly instead of copying.
