---
layout: post
title: Disk
---

# Factors

1. Firmware boot mode

   BIOS; UEFI (EFI).
2. Partitioning style
3. Filesystem

# Disk tool

1. parted
2. fdisk
3. diskpart

> create partition table and partitions

# Partition table

1. GPT
2. MBR/MSDOS
3. MAC
4. BSD

# Format filesystem on partitions

1. NTFS
2. FAT(16)
3. FAT32
4. EXT2/3/4
5. BTRFS

# Bootable USB stick tool

1. *dd* (rude but robust)
2. Rufus (reliable and versatile tool)
3. UNetbootin
4. Universal USB Installer
5. *diskpart* (Windows)
6. Ubuntu disk creater

# UEFI

Read [Dual boot with Windows](https://wiki.archlinux.org/index.php/Windows_and_Arch_dual_boot). Windows 8/8.1 AMD64 supports booting in AMD64 UEFI mode from GPT disk only, OR in BIOS mode from MBR/msdos disk only. They do not support IA32 UEFI boot, AMD64 UEFI boot from MBR/msdos disk, or BIOS boot from GPT disk.

1. GPT and UEFI; MBR/MSDOS and legacy BIOS.
2. Disable *legacy* BIOS boot. Only allow UEFI for both Windows and Linux in BIOS setup.
3. Main hard drive should adopt GPT scheme instead of MBR/MSDOS.
4. Windows partition uses NTFS while Linux mainly uses EXT4.
5. EFI partition must be FAT32/VFAT.
6. UEFI requires 64-bit OS. 32-bit version is unacceptable.
7. Bootable USB stick may adopts GPT.

   If you receive such error during installing Windows:

   >We couldn't create a new partition or locate an existing one.  For more information, see the Setup log files.

   then [create MBR/MSDOS disk table instead](https://wiki.archlinux.org/index.php/Windows_and_Arch_dual_boot#Couldn.27t_create_a_new_partition_or_locate_an_existing_one).
8. USB stick partition must be FAT32.

   If you really have good reason to use NTFS, then resort to Rufus (built-in [UEFI bootable NTFS](https://github.com/pbatard/uefi-ntfs) support).

## Notes

1. Don't use *dd* command for Windows 7 x64 UEFI bootable usb stick. By default, Windows 7 ISO does not prepare the *bootx64.exe* and relevant directories correctly.

   Use robust tool like Rufus. For whatever reason you want to create usb stick manually, get a *bootx64.exe* copy from an existing Windows 7 OS. Alternatively, extract (i.e. by *7zip*) *\1\Windows\Boot\EFI\bootmgfw.efi* from ISO *\sources\install.wim* and rename it to *bootx64.exe*.

   From within the USB stick, copy *\efi\microsoft\boot* directory to a upper level, namely *\efi\boot* where we put *bootx64.exe*.

   Actually, Rufus just creates *\efi\boot* directly instead of copying.
2. Windows does not care about file/directory name letter case.

## Manual USB stick

>Rule: just copy the ISO contents into a FAT32 USB stick

This tutorial shows how to manually create a Windows 8.1 bootable USB stick under Linux. **gpt does NOT ask for confirmation on each command like fdisk**.

1. Create partition table - GPT

   ```bash
   # parted -a optimal /dev/sdb
   (parted) mktable/mklabel gpt
   (parted) unit s
   (parted) print free
   ```

   *mktable/mklabel* **erases** the whole disk data!
2. Create a *primary* partition

   ```
   (parted) mkpart primary fat32 0% 5GiB
   (parted) print free
   (parted) align-check opt 1
   (parted) name 1 Win81USB
   (parted) print free
   ```

   1. The *fat32* argument is optional at this step. It may be specified to set an appropriate partition ID.
   2. *0%* is better for *start* of the very first partition alignment. Similarly, *100%* is better for *end* of the very last partition alignment.
   3. *5GiB* is enough for Windows 8.1 ISO. The remaining space can be used to store data (i.e. create *ext4* partition).
   4. *align-check* checks partition alignment. *optimal/opt* guarantees optimal disk performance. Try to use *unit s* (sector) when *mkpart* for better disk performance.
3. Set/toggle *boot* flag

   ```
   (parted) set 1 boot on
   or
   (parted) toogle 1 boot
   (parted) print free
   (parted) quit
   ```

   1. Set the partition be *bootable*.
4. Format the partition to FAT32 filesystem

   ```bash
   # mkfs.vfat -F 32 /dev/sdb1
   ```

   1. Though we sepcify `fat32` argument when creating partition above, we MUST *explicitly* format the partition.
5. Mount ISO file

   ```bash
   # mount -t udf -o loop,ro,unhide /path/to/file.iso /mnt/iso
   ```

   Make usre *udf_fs* is enabled in kernel to support DVD ISO. The default *iso9660* only supports CD ISO.
6. Mount the USB stick

   ```bash
   # mount /dev/sdb1 /mnt/usbstick
   ```
   
6. Copy ISO files to USB stick

   ```bash
   # cp -rv /mnt/iso/* /mnt/usbstick
   ```

7. Flush file system buffers

   ```bash
   # sync
   ```

8. Umount ISO and USB stick

   ```bash
   # umount /mnt/iso
   # umount /mnt/usbstick
   ```

9. Refs
   1. [fedora create windows 8.1 USB stick](https://superuser.com/questions/729087/fedora-create-windows-8-1-bootable-usb)
   2. [howtogeek cd/dvd ISO mount](http://www.howtogeek.com/168137/mount-an-iso-image-in-linux/?PageSpeed=noscript)
