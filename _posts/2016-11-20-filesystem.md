---
layout: post
title: Filesystem
---

>This post talks about filesystem operations under Linux environment.

# Filesystem

1. Filename is a string encoded and stored in fileystem.
2. Linux kernel first decodes filename stored on disk (more specific, in filesystem on disk), then encodes it again, and finally submit to user space applications (like *ls*, *thuar* etc.).
3. When it comes to reading filenames on disk,
   1. Windows NT kernel reads two bytes at a time. Everything in Windows NT kernel is UCS-2.
   2. Linux kernel reads byte by byte, which means it recoginizes only variable length encoding streams, single byte encoding (i.e. cp437) included.

      The possible encoding streams are listed in Native Language Support (NLS) under File Systems in *make menuconfig*. But be careful, this NLS (in kernel space) is different from that one (*nls* USE and sytem *locale*) in user space.

# FAT

1. FAT is not a separate filesystem, but a common part of the [MSDOS, UMSDOS and VFAT](https://en.wikipedia.org/wiki/FAT_filesystem_and_Linux).
   1. Mount options of FAT applies to MSDOS, UMSDOS and VFAT either.
   2. Attention, they are Linux filesystem drivers, not the original Windows FAT filesystems.
2. You might encounter two confusing options namely *codepage* and *iocharset*. Without proper setting, you would see garbled filenames.
   1. To be simple, *codepage* encodes (on creation) and decodes (on display) *shortname* while *iocharset* should be the system *locale*.
   2. No matter of *codepage* or *iocharset*, it's used by kernel instead of application.
   2. The *codepage* for display should be the same as that of creation, otherwise it's decoded as garbage.
4. Details are a long story as follows:

## shortname and longname

1. Shortname is a concept belongs to MSDOS only.
   1. Looks like *8.3*. The former is at most 8-byte long and the extension occupies 3 bytes.
   2. Case insensitive. Actually only allow upper case characters.
2. Upon VFAT, long filename is supported, abandoning that two limitations.

   In order to be compatible with MSDOS, each filename has a shortname version. We call the original one as longname. That is to say, VFAT stores two filenames in filesystem like:

   >longname; shortname, codepage
3. Shortname is encoded as *codepage* (i.e. 936 for Simplified Chinese), while longname is encoded as Unicode (NOT UTF-8).

   >Actually, most modern fileystems encode filenames as Unicode.

## Mount

> When it comes to *mount*, we talk about *mounting FAT (MSDOS VFAT) under Linux*.

1. *codepage* option. Kernel uses it to decode shortname and then translates it into Unicode. Longname is Unicode by default.
2. *iocharset* option. Kernel uses it to encode the Unicode shortname (MSDOS) or longname (VFAT). Then the encoded stream is passed to use space.

   There is a special *iocharset* value *utf8*. You should set it in a different way (discussed next).
3. Upon receiving the stream, application decodes it with the system locale (*nls* in user space).

## Set the correct value

1. Suppose we have a VFAT fileystem initialized in Windows GBK sysetem. Now it will be mounted and used under Linux.

   Shortname's codepage is cp936 while longname is Unicode.
2. Set *codepage=936* mount option (without prefix *cp*).
   1. If you want to see the shortname, use *-t msdos* instead of *-t vfat*.
   2. Actually, we rarely use MSDOS nowadays. If you don't care, just leave it the default (in kernel).
3. *iocharset=* depends on system locale *xx_YY.ZZ*:
   1. If ZZ is Chinese encoding like GB2312, GBK, GB18030, then set *iocharset=cp936* (with prefix *cp*).
   2. If ZZ is UTF-8, DO NOT set *iocharset=utf8*! Instead, use *utf8* alone while keeping the default *iocharset* value.
4. If *iocharset=utf8*, the kernel *vfat* module allows lower case shortname which conflicts with Windows's tradition.

   The kernel will throws a warning on such a case.

5. Remember the relevant NLS is compiled as Y or M.
6. The default *codepage* and *iocharset* (maybe *utf8*) value can be set in kernel.

# NTFS

1. Shortname has nothing to do with NTFS. There is not such mount option as *codepage*.
2. The *iocharset* option of NTFS has changed to *nls*.
3. Similarly, if *nls=utf8*, you can use *ntf8* alone.

   There does not exist lower/upper case filename issue with *nls=utf8*.