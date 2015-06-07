---
layout: post
title: library soname
---
In Unix operating systems, a `soname` is a field of data in a shared object file. The `soname` is a string, which is used as a "logical name" describing the functionality of the object.

The `soname` is often used to provide version **backwards-compatibility** information. For instance, if versions 1.0 through 1.9 of the shared library libx provide identical interface, they would all have the same `soname`, e.g. *libx.so.1*. If the system only includes version 1.3 of that shared object, with `filename` *libx.so.1.3*, the *soname* field of the shared object tells the system that it can be used to fill the dependency for a binary which was originally compiled using version 1.2.

Linux 系统，也同样面临和Window一样的问题，如何控制动态库的多个版本问题。Window之前没有处理好，为此专门有个名词来形容这个问题 “Dll hell”，其严重影响软件的升级和维护。 Dll hell 是指windows 上动态库新版本覆盖旧版本，﻿﻿但是却不兼容老版本。常常发生在程序升级之后，动态库更新，原有程序运行不起来；或者装新软件，但是已有的软件运行不起来。 同样Linux操作系统，也有同样的问题，那么它是怎么解决的呢？

Linux 为解决这个问题，引入了一套机制，如果遵守这个机制来做，就可以避免这个问题。 但是这只事一个约定，不是强制的。但是建议遵守这个约定，否则同样也会出现 Linux 版的Dll hell 问题。 下面来介绍一个这个机制。 这个机制是通过文件名，来控制dll （shared library） 的版本。

Linux 上的Dll ，叫shared library，其有三个名字，分别又不同的目的。

Every shared library has a special name called the `soname`. The soname has the prefix `lib`, the name of the library, the phrase `.so`, followed by a period and a version number that is incremented whenever the interface changes (as a special exception, the lowest-level C libraries don't start with `lib`). A fully-qualified soname includes as a prefix the directory it's in; on a working system a fully-qualified soname is simply a symbolic link to the shared library's `real name`.

Every shared library also has a `real name`, which is the filename containing the actual library code. The real name adds to the soname a period, a minor number, another period, and the release number. The last period and release number are optional. The minor number and release number support configuration control by letting you know exactly what version(s) of the library are installed. Note that these numbers might not be the same as the numbers used to describe the library in documentation, although that does make things easier.

In addition, there's the name that the compiler uses when requesting a library, (I'll call it the `linker name`), which is simply the soname without any version number.

The key to managing shared libraries is the separation of these names. Programs, when they internally list the shared libraries they need, should only list the soname they need. Conversely, when you create a shared library, you only create the library with a specific filename (with more detailed version information). When you install a new version of a library, you install it in one of a few special directories and then run the program ldconfig(8). ldconfig examines the existing files and creates the sonames as symbolic links to the real names, as well as setting up the cache file /etc/ld.so.cache (described in a moment).

ldconfig doesn't set up the linker names; typically this is done during library installation, and the linker name is simply created as a symbolic link to the `latest` soname or the latest real name. I would recommend having the linker name be a symbolic link to the soname, since in most cases if you update the library you'd like to automatically use it when linking. I asked H. J. Lu why ldconfig doesn't automatically set up the linker names. His explanation was basically that you might want to run code using the latest version of a library, but might instead want development to link against an old (possibly incompatible) library. Therefore, ldconfig makes no assumptions about what you want programs to link to, so installers must specifically modify symbolic links to update what the linker will use for a library.

Thus, /usr/lib/libreadline.so.3 is a fully-qualified soname, which ldconfig would set to be a symbolic link to some realname like /usr/lib/libreadline.so.3.0. There should also be a linker name, /usr/lib/libreadline.so which could be a symbolic link referring to /usr/lib/libreadline.so.3.

1. real name: libmath.so.1.1.1234

    lib + name + .so + .major number + .minor number + .release number
     
    主板号，代表当前动态库的版本，就是共享库文件本身的文件名，里面是共享库的代码。**如果动态库的接口有变化，那么这个版本号就要加1**；后面的两个版本号（小版本号 和 build 号）是告诉你详细的信息，比如为一个hot-fix 而生成的一个版本，其小版本号加1，build号也应有变化。
2. soname: libmath.so.1
动态库的soname，其是应用程序加载共享库时，帮助寻找共享库用的文件名。其格式为:

    lib + name +.so + .major number

    其只包含major version number，换句话说，也就是只要其接口没有变，应用程序都可以用，不管你其后minor build version or build version。soname通常存在于几个地方：作为软链接指向real name；存放在共享库本身的文件头部header；存放在应用程序的头部共享库声明部分。
3. linker name: libmath.so

    lib + name + .so

    连接名（linker name），是专门为应用程序在链接link阶段找到共享库而设置的指向soname或这real name的软链接，一般在系统安装共享库时，系统会自动创建，通常默认应该指向系统里**最新的**soname or real name。

4. 在共享库编译过程中，连接（link） 阶段，编译器将生成一个共享库及real name，同时将共享库的soname，写在共享库文件里的文件头里面。可以用命令 `readelf -d libmath.so.1.1.1234` 去查看。在应用程序编译阶段，引用共享库时，其会用到共享库的linker name，其通过linker name 找到最新的动态库库文件，并且把共享库的soname提取出来，写在自己的共享库的头里面。当应用程序加载时候就会通过soname软链接去在给定的路径下寻找该共享库。

    libmath.so -> libmath.so.1 -> libmath.so.1.1.1234
5. **linker name是link阶段找到最新共享库版本服务的**

## 共享库小版本升级

当升级小版本时，即接口不变。共享库的soname是不变的，所以需要重新把soname的那个连接文件指定新版本就可以。 调用ldconfig命令，系统会帮你做修改那个soname link文件，并把它指向新的版本呢。这时候你的应用程序就自动升级了。

## 共享库，主版本升级

即接口发生变化，这是共享库的soname发生了变化。如果多个版本的共享库并存，那么就多多个soname 分别链接到对应的real name，而linker name总是指向最新的那个real name。

Reference [shared-libraries](http://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html).
