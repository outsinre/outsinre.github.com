---
layout: post
title: Fontconfig
---

Fontconfig is a library designed to provide system-wide font configuration, customization and application access. On Unix-like sytems, it is the main tool to manage fonts. This article aims to introduce installing fonts manually for single personal use.

# Fontconfig notes

Fontconfig depends on `configuration files` and `font files`.

These are the locations for configuration files:

```
/etc/fonts/fonts.conf
/etc/fonts/fonts.dtd
/etc/fonts/conf.d
$XDG_CONFIG_HOME/fontconfig/conf.d
$XDG_CONFIG_HOME/fontconfig/fonts.conf
~/.fonts.conf.d
~/.fonts.conf
```
Actually, there is another system-wide configuration file `/etc/fonts/local.conf`. Please check `/etc/fonts/conf.d/51-local.conf` to check details.

And the font files directory:

```
<dir>/usr/share/fonts</dir>
<dir>/usr/local/share/fonts</dir>
<dir prefix="xdg">fonts</dir>
<!-- the following element will be removed in the future -->
<dir>~/.fonts</dir>
```
Refer to [fonts-conf](http://freedesktop.org/software/fontconfig/fontconfig-user.html).

# XDG Base Directory Specification

Various specifications specify files and file formats. This specification defines where these files should be looked for by defining one or more base directories relative to which files should be located.

Refer to the official document [XDG Base Directory Specification](http://standards.freedesktop.org/basedir-spec/basedir-spec-0.6.html). You can find out where is the default value for `XDG_DATA_HOME`, `XDG_CONFIG_HOME`, etc.

# Per-user installation

Before installing fonts, refer to [Linux字体美化实战](http://www.jinbuguo.com/gui/linux_fontconfig.html) on choosing different fonts for system.

1. Get your fonts file like `*.ttf` and `*.otf`:

    ```
MTEXTRA.TTF   fonts.dir    msyhbd.ttc   simhei.ttf  symbol.ttf    webdings.ttf
WINGDNG2.TTF  fonts.scale  msyhl.ttc    simkai.ttf  tahoma.ttf    wingding.ttf
WINGDNG3.TTF  msyh.ttc     simfang.ttf  simsun.ttc  tahomabd.ttf
    ```
    Some of the fonts are for `wps-office` and others are for Chinese fonts.
2. _$_ mkdir -p ~/.local/share/fonts/myFonts, create a subdirectory under `.local/share/fonts` to hold your fonts.
3. _$_ cd ~/.local/share/fonts/myFonts
4. Copy your downloaded fonts to `myFonts`.
    1. Pay attention to font directory and files permission. Use `chmod` to set the correct permissions (i.e. at least 0444 for files and 0555 for directories). But don't worry about the permission issue since we are installing fonts for a single user. And the default permission for current user account is `rwx`. When installing globally for system, this is important.
5. _$_ fc-cache -fv \<path-to-font-directory\>, since we are in the new font directory, the `path-to-font-directory` which is `.local/share/fonts/myFonts` can be eliminated.
6. _$_ mkfontscale \<path-to-font-directory\>
7. _$_ mkfontdir \<path-to-font-directory\>

# System-wide installation
1. Install system-wide fonts are similar except that the fonts directories are different like  `/usr/share/fonts/`.
2. _#_ mkdir /usr/share/fonts/winfonts
3. _#_ chmod 755 /usr/share/fonts/winfonts
4. _#_ cd /usr/share/fonts/winfonts
4. Copy all the fonts fileto directory `winfonts`
5. _#_ chmod 644 *
6. _#_ fc-cache -fv && mkfontscale && mkfontdir

# Xserver support
1. For Xserver to load fonts directly (as opposed to the use of a font server) the directory for your newly added font must be added with a `FontPath` entry. This entry is located in the `Files section` of your Xorg configuration file.
8. _#_ mkdir /etc/X11/xorg.conf.d/, there is an example copy of `xorg.conf` at /usr/share/doc/xorg-server-1.16.4/xorg.conf.example.bz2. But we only need the `Files section`.
9. _#_ cd /etc/X11/xorg.conf.d/
10. _#_ emacs -nw /etc/X11/xorg.conf.d/10-fonts-xorg.conf, the file contents are like:

    ```
Section "Files"
# Multiple FontPath entries are allowed (which are concatenated together),
# as well as specifying multiple comma-separated entries in one FontPath
# command (or a combination of both methods).
# The default path is shown here.
#    FontPath	/usr/share/fonts/misc/,/usr/share/fonts/TTF/,/usr/share/fonts/OTF/,/usr/share/fonts/Type1/,/usr/share/fonts/100dpi/,/usr/share/fonts/75dpi/
    FontPath    "/home/zachary/.local/share/fonts/myFonts"
    ...
    FontPath    "/usr/share/fonts/100dpi"
    FontPath    "/usr/share/fonts/75dpi"
    FontPath    "/usr/share/fonts/misc"
    FontPath    "/usr/share/fonts/util"
    FontPath    "/usr/share/fonts/winfonts"
    ...
    FontPath    "/usr/share/fonts/encodings"
EndSection
    ```
Refer to:

1. [Font paths](https://wiki.archlinux.org/index.php/Font_configuration#Font_paths)
2. [Manual Installation](https://wiki.archlinux.org/index.php/Fonts#Manual_installation)
3. [TeX Live Fonts](https://wiki.archlinux.org/index.php/TeX_Live#Fonts)
4. [Install Windows Chinese Fonts](http://www.fangxiang.tk/2015/02/03/TeXLive-2014-Ubuntu-Installation/)

# Fontconfig Customization - infinality
1. _#_ echo "media-libs/freetype adobe-cff infinality" > /etc/portage/package.use/freetype

    It seems that `adobe-cff` is enabled by default.
2. _#_ emerge -av media-libs/freetype, reinstall freetype to take advantage of `adobe-cff` and `infinality`.

    It will draw in another two packages `fontconfig-infinality` and `eselect-infinality`. If not, emerge them manually.
2. _#_ eselect fontconfig list, you will find a new option `52-infinality.conf` in the list.
    1. _#_ eselect fontconfig enable 52-infinality.conf
    2. `fontconfig-infinality` will draw in its own settings which will interfere with the other fontconfig configuration; so need to disable most of the other fontconfig options while keeping `52-infinality.conf`.

        Where is `infinality`'s own settings locates? Refer to `/etc/fonts/infinality`.
    3. _#_ eselect fontconfig disable xx yy zz ...
        1. [deprecated, not necessary at all] <s>I choose to **keep** `50-user.conf` (for per-user config files) instead of disabling.</s>

        Up to now, `eselect` has created two conf files under `/etc/fonts/conf.d`. Use `ls -l`, you will find them actually symbolic link referring to corresponding files under `/etc/fonts/conf.avail`. Please read the two conf files for an overview.

        config related to fonts themself can be kept like: 62-croscore-*.conf and 57-dejavu-*.conf

        You will find `fontconfig` (`/etc/fonts/`) and `infinality` (`/etc/fonts/infinality/`) has the nearly the same directory architecture. The `conf.d` sub-directory stores the current font configuration files, while `conf.avail` (or `conf.src`) stores all the possible configuration files. All we need to to is to `eselect` a font portfolio which will choose and create symbolic links under `conf.d` sub-directory.
    4. Refer to reference number 3, set `embeddedbitmap` in `/etc/fonts/infinality/infinality.conf` to `true`.
3. _#_ eselect lcdfilter list
4. _#_ eselect lcdfilter set 14, set to `windows-7`.
5. _#_ eselect infinality list
6. _#_ eselect infinality set to `win7`.
    1. Make sure `lcdfilter` and `infinality` choose the same category.
7. [deprecated, see step 3.3.1]<s>Choose to place customized fontconfig file in `~/.config/fontconfig/fonts.conf` (50-user.conf) or `/etc/fonts/local.conf` (51-local.conf). Here is the sample for per-user [fonts.conf]({{site.baseurl}}assets/fonts.conf).</s>
8. Use `fc-list | head` to check if any errors occur. This is important. During the setting, I found an error about `infinality` configuration files.
9. Refer to [Gentoo字体设置]({{site.baseurl}}assets/Gentoo字体设置.markdown) for the basic procedures.
10. Reference:
    1. [Gentoo Linux on T43 (7) 中文字体](http://ted.is-programmer.com/categories/1547/posts)
    2. [字体配置local.conf详解[带Win效果和AA效果]](https://www.freebsdchina.org/forum/viewtopic.php?t=34824&start=0&postdays=0&postorder=asc&highlight=)
    3. [gentoo 下字体美化该如何设置](https://groups.google.com/forum/#!topic/gentoo-china/gzW8mg9OIhg)
    4. [在Gentoo上配置Infinality](https://gist.github.com/kidlj/f30e82c2c6f064990596)
    5. [Gentoo Fontconfig](https://wiki.gentoo.org/wiki/Fontconfig).
    