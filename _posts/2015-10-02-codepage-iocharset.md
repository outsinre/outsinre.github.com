---
layout: post
title: codepage iocharset
---

# 1 Introduction

In Linux, X screen or console is usually garbled with squares and unknown characters. The display is in a mess with garbage message of either filename or file contents, especially for displaying Windows FAT filesystem. In this post, I will touch on several technologies related to garbled screen. And I also try to elaborate the process of storage, transfer, and display characters in computer.

There are mainly four parts to be clarified, namely *Character*, *File*, *Font* and *Locale*. Character refers to the basics of language of personal interest, including *Character set* and *Character encoding*. *File* involves *filesystem* and *File Content*, while *Font* requires support of *fontconfig* and *locale*.

Dispalying Chinese filename and contents of FAT filesystem within Linux is troublesome in that it involves *locale*, *kernel*, *userspace program*, *filesystem*, *X* etc. But in this post, I emphasize displaying FAT filename in Linux.

# 2 Character

Windows/Linux kernel (display FAT NTFS filename) or application (read/write file content) must understand both the *character set* and *character encoding*. We usually take *character encoding* and *character set* for each other without clarification, for they always assume each other.

## 2.1 Character Set

1. 字符/语言的存储表达。目前的电子计算设备只能存储和运算数字，很显然，对于非数字信息，必须有一个“转义”过程，而这个“转义”规则本身并不能完全被存储，一部分需要人工设置。我们讨论的主要对象是“字符集”，如中文字符集，英文字符集，日文字符集等。
2. 字符集 “character set”。当我需要使用（存储、传输、显示等）字符时（如你要浏览俄文网页），首先规定一个字符范围（一般是本国语言涉及到的字符），范围内的字符的使用规则都有确切的定义，这个范围就是字符集。字符集中的每个字符将使用一个唯一的数字指代，叫“code point”。字符集其实是一张字符和数字间的逻辑映射表。

    这里我要对“逻辑”二字解释一下。计算机设计中，我们通常需要用到逻辑的概念，譬如逻辑内存，逻辑地址空间，表示虚拟的、脱离实际计算机设备的概念。字符集就是逻辑概念，只对计算机设计者有意义，对计算机本身来说z这张表格没有任何意义。这张字符集表格可以写在纸张上，譬如第0项是“你”，第1024项是“好”，计算机是理解不了这个表的。

    Microsoft designs its own *character set* standard to render different lanauge characters, which is called *codepage*. The *cp437* is for United States and Canada English characters, while the *cp936* is for Chinese characters.

3. 字符集的定义在早期是各自为战的状态，国家、组织等某个地区的权威为各自区域/国家范围内使用到的字符定义字符集。当你的计算机系统只用到本区域/国家的字符时，并没有什么问题。

    但当你需要使用其它区域/国家字符集的字符时，就会出问题，因为不同的字符集可能有“code point”冲突。日本人的日文字符集里，可能数字100是日文中的“他”；泰国的给泰文字符集里数字100则指泰文中“好”。很显然使用日文字符集的计算机上无法存储、传输、显示泰文的“好”字。
4. 为了能让计算机同时使用各区域/国家的语言/字符，有必要设计一个包含尽可能多字符的字符集，对全世界的所有语言字符设置一个统一字符集，这个字符集表肯定非常大，就是我们耳熟能详的“Unicode”字符集。
5. 同一语言，可以有不同的字符集。譬如中文可以有 Windows “cp936”字符集，你自己可以设计一个，只要实力足够说法其它的软件和操作系统使用你设置的字符集表。

## 2.2 Character Encoding

1. With just a table of *character set*, computer can NOT handle (understand, store, transfer, display etc.) characters. Before that, *code point* should be encoded into computer *binary digit*, which is the responsibility of *character encoding*. There are over 20,000 Chinese characters and most of them takes at least two bytes to represent *code point* no matter by *Unicode* or *cp936*. How to store the two bytes, i.e. Unicode 6E49? With *big endian* (6E first) or *little endian* (49 first)? Aside from storage, the transfer order of the two bytes online counts when the network MTU is limited.

    Does computer use *fixed* or *varied* width/bit to handle *code point*? If fixed width (say 16 bits), then code point 128 and 1024 are recoginized by computer as 0x0080 and 0x0400 respectively. If varied width, code point 128 and 1024 might be 0x80 and 0x400.

    In varied width encoding, when encountering two bytes, how should computer treat them? Treat them as a single code point or two sequencial code points?
2. That is where *chracter encoding* comes into effect. *encoding* takes care of representing *human-being code point* as *computer binary digit*. For a programmer, ASCII might be the most famous *character encoding* method, others are ISO 8859, UTF-8, GBK etc. By the way, character set Unicode itself is also a *character encoding*.

    *code point* is designed by human beings and a *logical* table which cannot be recognized by computers. In order to let computer handle *code point*, it should be *encoded* to *binary digit* recognizable by computers, which is a process of converting *logical* table to *binary digit* table.

    Chinese "严"'s Unicode code point is 4E25, but its UTF-8 encoding is E4B8A5. So *character encoding* does NOT guarantee the encoded binary digit equals to original *code point*.
3. A *character set* might have several different *character encoding* methods. For example, *cp936* can use one of GB2312, GBK and GB18030. However, reversely a *character encoding* assumes a specified/fixed *character set*.

    For example, if a file is stored on disk as ASCII, the *character set* must be *cp437* on Windows. GB2312 assumes *cp20936*, GBK assumes *cp936* while GB18030 assumes *cp54936*. GB18030 is a varied width encoding method with some characters encoded into 4 bytes. However Windows *code page* only support single-byte or doulbe-byte encoding, *cp54936* actually does NOT work.
4.Though a language characters can be represented by different *character sets* and *character encodings*, most often, those different sets and/or encodings are *downward compatible* (ASCII -> GB2312 -> GBK -> GB18030). By the way, GBK is not 国家标准.
5. 通常看到网页上提到“内码”，这个内码就是内核处理字符的“character set”。早期 Windows 使用的是“codepage”，自从 Windows 2000 后，Windows 内核默认使用 Unicode，但依然支持那些依然依赖 codepage 编码的应用。Current Windows kernel uses UTF-16 character encoding method (Unicode character set of course). So applications that takes Unicode character set can run smoothly. If application uses other character set like GBK, then we should set *default character set* for *non-Unicode Programs* in Windows control panel. Linux 内核倒是一直都是 Unicode。
6. When we talk *... is stored/encoded as codepage XXX* (i.e. cp936), we actually refer to a specific encoding of that codepage (i.e. GBK).

# 3 File

## 3.1 filesystem

A filesystem is the methods and data structures that an operating system uses to keep track of files on a disk or partition; that is, the way the files are organized on the disk. The word is also used to refer to a partition or disk that is used to store the files or the type of the filesystem. It only cares about the meta data of indexing, locating, organizing files.

### 3.2 Windows NTFS

NTFS is superior to FAT. Most of the disucssion on FAT below applies to NTFS as well. One difference is that NTFS does not need *codepage* anymore.

### 3.3 Windows FAT

[The FAT filesystem](https://www.win.tue.nl/~aeb/linux/fs/fat/fat.html#toc1).

The traditional DOS filesystem types are FAT12 and FAT16. Here FAT stands for *File Allocation Table*: the disk is divided into *clusters*, the *unit* used by the file allocation, and each FAT entry describes which clusters are used by which files. The number of sectors per cluster is given in the *boot sector* byte 13.

#### 3.3.3.1 FAT disk Layout

1. First the boot sector (at relative address 0), and possibly other stuff. Together these are the Reserved Sectors. Usually the boot sector is the only reserved sector.
2. Then the FAT entries (following the reserved sectors; the number of reserved sectors is given in the boot sector, bytes 14-15; the length of a sector is found in the boot sector, bytes 11-12). The File Allocation Table has one entry per cluster. This entry uses 12, 16 or 28 bits for FAT12, FAT16 and FAT32.
3. Then the Root Directory (following the FATs; the number of FATs is given in the boot sector, byte 16; each FAT entry has a number of sectors given in the boot sector, bytes 22-23).
4. Finally the Data Area (following the root directory; the number of root directory entries is given in the boot sector, bytes 17-18, and each directory entry takes 32 bytes; space is rounded up to entire sectors). 

#### 3.3.3.2 12 16 VFAT 32

DOS 1.0 and 2.0 used FAT12. The maximum possible size of a FAT12 filesystem (volume) was 8 MB (4086 clusters of at most 4 sectors each).

DOS 3.0 introduced FAT16. Everything very much like FAT12, only the FAT entries are now 16 bit. Now the maximum volume size was 32 MB, mostly since DOS 3.0 used 16-bit sector numbers. This was fixed in DOS 4.0 that uses 32-bit sector numbers. Now the maximum possible size of a FAT16 volume is 2 GB (65526 clusters of at most 64 sectors each).

In Windows 95 a variation was introduced: VFAT. VFAT (Virtual FAT) is FAT together with long filenames (LFN), that can be up to 255 bytes long. The implementation is an ugly hack. These long filenames are stored in special directory entries. VFAT is not commonly used due to its subsequent FAT32.

FAT32 was introduced in Windows 95 OSR 2 and quickly replaced VFAT on Windows system. It is not supported by Windows NT. Everything very much like FAT16, only the FAT entries are now 32 bits of which the top 4 bits are reserved. The bottom 28 bits have meanings similar to those for FAT12, FAT16. For FAT32: Cluster size used: 4096-32768 bytes. The maximum file size on a FAT32 filesystem is one byte less than 4 GiB (4294967295 bytes).

To be compatible with FAT12 FAT16, when you create a long filename (longer than 8.3) with VFAT, FAT32, or NTFS, the filesystem actually creates two different filenames. One is the actual long filename. This name is visible to Windows 95, Windows 98, and Windows NT (4.0 and later). The second filename is called an MS-DOS® alias. An MS-DOS alias is an abbreviated form of the long filename. The filesystem creates the MS-DOS alias by taking the first six characters of the long filename (not counting spaces), followed by the tilde [~] and a numeric trailer. For example, the filename Brien's Document.txt would have an alias of BRIEN'~1.txt.

Microsoft operating systems use the following rule to distinguish between FAT12, FAT16 and FAT32. First, compute the number of clusters in the data area (by taking the total number of sectors, subtracting the space for reserved sectors, FATs and root directory, and dividing, rounding down, by the number of sectors in a cluster). If the result is less that 4085 we have FAT12. Otherwise, if it is less than 65525 we have FAT16. Otherwise FAT32. 

#### 3.3.3.3 Short/Long filename

*filename* is one of *filesystem* meta data which usually causes garbled screen. FAT12 and FAT16 only support short filename, while VFAT and FAT32 implement both short and long filename. Short filename is case *insensitive*.

Long filename is stored (on VFAT and FAT32 filesystem) with character set Unicode and short filename is stored (on FAT12 FAT16 filesystem) with character sets *codepage* (i.e. cp936). No matter what FAT version is used, short filename is stored as codepage while long filename is stored as Unicode. NTFS stores filename in Unicode despite of short or long filename.

We can even disable the shortname creation on NTFS which will improve system performance. However some old applications still depend on shortname and would not locate the correct filename.

Garbled *filename* with squares and weird characters is when your computer incorrectly renders filesystem filename.

### 3.3.4 Linux FAT support

When manually compiling Linux kernel, there are a few options related to *codepage* and *iocharset* (both are *character set*). These options are mainly support of Windows FAT meta data decoding and conversion, especially for *filename display* on screen. In short, the *codepage* option is for *short* filename, and the *iocharset* option is for *long* filename.

Filesystems with MS-DOS or Windows origin (i.e.: vfat, ntfs, smbfs, cifs, iso9660, udf) need the *iocharset* mount option in order for non-ASCII characters in file names to be interpreted properly. The value of this option should be the same as the character set of your *locale*, adjusted in such a way that the kernel understands it. This works if the relevant character set definition (found under File systems -> Native Language Support) has been compiled into the kernel or built as a module. The *codepage* option is also needed for *vfat* and *smbfs* filesystems. It should be set to the codepage number used under MS-DOS in your country.

Most of the time, we don't care about *codepage* since short filenames are almost dead. Currently, VFAT and FAT32 filesystems support long filenames.

Linux support FAT filesystems by enabling MSDOS and VFAT support in kenrel. Most of the time, we use *-t msdos* to mount FAT12 and FAT16; use `-t vfat` to mound VFAT and FAT32. If we don't use special mount options, *-t* mouint option can be omitted since mount detects filesystem itself. There are three options need special attention: *codepage=xxx*, *iocharset=xxx*, *utf8*. Possible option values should be enabled in kernel and/or in corresponding mount command. We can set default *codepage* and *iocharset* value in kernel, which can be overriden in mount command. For *utf8*, you should explicitly add it to *mount* command or in *fstab* (see below).

#### 3.3.4.1 codepage

In old Windows system, both filesystem meta data and file contents are stored as codepage. But here, we only cares about short filename part.

Set the codepage number for converting to shortname characters on FAT filesystem. By default, `FAT_DEFAULT_CODEPAGE` setting is used. Short filename is encoded by codepage in filesystem meta data and should be decoded to characters before displaying it.

When should we use *codepage*? FAT12 and FAT16 filesystem only support short filename namely `8.3` format with codepage (i.e. cp936) encoding (i.e. GBK). If you mount those filesystems, *cp936* must be enabled in Linux kernel (M or Y) and *codepage=936* be added to mount command such that fileystem meta data can be decoded for updating or displaying.

Say you have a FAT32 parition and mount it with option `-t msdos`, all the filenames are short version. If the filenames are Chinese, you should also append `-o codepage=936`. If you want to display true long filename, use `-t vfat` or just remove `-t` option instead.

If we don't care about short filename (i.e. no application will depend on short filename), just ignore this option. Actually, most of the time, we do omit this option as short filename is almost deprecated. Rare applications depend on it.

#### 3.3.4.2 iocharset

Character set to use for converting between the encoding used for user visible long filename characters (display on screen) and 16 bit Unicode characters (long filename are encoded by Unicode UTF-16). Long filenames are stored on disk in Unicode format (usually UTF-16), but Unix for the most part doesn't know how to deal with Unicode. By default, `FAT_DEFAULT_IOCHARSET` setting is used.

There is also an option of doing UTF-8 translations with the *utf8* option (NOTE: "iocharset=utf8" is not recommended). If unsure, you should consider the *utf8* (similar to but different from *iocharset=utf8*) option instead. If you set *iocharset=utf8* option, Linux VFAT module will be case sensitive which is uncompatible with Windows FAT which does not differentiate filename case. *utf8* alone does not have this side effect.

It should be set to be the corresponding *character encoding* of your Linux system *locale*. If locale is not UTF-8, like GBK or GB2312, set it to *iocharset=cp936*. If locale is UTF-8, set *utf8* or *iocharset=utf8*. As noted above, *ntf8* alone is better than *iocharset=utf8*. This value should NOT be ignored since it controls the final display of FAT filename unless your kernel default setting equals to your system *locale*.

> You may find that *codepage* option is set based on the encoding method of FAT meta data on disk while *iocharset* is set based on Linux *locale*. The value you set for *iocharset* and *codepage* should be enabled as Y or M in kernel.

## 3.4 File Content

After talking about *filesystem* and *filename*, let's turn to *file content*. *filesystem* does NOT care about or know *file content*. The former only takes care of file meta data to organize directory, file location, indexing, etc. by system kernel, while the later are real data of user/userspace application interest.

Operating system kernel is responsible of filesystem support. For file contents, they are owned by user or userspace program (like file editor). Therefore, displaying filename (need kernel support) is different from displaying file content (need userspace program support) on screen (most of the time, it is X in Linux).

For example, we have a GBK-encoded TXT file contents on FAT32 partition and want to display the file, the corresponding program for displaying should support GBK character encoding.

If we use `less` or `cat` command to display the GBK TXT file, the X screen will be garbled, for those commands do not know GBK at all. However, if we use `emacs` or `gedit` to open the file, GBK TXT will be well rendered on X screen.

# 4 Font

## 4.1 Fonts

Fonts is the graphical skeleton of characters to display. The common western fonts are classified into *sans* (like *Courier*), *serif* (like Times New Man) and *monospace*. Mostly used Chinese fonts are "宋体", "楷体", "黑体" etc.

When filename or file content is displayed, a good font design help relieve eye and add aesthetic sense.

## 4.2 Fontconfig

Usually on our system, we have a set of different fonts, like "宋体" for Chinese, "Times" for English, etc. What is more, different fonts may overlap. For example, "宋体" can also be used to display English letters; Google "Nato" can display Chinese, Japanese and Korean. So when system display a character which is covered by several fonts, which font should be chosen? This is when *fontconfig* come into effect. Fontconfig not only chooses the font to display, but also renders the chosen font in a more aesthetic way.

Fontconfig (or *fontconfig*) is a free software program library designed to provide configuration, enumeration and substitution of fonts to other programs (like file editor, web browser etc.).

More on *fontcnfig*, refer to [fontconfig](http://www.fangxiang.tk/2015/04/13/fontconfig/).

# 4.3 Locale

Locale is the final step responsible for displaying characters on screen no matter of filename or file contents. FAT's *iocharset* and NTFS's *nls* should be set to your system locale to display filename. Userspace program must support GBK (through userspace library like `libiconv`) to decode GBK TXT file. And then your *locale* (at least `LC_CTYPE`) should support GBK (like `zh_CN.GBK`) or compatible with GBK (like `zh_CN.UTF-8`).

# Refs

1. [iocharset和codepage有何联系和区别](http://tieba.baidu.com/p/2317422644)
2. [关于mount/samba/字符集的两篇好文](http://zengrong.net/post/1019.htm)
3. [ 字符编码详解](http://www.crifan.com/files/doc/docbook/char_encoding/release/html/char_encoding.html)
4. [字符集编码cp936、ANSI、UNICODE](http://blog.csdn.net/wanghuiqi2008/article/details/8079071)
5. [UTF-8 WiKi](https://wiki.gentoo.org/wiki/UTF-8).
