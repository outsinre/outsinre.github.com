---
layout: post
title: codepage iocharset Linux
---

# Background

In this post, I will try to elaborate concepts related to FAT *codepage* and *iocharset* Linux kernel options, while I will also touch on short/long file names, chracter encoding etc.

Dispalying Chinese file names and contents of FAT system within Linux is troublesome in that it involves different parts, like *locale*, *kernel*, *file system*, *X* etc.

# Character Set

1. 文字/字符/语言的存储表达。目前的电子计算设备只能存储和运算数字，很显然，对于非数字信息，必须有一个“转义”过程，而这个“转义”规则本身并不能完全被存储，一部分是需要人去设置解释的。我们讨论的主要对象是“字符集”，如中文字符集，英文字符集，日文字符集等。
2. 字符集 ”character set"。当我需要使用（存储、传输、显示等）字符时，首先规定一个字符范围（一般是本国语言涉及到的字符），范围内的字符的使用规则都有确切的定义，这个范围就是字符集。字符集中的每个字符将使用一个唯一的数字指代，叫”code point“。字符集其实是一张字符和数字的映射表格，计算设备使用的就是指代字符的数字。

    Microsoft designs its own *character set* standard to represent different lanauge characters, which is called *codepage*. *cp437* is for United States and Canada English characters, while *cp936* is for Chinese characters.

3. 字符集的定义在早期是各自为战的状态，国家、组织等某个地区的权威为各自区域/国家范围内使用到的字符定义字符集。当你的计算机系统只用到本区域/国家的字符时，并没有什么问题。

    但当你需要使用其它区域/国家字符集的字符时，就会出问题，因为不同的字符集可能有“code point”冲突。日本人的日文字符集可能用数字100指代日文中“他”；泰国的给泰文字符集里数字100则可能指代泰文中“好”。很显然使用日文字符集的计算机上无法存储、传输、显示泰文的“好”。
4. 为了能够在计算机上能够同时使用各区域/国家的语言/字符，有必要对全世界的所有语言字符设置一个统一字符集，这个字符集表格肯定非常大，就是我们耳熟能详的“Unicode”字符集。
5. 同一字符集合，可以有不同的字符集。譬如中文可以有 Windows “cp936”字符集，你自己可以设计一个，只要实力足够说法其它的软件和操作系统使用你的中文字符集表。

# Character Encoding

1. With just a table of *character set*, computer can NOT handle (understand, store, transfer, display etc.) characters. There are over 20,000 Chinese characters and most of them takes at least two bytes to represent *code point* no matter by *Unicode* or *cp936*. How to store the two bytes, i.e. Unicode 6E49? With *big endian* (6E first) or *little endian* (49 first)? Aside from storage, the transfer order of the two bytes online counts when the network MTU is limited.

    Does computer use *fixed* or *varied* width/bit to handle *code point*? If fixed width (say 16 bits), then code point 128 and 1024 are recoginized by computer as 0x0080 and 0x0400 respectively. If varied width, code point 128 and 1024 might be 0x80 and 0x400.

    In varied width encoding, when encountering two bytes, how should computer treat them? Treat them as a single code point or two sequencial code points?
2. That is where *chracter encoding* comes into effect. *encoding* takes care of representing *human-being code point* as *computer binary digit*. For a programmer, ASCII might be the most famous *character encoding* method, others are ISO 8859, UTF-8, GBK etc. By the way, character set Unicode itself is also a *character encoding*.

    *code point* is designed by human beings and a *logical* table which cannot be recognized by computers. In order to let computer handle *code point*, it should be *encoded* to *binary digit* recognizable by computers, which is a process of converting *logical* table to *binary digit* table.

    Chinese "严"'s Unicode code point is 4E25, but its UTF-8 encoding is E4B8A5. So *character encoding* does NOT guarantee the encoded binary digit equals to original *code point*.
3. A *character set* might have several different *character encoding* methods. For example, *cp936* can use one of GB2312, GBK and GB18030. However, reversely a *character encoding* assumes a specified/fixed *character set*.

    For example, if a file is stored on disk as ASCII, the *character set* must be *cp437* on Windows. GB2312 assumes *cp20936*, GBK assumes *cp936* while GB18030 assumes *cp54936*. GB18030 is a varied width encoding method with some characters encoded into 4 bytes. However Windows *code page* only support single-byte or doulbe-byte encoding, *cp54936* actually does NOT work.
4.Though a language characters can be represented by different *character sets* and *character encodings*, most often, those different sets and/or encodings are *downward compatible* (ASCII -> GB2312 -> GBK -> GB18030).
5. 通常看到网页上提到“内码”。这个内码就是字符的“code point”被编码后的计算机能理解的二进制数，也就是“character encoding”结果。

# Procedure

The process of a human-being language character to  binary digit (i.e. storing to disk):

1. Query the *character set* to get *code point*.
2. Encode the *code point* to *binary digit* by *character encoding* method.

The reverse process is similar when retrieving characters from disk to display or transfer.

A Linux kernel or application must understand both the *character set* (reference for programmers) and *character encoding* (computer handling). In our daily programming, we usualy only talks about *character encoding* and ignore *character set* since the former assumes the later one.

Windows kernle uses UTF-16 character encoding method (Unicode character set of course). So applications that takes Unicode character set can run smoothly. If application uses other character set like GBK, then we should set *default character set* for *non-Unicode Programs*.

Linux kernel uses UTF-8 character encoding method instead.

Attention, when doing calculation, updating, deleteing, padding etc operation, kernel use pure unicode *code point* instead of *binary digit*.

# FAT filenames in Linux

When manually compiling Linux kernel, there are a few options related to *codepage* and *iocharset*.

These options are mainly support of Windows FAT *filename display* under Linux. Both of them are referring to *character set*. In short, the *codepage* option is for *short* filenames, and the *iocharset* option is for *long* filenames.

Most of the time, we don't care about *codepage* since short filenames are almost dead. Currently, vfat and FAT32 file systems support long filenames.

# FAT

Windows FAT file system developed along with Windows, including FAT12, FAT16, *vfat* and FAT32. We usually call FAT for FAT12 and FAT16. *vfat* is an extension of FAT while FAT32 extends *vfat* in return. Each branch itself developed as well during the years.

FAT12 and early FAT16 only support short filename, while later FAT16 version, *vfat* and FAT32 support both short and long filename. Short filename is case *insensitive*.

Long filename is stored (on *vfat* and FAT32 file system) with character set Unicode (i.e. UTF-8) and short filename is stored (on FAT12 file system) with character set *codepage* (i.e. cp936). 无论是FAT还是*vfat*, FAT32，短文件名按“codepage”编码存储，长文件名按“Unicode”编码存储。

Linux support FAT file systems by enabling *vfat* support. Most of the time, we use *-t vfat* to mount FAT12, FAT16, *vfat8* and FAT32 file system. If we don't use special mount options, *-t vfat* can be omitted. There are three options need special attention: *codepage=xxx*, *iocharset=xxx*, *utf8*. Possible option values should be enabled in kernel and corresponding *mount* command. We can set both *codepage* and *iocharset* default value in kernel. For *utf8*, you should explicitly set it on *mount* command or in *fstab* (see below).

# codepage

Sets the codepage number for converting to shortname characters on FAT filesystem. By default, `FAT_DEFAULT_CODEPAGE` setting is used.

When should we use *codepage*? FAT12 and early FAT16 file system only support short filename namely `8.3` format with codepage (*cp936*) encoding (*GBK*). If you need to mount such kind of FAT partitions, you must enable *cp936* in Linux kernel and add *codepage=936* option for *mount* command.

To be compatible with FAT, when you create a long filename (longer than 8.3) with VFAT or FAT32, the file system actually creates two different filenames. One is the actual long filename. This name is visible to Windows 95, Windows 98, and Windows NT (4.0 and later). The second filename is called an MS-DOS® alias. An MS-DOS alias is an abbreviated form of the long filename. The file system creates the MS-DOS alias by taking the first six characters of the long filename (not counting spaces), followed by the tilde [~] and a numeric trailer. For example, the filename Brien's Document.txt would have an alias of BRIEN'~1.txt.

If we don't care about short filename, just ignore this option. Say you have a FAT32 parition and mount it with option `-t msdos`, all the filenames are short version. If the filenames are Chinese, you should also append `-o codepage=936`. If you want to show long filename, use `-t vfat` or just remove `-t` option instead.

# iocharset

Character set to use for converting between the encoding is used for user visible filename and 16 bit Unicode characters. Long filenames are stored on disk in Unicode format, but Unix for the most part doesn't know how to deal with Unicode. By default, `FAT_DEFAULT_IOCHARSET` setting is used.

There is also an option of doing UTF-8 translations with the *utf8* option. NOTE: "iocharset=utf8" is not recommended. If unsure, you should consider the *utf8* (similar to but different from *iocharset=utf8*) option instead. If you set *iocharset=utf8* option, Linux *vfat* module will be case sensitive which is uncompatible with Windows FAT. *utf8* alone does not have this issue.

This value should NOT be ignored since it controls the final display of FAT filenames.

It should be set to be the corresponding *character encoding* of your Linux locale. If locale is not UTF-8, like GBK or GB2312, set it to *iocharset=cp936*. If locale is UTF-8, set *iocharset=utf8*. As noted above, *ntf8* alone is better than *iocharset=utf8*.

# Refs

1. [iocharset和codepage有何联系和区别](tiebahttp://tieba.baidu.com/p/2317422644)
2. [关于mount/samba/字符集的两篇好文](http://zengrong.net/post/1019.htm)
3. [ 字符编码详解](http://www.crifan.com/files/doc/docbook/char_encoding/release/html/char_encoding.html)
4. [字符集编码cp936、ANSI、UNICODE](http://blog.csdn.net/wanghuiqi2008/article/details/8079071)
