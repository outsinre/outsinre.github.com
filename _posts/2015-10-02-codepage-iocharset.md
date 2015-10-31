---
layout: post
title: codepage iocharset
---

# 1 Introduction

In Linux, X or console is usually garbled with squares and unknown characters. The screen is in a mess with garbage messages of either filename or file contents, especially of Windows MSDOS/FAT/NTFS filesystem. In this post, I will discuss the reason of garbled screen. And I also try to elaborate the process of storage, transfer, and display characters in computer.

There are mainly four parts to be clarified, namely *Character*, *File*, *Font* and *Locale*. Character refers to the basics of language of personal interest, including *Character set* and *Character encoding*. *File* involves *filesystem* and *File Content*, while *Font* requires the support of *fontconfig* and *locale*.

Dispalying Chinese filename and file contents of MSDOS/FAT/NTFS filesystem within Linux is troublesome in that it involves *locale*, *kernel*, *userspace program*, *filesystem*, *X*, *Character* etc.

# 2 Character

Computers themselves do not understand printed text as a human would. For computers, every character of text is represented by a number (more concretely *binary digit*). Traditionally, each set of numbers used to represent alphabets and characters (known as a *coding system*, *encoding*, or *character set*) was limited in size due to limitations in computer hardware.

Windows/Linux kernel (display MSDOS/FAT/NTFS filename) or application (read/write MSDOS/FAT/NTFS file content) must understand both the *character set* and *character encoding*. We usually take *character encoding* and *character set* for each other without explicitly clarification, for they always assume each other. For example, Unicode usualy implies UCS-2 or UTF-16. Keep in mind: *character set* is meaningless withtout *character encoding*.

## 2.1 Character Set

1. 字符/语言的存储表达。目前的电子计算设备只能存储和运算数字，很显然，对于非数字信息，必须有一个“转义”过程，而这个“转义”规则本身并不能完全被存储，一部分需要人工设置。我们讨论的主要对象是“字符集”，如中文字符集，英文字符集，日文字符集等。
2. 字符集 “character set”。当我需要使用（存储、传输、显示等）字符时（如你要浏览俄文网页），首先规定一个字符范围（一般是本国语言涉及到的字符），范围内的字符的使用规则都有确切的定义，这个范围就是字符集。字符集中的每个字符将使用一个唯一的数字指代，叫“code point”。字符集其实是一张字符和数字间的逻辑映射表。

    这里我要对“逻辑”二字解释一下。计算机设计中，我们通常需要用到逻辑的概念，譬如逻辑内存，逻辑地址空间，表示虚拟的、脱离实际计算机设备的概念。字符集就是逻辑概念，只对计算机设计者有意义，对计算机本身来说z这张表格没有任何意义。这张字符集表格可以写在纸张上，譬如第0项是“你”，第1024项是“好”，计算机是理解不了这个表的。

    Microsoft designs its own *character set* standard to render different lanauge characters, which is called *codepage*. The *cp437* is for United States and Canada English characters, while the *cp936* is for Chinese characters.

3. 字符集的定义在早期是各自为战的状态，国家、组织等某个地区的权威为各自区域/国家范围内使用到的字符定义字符集。当你的计算机系统只用到本区域/国家的字符时，并没有什么问题。

    但当你需要使用其它区域/国家字符集的字符时，就会出问题，因为不同的字符集可能有“code point”冲突。日本人的日文字符集里，可能数字100是日文中的“他”；泰国的给泰文字符集里数字100则指泰文中“好”。很显然使用日文字符集的计算机上无法存储、传输、显示泰文的“好”字。

    为了能让计算机同时使用各区域/国家的语言/字符，有必要设计一个包含尽可能多字符的字符集，对全世界的所有语言字符设置一个统一字符集，这个字符集表肯定非常大，就是我们耳熟能详的“Unicode”字符集。
4. 同一语言，可以有不同的字符集。譬如中文可以有 Windows “cp936”字符集，可以有 Unicode 字符集，我们自己可以设计一个，只要实力足够说服软件和操作系统使用你设置的字符集。

## 2.2 Character Encoding

1. With just a table of *character set*, computer can NOT handle (understand, store, transfer, display etc.) characters. Before that, *code point* should be encoded into computer's language - *binary digit*, which is the responsibility of *character encoding*. The simplest way to encode is just use the index/code point of characters in *chatacter set* table. But that is not enough.

    There are over 20,000 Chinese characters and most of their code points take at least two bytes no matter in *Unicode* or *cp936*. How does computer orgnazie the two bytes, i.e. Unicode 6E49? With *big endian* (6E first) or *little endian* (49 first)? Aside, the transfer order of the two bytes online counts when the network MTU is limited.

    Does computer use *fixed* or *varied* width/bits to organize *code point*? If fixed width (say 16 bits), then code point 128 and 1024 are recoginized by computer as 0x0080 and 0x0400 respectively. If varied width, code point 128 and 1024 might be 0x80 and 0x400.

    In varied width encoding, when encountering two bytes, how should computer treat them? Treat them as a single code point or two sequencial code points? Need pattern support.
2. That is where *chracter encoding* comes into effect. *encoding* takes care of representing *human-being code point* as *computer binary digit*. For a programmer, ASCII (7-bit encoding) might be the most famous *character encoding* method, others are ISO 8859 series, UTF-8, GBK etc. By the way, character set Unicode itself is also a *character encoding* (usually implies UCS-2 Or UTF-16).

    *code point* is designed by human beings and a *logical* table which cannot be recognized/understood by computer. It should be *encoded* into *binary digit*, which is a process of converting human language to computer language.

    Chinese "严"'s Unicode code point is 4E25, but its UTF-8 encoding is E4B8A5. So *character encoding* does NOT guarantee the encoded binary digit equals to original *code point*.
3. A *character set* might correspond to different *character encodings*. For example, Unicode can be encoded into UCS-2, UTF-16, UTF-8, UCS-4 etc. However, reversely a *character encoding* determines its *character set*.

    For example, if a file is stored on disk as ASCII, the *character set* must be *cp437* on Windows. GB2312 assumes *cp20936*, GBK assumes *cp936* while GB18030 assumes *cp54936*. GB18030 is a varied width encoding method with some characters encoded into 4 bytes. However Windows *code page* only support single-byte or doulbe-byte encoding, *cp54936* does NOT actually implemented on Windows. It is just a name reservation.
4.Though a language's characters can be represented by different *character sets* and *character encodings*, most often, those different sets and/or encodings are *downward compatible* (ASCII -> GB2312 -> GBK -> GB18030). By the way, GBK is not 国家标准.
5. 通常看到网页上提到“内码”，这个内码就是内核处理字符的“character set”。早期 Windows 使用的是“codepage”，自从 Windows 2000 后，Windows 内核默认使用 Unicode，但依然支持那些依然依赖 codepage 编码的应用。Current Windows kernel uses UTF-16 character encoding method (Unicode character set of course). So applications that takes Unicode character set can run smoothly. If application uses other character set like GBK, then we should set *default character set* for *non-Unicode Programs* in Windows control panel. Linux 内核倒是一直都是 Unicode。

## 2.3 More on Character set/encoding

The most common (or at least the most widely accepted) character set is **ASCII** (American Standard Code for Information Interchange) for modern English. ASCII is strictly seven-bit, meaning that it uses bit patterns representable with seven binary digits, which provides a range of 0 to 127 in decimal.

Although ASCII was enough for communication in modern English, in other European languages that include accented characters, things were not so easy. The ISO 8859 standards were developed to meet these needs. They were backwards compatible with ASCII, but instead of leaving the eighth bit blank, they used it to allow another 127 characters in each encoding.

ISO 8859's limitations soon came to light, and there are currently 15 variants of the ISO 8859 standard (8859-1 through to 8859-15). Outside of the ASCII-compatible byte range of these character sets, there is often conflict between the letters represented by each byte.

To complicate interoperability between character encodings further, Windows-1252 is used in some versions of Microsoft Windows instead for Western European languages. This is a super-set of ISO 8859-1, however it is different in several ways. These variants do all retain ASCII *compatibility*.

### 2.3.1 Unicode UTF-8

Unicode throws away the traditional single-byte limit of character sets. It uses 17 "planes" of 65,536 code points to describe a maximum of 1,114,112 characters. As the first plane, aka. "Basic Multilingual Plane" or BMP, contains almost every character a user will ever need. Many have made the *wrong* assumption that Unicode was a 16-bit character set (earlier Unicode encoding do indeed use 16-bit unit like UCS-2 and UTF-16).

Unicode has been mapped/encoded in many different ways, but the two most common are UTF (Unicode Transformation Format) and UCS (Universal Character Set). A number after UTF indicates the number of bits in one unit, while the number after UCS indicates the number of bytes, like UCS-2. UTF-8 has become the most widespread means for the interchange of Unicode text as a result of its eight-bit clean nature; it is therefore the subject of this document.

When Unicode first started gaining momentum in the software world, multibyte character sets were not well suited to languages like C, which is the base language of most commonly used programs. Even today, some programs are not able to handle UTF-8 properly. Fortunately the majority of programs, especially the common ones, are supported.

# 3 File

## 3.1 filesystem

A filesystem is the methods and data structures that an operating system uses to keep track of files on a disk or partition; that is, the way the files are organized on the disk. The word is also used to refer to a partition or disk that is used to store the files or the type of the filesystem. It only cares about the meta data of indexing, locating, organizing files.

## 3.2 Windows NTFS

NTFS is superior to FAT. Most of the disucssion on FAT below applies to NTFS as well. One difference is that NTFS does not need *codepage* anymore. The reason of omitting codepage is not clear yet. Maybe NTFS uses Unicode for short filename.

## 3.3 Windows FAT

[The FAT filesystem](https://www.win.tue.nl/~aeb/linux/fs/fat/fat.html#toc1).

The traditional DOS filesystem types are MSDOS, FAT12 and FAT16. Here FAT stands for *File Allocation Table*: the disk is divided into *clusters*, the *unit* used by the file allocation, and each FAT entry describes which clusters are used by which files. The number of sectors per cluster is given in the *boot sector* byte 13.

Since MSDOS is nearly dead. I will not go into details.

### 3.3.1 FAT disk Layout

1. First the boot sector (at relative address 0), and possibly other stuff. Together these are the Reserved Sectors. Usually the boot sector is the only reserved sector.
2. Then the FAT entries (following the reserved sectors; the number of reserved sectors is given in the boot sector, bytes 14-15; the length of a sector is found in the boot sector, bytes 11-12). The File Allocation Table has one entry per cluster. This entry uses 12, 16 or 28 bits for FAT12, FAT16 and FAT32.
3. Then the Root Directory (following the FATs; the number of FATs is given in the boot sector, byte 16; each FAT entry has a number of sectors given in the boot sector, bytes 22-23).
4. Finally the Data Area (following the root directory; the number of root directory entries is given in the boot sector, bytes 17-18, and each directory entry takes 32 bytes; space is rounded up to entire sectors). 

### 3.3.2 12 16 VFAT 32

DOS 1.0 and 2.0 used FAT12. The maximum possible size of a FAT12 filesystem (volume) was 8 MB (4086 clusters of at most 4 sectors each).

DOS 3.0 introduced FAT16. Everything very much like FAT12, only the FAT entries are now 16 bit. Now the maximum volume size was 32 MB, mostly since DOS 3.0 used 16-bit sector numbers. This was fixed in DOS 4.0 that uses 32-bit sector numbers. Now the maximum possible size of a FAT16 volume is 2 GB (65526 clusters of at most 64 sectors each).

In Windows 95 a variation was introduced: VFAT. VFAT (Virtual FAT) is FAT together with long filenames (LFN), that can be up to 255 bytes long. The implementation is an ugly hack. These long filenames are stored in special directory entries. VFAT is not commonly used due to its subsequent FAT32.

FAT32 was introduced in Windows 95 OSR 2 and quickly replaced VFAT on Windows system. It is not supported by Windows NT. Everything very much like FAT16, only the FAT entries are now 32 bits of which the top 4 bits are reserved. The bottom 28 bits have meanings similar to those for FAT12, FAT16. For FAT32: Cluster size used: 4096-32768 bytes. The maximum file size on a FAT32 filesystem is one byte less than 4 GiB (4294967295 bytes).

To be compatible with FAT12 FAT16, when you create a long filename (longer than 8.3) with VFAT, FAT32, or NTFS, the filesystem actually creates two different filenames. One is the actual long filename. This name is visible to Windows 95, Windows 98, and Windows NT (4.0 and later). The second filename is called an MS-DOS® alias. An MS-DOS alias is an abbreviated form of the long filename. The filesystem creates the MS-DOS alias by taking the first six characters of the long filename (not counting spaces), followed by the tilde [~] and a numeric trailer. For example, the filename Brien's Document.txt would have an alias of BRIEN'~1.txt.

Microsoft operating systems use the following rule to distinguish between FAT12, FAT16 and FAT32. First, compute the number of clusters in the data area (by taking the total number of sectors, subtracting the space for reserved sectors, FATs and root directory, and dividing, rounding down, by the number of sectors in a cluster). If the result is less that 4085 we have FAT12. Otherwise, if it is less than 65525 we have FAT16. Otherwise FAT32. 

### 3.3.3 Short/Long filename

*filename* is one kind of *filesystem* meta data which usually causes garbled screen. FAT12 and FAT16 only support/implement short filename, while VFAT, FAT32 and NTFS implement both short and long filename. Short filename is case *insensitive*.

No matter what MSDOS/FAT/NTFS version is used, short filename (on VFAT and FAT32 filesystem) is stored with Windows codepage (i.e. cp936) while long filename (on FAT12 and FAT16 filesystem) is stored with Unicode (i.e. UCS-2 and UTF-16) . NTFS stores filename in Unicode despite of short or long filename.

We can even disable the shortname creation on NTFS which will improve system performance. However some old applications still depend on shortname and would not locate the correct filename.

Garbled *filename* with squares and weird characters is when your computer incorrectly display filesystem filename on console or X.

## 3.4 Linux MSDOS/FAT/NTFS support

When we say *Linux supports Windows MSDOS/FAT/NTFS filesystem*, it means MSDOS/FAT/NTFS part is enabled (Y or M) in Linux kernel. Pay attention to *kernel* keyword. If a filesystem is supported by operating system, it is indeed supported by the kernel of that operating system, including mounting the filesystem, updating filesystem meta data (creating/deleting/updating files etc.), and displaying filename on console and X which is the focus in the following discussion.

For short/long filename to be displayed properly under Linux, we need:

1. Enable corresponding MSDOS/FAT/NTFS in kernel.

    ```
    File systems  --->
	<*> FUSE (Filesystem in Userspace) support
        ...
	DOS/FAT/NT Filesystems  --->
	<*> MSDOS fs support
	<*> VFAT (Windows-95) fs support
	(437) Default codepage for FAT
	(iso8859-1) Default iocharset for FAT
	<M> NTFS file system support
	[ ]   NTFS debugging support
	[ ]   NTFS write support 
        ...
    ```
    There is an option *MSDOS fs support* which is almost dead and can only be found on floppy disk. I will focus only FAT/NTFS in the following sections.
2. Enable *codepage* and *iocharset* in kernel.

    Apart from the basic support in step 1, we should enable NLS options (mainly for displaying filename) in Linux kernel configuration menu, namely - *Native Language Support*. NLS will build those *character encoding* into kernel (Y or M). It is important to not become confused. For the most part, the only thing that needs to be done is to build *UTF-8 NLS* support into the kernel, and change the *Default NLS option* to *utf8* if we don't mount Windows MSDOS/FAT or Apple HFS.

    ```
    File Systems -->
      Native Language Support -->
	(utf8) Default NLS Option
	...
	<M>   Simplified Chinese charset (CP936, GB2312)
	...
	## (Also <*> or <M> other character sets that are in use in the system's FAT filesystems or Joilet CD-ROMs.)
	<*> UTF-8 NLS
    ```
    Take MSDOS/FAT/NTFS for example, filename is encoded by codepage (short filename) or Unicode (long filename) and stored on disk. Enabled NLS options (i.e. cp936) are to decode binary digits (i.e. GBK or GB2312) to human readable short/long filename when displaying. The first option is the default NLS used for decoding when mounting filesystem. The above example sets it to *utf8*. You can also find that `NLS_CODEPAGE_CP936` (to decode Chinese short filename) and `NLS_UTF8` (to decode Chinese long filename) are enabled.

    You can notice many NLS options started with `ISO 8859-` of them most are not necessary for Chinese users. Russian users need to enable `NLS_KOI8_R` instead of `NLS_CODEPAGE_CP936`.

    **NLS is designed for not only MS-DOS/Windows filesystem, but other operating system filesystem** like Apple HFS file system that uses MAC codepages. You can also enable those NLS support if needed.
    
3. cjktty patch for Chinese on console.

    In order to display Chinese on console, we need [cjktty patch](http://www.fangxiang.tk/2015/09/18/git-diff-patch/).
4. Correct *mount* command arguments.

    Having enabled MSDOS/FAT/NTFS filesystem and corresponding NLS in kernel, we then specify add those NLS to mount command through *codepage*, *iocharset* and/or *utf8* arguments. Suppose a FAT32/NTFS partition with Chinese filename and file contents, they can be mounted in Linux of `zh_CN.GBK` *locale*:

    ```bash
    # mount -t vfat -o codepage=936,iocharset=cp936 /dev/sda5 /mnt/data
    # mount -t ntfs-3g -o utf8 /dev/sdb7 /mnt/misc
    ```
 
    Most of the time, we use *-t msdos* to mount FAT12 and FAT16; use `-t vfat` to mound VFAT and FAT32. If you don't use special mount options (i.e. English language users), *-t* mouint option can be omitted since mount detects filesystem itself. There are three options need special attention: *codepage*, *iocharset*, and *utf8*. Possible argument values must be enabled in kernel. We can set their default values in kernel - *Default codepage for FAT*, *Default iocharset for FAT*, *Default NLS Option* which can be overriden in mount command. *codepage*/*iocharset*/*utf8* arguments support Windows MSDOS/FAT/NTFS short/long filename decoding/conversion. In short, the *codepage* option is for *short* filename, and the *iocharset* option is for *long* filename. The *utf8* option alone is to replace *iocharset=utf8* which is not recommended.

    Filesystems with MS-DOS or Windows origin (i.e.: vfat, ntfs, smbfs, cifs, iso9660, udf) need the *iocharset* mount option in order for non-ASCII characters in file names to be interpreted properly. The value of this option should be the same as the character set of your *locale*, adjusted in such a way that the kernel understands it. This works if the relevant character set definition (found under File systems -> Native Language Support) has been compiled into the kernel or built as a module. The *codepage* option is also needed for *vfat*-compatible (MSDOS/FAT) and *smbfs* filesystems. It should be set to the codepage number used under MS-DOS/Windows in your country.

### 3.4.1 codepage

In old Windows system, both filesystem meta data and file contents are stored as codepage. But in this section, I only talk short filename display under Linux.

NLS *codepage* option sets the codepage number for converting to shortname characters on MSDOS/FAT. By default, *mount* will take `FAT_DEFAULT_CODEPAGE` in Linux kernel. Short filename is encoded by codepage and should be decoded to human readable characters.

When should we use *codepage*? MSDOS/FAT12/FAT16 only support short filename of `8.3` format by codepage (i.e. Chinese cp936) encoding (i.e. GB2312 or GBK). If you mount such partitions, NLS *cp936* must be enabled in Linux kernel (M or Y) and *codepage=936* argument be added to mount command such that the short filename can be displayed..

Say you have a FAT32 parition and mount it with option `-t msdos -o codepage=936`, filenames are displayed short. If you want to display true long filename, use `-t vfat` or just remove `-t` option instead.

Most of the time, we needn't *codepage* mount argument at all since MSDOS/FAT12/FAT15 and applications depending on short filename are almost dead. Currently, VFAT and FAT32 support long filename by Unicode encoding. Actually, we do omit this argument as short filename is almost deprecated.

### 3.4.2 iocharset

Character set used for converting between the encoding used for user visible long filename characters (display on screen) and 16 bit Unicode characters (long filename are encoded by Unicode UTF-16). Long filenames are stored on disk in Unicode format (usually UTF-16), but Unix for the most part doesn't know how to deal with Unicode. By default, `FAT_DEFAULT_IOCHARSET` setting is used. Since only VFAT/FAT32/NTFS support long filename, when mounting MSDOS/FAT12/FAT16, you should not use *iocharset* argument.

This argument should NOT be ignored since it controls the final display of true long filename unless your kernel Default NLS Option (*utf8* in above kernel configuration menu) equals to your system *locale*. It should be set to be the corresponding *character encoding* of your Linux system *locale*. If locale is not UTF-8, like GBK or GB2312, set it to *iocharset=cp936*. If locale is UTF-8, use *utf8*,  *iocharset=utf8*/*nls=utf8*.

The argument of doing UTF-8 translations with just *utf8* (NOTE: "iocharset=utf8" or *nls=utf8* is not recommended) is recommended if unsure. If *iocharset=utf8* mount argument is used, Linux kernel VFAT module will be case sensitive which is incompatible with Windows short filename which does not differentiate filename character case. *utf8* alone does not have this side effect. Attention: the counterpart of NTFS *iocharset* option is now deprecated and renamed as *nls*. Details refer to `man mount`.

> We notice that *codepage* option is decided basically by the language of Windows users while *iocharset* is set based on Linux *locale*. The NLS value set for *iocharset* and *codepage* should be enabled in kernel - Native Language Support.

### 3.4.3 Logics

1. There is a Windows FAT32 partition with Chinese filenames. Each file has two filenames - the short one and the long one.
2. Short filename is encoded by codepage *cp936* / GBK.
3. Long filename is encoded by Unicode / UTF-16.
4. `mount -t vfat -o codepage=936,utf8 /dev/sda1 /mnt/data` on Linux with `zh_CH.UTF-8` locale.
5. Kernel *cp936* module decodes binary digits of short filename to human readable characters.

    But the decoded human readable short filename won't be displayed on screen! Instead, the default true long filename will be displayed.

    If use `-t msdos` option, the default true filename will be the short one.
6. Linux kernel decodes UTF-16 binary digits to Unicode (this happens naturally since Linux kernel uses Unicode naturally). Unicode binary digits are encoded into UTF-8.
7. Encoded UTF-8 binary digits are delivered to X server.

    X server (application) decodes the binary digits based on `zh_CN.UTF-8` *locale* and display the long filename in screen.

## 3.5 File Content

After talking about *filesystem* and *filename*, let's turn to *file content*. *filesystem* does NOT either care about or know *file content*. The former only takes care of file meta data to organize directory, file location, indexing, etc. by system kernel, while the later are real data of user/userspace application interest.

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

# 5 Locale

Locale is the final step responsible for displaying characters on screen no matter of filename or file contents. FAT's *iocharset* and NTFS's *nls* should be set to or the Default NLS Option equals to system locale for filename display. Userspace program must support GBK (through userspace library like `libiconv`) to decode GBK TXT file. And then your *locale* (at least `LC_CTYPE`) should support GBK (like `zh_CN.GBK`) or compatible with GBK (like `zh_CN.UTF-8`).

# 6 Refs

1. [iocharset和codepage有何联系和区别](http://tieba.baidu.com/p/2317422644)
2. [关于mount/samba/字符集的两篇好文](http://zengrong.net/post/1019.htm)
3. [ 字符编码详解](http://www.crifan.com/files/doc/docbook/char_encoding/release/html/char_encoding.html)
4. [字符集编码cp936、ANSI、UNICODE](http://blog.csdn.net/wanghuiqi2008/article/details/8079071)
5. [UTF-8 WiKi](https://wiki.gentoo.org/wiki/UTF-8).

# More

1. display on X VS console, different?
