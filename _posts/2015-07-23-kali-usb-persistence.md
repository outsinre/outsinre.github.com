---
layout: post
title: Kali Linux Live USB Persistence
---

This post introduces making a bootable Kali USB stick while making changes persistent.

1. Download [Kali][1].

   >kali-linux-1.1.0a-amd64.iso

   Don't forget to verify *SHA1Sum* befere proceding.
2. Create a bootable Kali Live USB drive. My current working system is *Gentoo*. Refer to [Making a Kali Bootable USB Drive][2].
   1. Make sure USB flash at least 8GB. Plug USB into PC.
   2. \# parted -l

      Find the **correct USB device** name. In my system it is `/dev/sdb`. Though `dd` command is magical, you would lost all the data if you provided the wrong device name to `dd`.
   3. The blocksize parameter can be increased, and while it may speed up the operation of the `dd` command, it can occasionally produce unbootable USB drives, depending on your system and a lot of different factors. The recommended value, `bs=512k`, is conservative and reliable.
   4. Magic:
    
      ```bash
      # dd if=/path/to/kali-linux-1.1.0a-amd64.iso of=/dev/sdb bs=512k
      ```

      You don't need to format Live USB. `dd` will handle it.
   5. Image the USB drive can take a good amount of time, over ten minutes or more is not unusual, as the sample output below shows. Be patient!
   6. After `dd`, use `parted` command to see what happens to your USB flash `/dev/sdb`:

      ```bash
      # parted -a optimal /dev/sdb
      ```

      This is my system output:

	 zhtux ~ # parted -a optimal /dev/sdb                                      
	 GNU Parted 3.2
	 Using /dev/sdb
	 Welcome to GNU Parted! Type 'help' to view a list of commands.
	 (parted) unit MiB                                                         
	 (parted) print free                                                       
	 Model: SanDisk Cruzer Edge (scsi)
	 Disk /dev/sdb: 15267MiB
	 Sector size (logical/physical): 512B/512B
	 Partition Table: msdos
	 Disk Flags: 

	 Number  Start    End       Size      Type     File system  Flags
		 0.02MiB  0.03MiB   0.02MiB            Free Space
	  1      0.03MiB  2858MiB   2858MiB   primary               boot, hidden
	  2      2858MiB  2921MiB   63.0MiB   primary  fat16
		 2921MiB  15267MiB  12346MiB           Free Space
   7. Several points from the output.
      1. `unit MiB` instead `unit MB`. `MiB` shows you the exact value (at Bytes) at the exact disk position, while `MB` might round up and the actual position might be 500KB ahead or 500KB after the `MB` value. Similarly, we have `GiB` and `TiB`. Read `parted` manual and [arch wiki rounding][3].
      2. You can find dd creates two primary partitions *1* (2858MiB, with *boot* flag) and *2* (63.0 MiB). Why `dd` creates two partitions? This is due to *kali-linux-1.1.0a-amd64.iso* itself a copy of two partitions. This is different from `Ubuntu` or `Gentoo` images which has only one partition.

	 ```bash
	 # fdisk -l /path/to/kali-linux-1.1.0a-amd64.iso
	 or
	 # parted /path/to/kali-linux-1.1.0a-amd64.iso print
	 ```

         This is the ouput of from *parted*:

	    zhtux mnt # parted /media/WLShare/kali-linux-1.1.0a-amd64/kali-linux-1.1.0a-amd64.iso print
	    Model:  (file)
	    Disk /media/WLShare/kali-linux-1.1.0a-amd64/kali-linux-1.1.0a-amd64.iso: 3063MB
	    Sector size (logical/physical): 512B/512B
	    Partition Table: msdos
	    Disk Flags: 

	    Number  Start   End     Size    Type     File system  Flags
	     1      32.8kB  2997MB  2997MB  primary               boot, hidden
	     2      2997MB  3063MB  66.1MB  primary  fat16
      3. `dd` is a stupid command only copy bytes by bytes fromm `if` to `of`. We can find there are a lot of free space untouched after partition *2*. We can make use of the reamaining free space. My *Gentoo* Live USB has only one partion consuming the whole flash storage.
3. Adding USB Persistence with `LUKS` Encryption, refer to [Kali persistence USB][4].
   1. *persistence* means changes to Kali system on Live USB remains accross reboots. Basically just create an extra primary partition on Live USB to store persistent files.
   2. Create and format an additional partition on the USB drive. Continue from step 2.6, use `parted` tool to create an `ext3` primary partition from the remaining free space.

      ```bash
      (parted) mkpart primary 2921MiB 100%
      Warning: You requested a partition from 2921MiB to 15267MiB (sectors 5982208..31266815).
      The closest location we can manage is 2921MiB to 15267MiB (sectors 5983104..31266815).
      Is this still acceptable to you?
      Yes/No? Yes                                                               
      Warning: The resulting partition is not properly aligned for best performance.
      Ignore/Cancel?
      ```

      The [Kali Doc][4] recommends *Ignore*. Here I want to try the disk alignment for better performance. Refer to [arch wiki warnings][5]. This alignment means the *start* position is not aligned. The *end* position *100%* will align itself automatically.
   3. Enter *Ignore* to go ahead anyway, print the partition table in sectors to see where it starts, and remove/recreate the partition with the start sector rounded up to increasing powers of 2 until the warning stops.

      I have tried  2^8, 2^9, 2^10, 2^11. Finally, *2^11* works.

      >5983104s % 2048 = 896s; 2048s - 896s = 1152s; 5983104s + 1152s = `5984256s`.

      ```bash
      Warning: The resulting partition is not properly aligned for best performance.
      Ignore/Cancel? Ignore                                                     
      (parted) unit s                                                           
      (parted) print free                                                       
      Model: SanDisk Cruzer Edge (scsi)
      Disk /dev/sdb: 31266816s
      Sector size (logical/physical): 512B/512B
      Partition Table: msdos
      Disk Flags: 

      Number  Start     End        Size       Type     File system  Flags
	      32s       63s        32s                 Free Space
       1      64s       5854015s   5853952s   primary               boot, hidden
       2      5854016s  5983103s   129088s    primary  fat16
       3      5983104s  31266815s  25283712s  primary               lba

      (parted) rm 3                                                             
      (parted) mkpart primary 5983232s 100%                   
      Warning: The resulting partition is not properly aligned for best performance.
      Ignore/Cancel? Cancel                                                     
      (parted) mkpart primary 5984256s 100%                                     
      (parted) print free                                                       
      Model: SanDisk Cruzer Edge (scsi)
      Disk /dev/sdb: 31266816s
      Sector size (logical/physical): 512B/512B
      Partition Table: msdos
      Disk Flags: 

      Number  Start     End        Size       Type     File system  Flags
	      32s       63s        32s                 Free Space
       1      64s       5854015s   5853952s   primary               boot, hidden
       2      5854016s  5983103s   129088s    primary  fat16
	      5983104s  5984255s   1152s               Free Space
       3      5984256s  31266815s  25282560s  primary               lba

      (parted) q                                                                
      Information: You may need to update /etc/fstab.
      ```
      
      From the final output, we can see there is a big free space (1152 * 512B = 576 MB) between partition *2* and *3*. Of course, you can just leave 3rd partition unaligned considering the limited flash storage.
   4. Initialize the `LUKS` encryption on the newly-created partition. You’ll be warned that this will overwrite any data on the partion. When prompted whether you want to proceed, type *YES* (all upper case). Enter your selected passphrase (use *kali* currently) twice when asked to do so.

      ```
      # cryptsetup --verbose --verify-passphrase luksFormat /dev/sdb3
      # cryptsetup luksOpen /dev/sdb3 my_usb
      ```
      
   4. Format the *persistence* partition *3* as `ext4` (the official reference use `ext3`).

      ```bash
      # mkfs.ext4 -L persistence /dev/mapper/my_usb
      # e2label /dev/mapper/my_usb persistence, this step might be optional.
      ```
      
      This might take several minutes. Don't touch the keyboard!
   5. Create a mount point, mount our new encrypted partition there, set up the *persistence.conf* file, and unmount the partition.

      ```bash
      # mkdir -p /mnt/usb
      # mount /dev/mapper/my_usb /mnt/usb
      # echo "/ union" > /mnt/usb/persistence.conf
      # umount /dev/mapper/my_usb
      ```
      
   6. Close the encrypted channel to our persistence partition.

      ```
      # cryptsetup luksClose /dev/mapper/my_usb
      ```

[1]:https://www.kali.org/downloads/
[2]:http://docs.kali.org/downloading/kali-linux-live-usb-install
[3]:https://wiki.archlinux.org/index.php/GNU_Parted#Rounding
[4]:http://docs.kali.org/downloading/kali-linux-live-usb-persistence
[5]:https://wiki.archlinux.org/index.php/GNU_Parted#Warnings
