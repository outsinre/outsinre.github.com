---
layout: post
title: Fontconfig
---

Fontconfig is a library designed to provide system-wide font configuration, customization and application access. On Unix-like sytems, it is the main tool to manage fonts.

# ABCs

1. Read [fonts-conf](http://freedesktop.org/software/fontconfig/fontconfig-user.html); [Jin Buguo](http://www.jinbuguo.com).
2. Everything starts with */etc/fonts/fonts.conf*.
3. [XDG Base Directory Specification](https://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html)

   This specification defines where these files should be looked for by defining one or more base directories relative to which files should be located. You can find out where is the default value for *XDG_DATA_HOME*, *XDG_CONFIG_HOME*, etc.

   BTW, different from *XDG base directory*, [xdg-user-dirs](https://www.freedesktop.org/wiki/Software/xdg-user-dirs/) refers to Desktop, Documents, Downloads, Pictures, Templates etc.
4. Try *fc-list*, *fc-query*, *fc-match* etc. commands.
5. Currently I use [Linux字体美化实战(Fontconfig配置)](http://www.jinbuguo.com/gui/linux_fontconfig.html) this scheme. The only difference is removing Korea fonts part.

# Per-user installation

Get fonts like `*.ttf` and `*.otf`:

```
MTEXTRA.TTF   fonts.dir    msyhbd.ttc   simhei.ttf  symbol.ttf    webdings.ttf
WINGDNG2.TTF  fonts.scale  msyhl.ttc    simkai.ttf  tahoma.ttf    wingding.ttf
WINGDNG3.TTF  msyh.ttc     simfang.ttf  simsun.ttc  tahomabd.ttf
```

Commands:

```bash
$ mkdir -p ~/.local/share/fonts/winfonts
$ cd ~/.local/share/fonts/winfonts
$ cp /path/to/font-files ./
$ fc-cache -fv ./
$ mkfontscale ./ && mkfontdir ./ (opt)
```

The last two commands *mkfontscale* and *mkfontdir* are for very ancient applications (i.e. GTK+1, xfontsel).

# System-wide installation

Install system-wide fonts are similar except that the fonts directories are different like */usr/share/fonts/*.

```bash
# mkdir /usr/share/fonts/winfonts
# chmod 755 /usr/share/fonts/winfonts
# cd /usr/share/fonts/winfonts
# cp /path/to/font-files ./
# chmod -R 644 ./*
# fc-cache -fv ./*
# mkfontscale ./ && mkfontdir ./ (opt)

# Xserver support (opt)

Most of time, Fontconfig works out of box without any further configuration on application side.

For Xserver to load fonts directly (as opposed to the use of a font server - Fontconfig), the directory for your newly added font must be added with a *FontPath* entry. This entry is located in the *Files section* of your Xorg configuration file.

Edit */etc/X11/xorg.conf.d/10-fonts-xorg.conf*:

```
Section "Files"
# Multiple FontPath entries are allowed (which are concatenated together),
# as well as specifying multiple comma-separated entries in one FontPath
# command (or a combination of both methods).
# The default path is shown here.
#    FontPath	/usr/share/fonts/misc/,/usr/share/fonts/TTF/,/usr/share/fonts/OTF/,/usr/share/fonts/Type1/,/usr/share/fonts/100dpi/,/usr/share/fonts/75dpi/
    FontPath    "/home/zachary/.local/share/fonts/winfonts"
    ...
    FontPath    "/usr/share/fonts/misc"
    FontPath    "/usr/share/fonts/100dpi"
    FontPath    "/usr/share/fonts/75dpi"
    FontPath    "/usr/share/fonts/util"
    FontPath    "/usr/share/fonts/encodings"
    ...
    FontPath    "/usr/share/fonts/winfonts"
EndSection
```

Refer to:

1. [Font paths](https://wiki.archlinux.org/index.php/Font_configuration#Font_paths)
2. [Manual Installation](https://wiki.archlinux.org/index.php/Fonts#Manual_installation)
3. [TeX Live Fonts](https://wiki.archlinux.org/index.php/TeX_Live#Fonts)
4. [Install Windows Chinese Fonts](http://www.fangxiang.tk/2015/02/03/TeXLive-2014-Ubuntu-Installation/)

# Sphere - infinality

*infinality* USE can be enabled globally in */etc/portage/make.conf*.

```bash
# echo "media-libs/freetype adobe-cff infinality" > /etc/portage/package.use/freetype
# emerge -av media-libs/freetype
# eselect fontconfig list
# eselect fontconfig enable 52-infinality.conf
```

*fontconfig-infinality* will draw in its own settings (*/etc/fonts/infinality*) which will interfere with the other fontconfig configuration. It's recommended to disable most of Fontconfig options while keeping *52-infinality.conf*. Specially, configurations related to specific fonts can be kept like *62-croscore-\**.conf* and *57-dejavu-\*.conf*.

You will find *fontconfig* (*/etc/fonts/*) and *infinality* (*/etc/fonts/infinality/*) has the nearly the same directory architecture. The *conf.d* sub-directory stores the selected symlinks, while *conf.avail* (or *conf.src*) stores all existing configurations. We use *eselect* which chooses and creates symbolic links under *conf.d* sub-directory.

Optionally, refer to reference number 3, set `embeddedbitmap` in `/etc/fonts/infinality/infinality.conf` to `true`.

## *infinality* symlinks

```bash
# eselect lcdfilter list
# eselect lcdfilter set 14, set to *windows-7*
# eselect infinality list
# eselect infinality set to *win7*
```

Make sure *lcdfilter* and *infinality* choose the same category.

# Reference:

1. [Gentoo Linux on T43 (7) 中文字体](http://ted.is-programmer.com/categories/1547/posts)
2. [字体配置local.conf详解[带Win效果和AA效果]](https://www.freebsdchina.org/forum/viewtopic.php?t=34824&start=0&postdays=0&postorder=asc&highlight=)
3. [gentoo 下字体美化该如何设置](https://groups.google.com/forum/#!topic/gentoo-china/gzW8mg9OIhg)
4. [在Gentoo上配置Infinality](https://gist.github.com/kidlj/f30e82c2c6f064990596)
5. [Gentoo Fontconfig](https://wiki.gentoo.org/wiki/Fontconfig).
