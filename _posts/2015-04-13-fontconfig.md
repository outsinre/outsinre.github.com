---
layout: post
title: Fontconfig
---

Fontconfig is a library designed to provide system-wide font configuration, customization and application access. On Unix-like sytems, it is the main tool to manage fonts. This article aims to introduce installing fonts manually for single personal use.

# Under home directory

1. Get your fonts file like `*.ttf` and `*.otf`.
2. _#_ mkdir -p .local/share/fonts/myFonts, create a subdirectory under `.local/share/fonts` to hold your fonts.
3. _#_ cd .local/share/fonts/myFonts
4. Copy your fonts in step 1 to the newly created folder.
    1. Pay attention to font directory and files permission. Use `chmod` to set the correct permissions (i.e. at least 0444 for files and 0555 for directories). But don't worry about the permission issue since we are installing fonts for a single user. And the default permission for current user account is `rwx`. When installing globally for system, this is important.
5. _#_ fc-cache -fv <path-to-font-directory>, since we are in the new font directory, the `path-to-font-directory` which is `.local/share/fonts/myFonts` can be eliminated.
6. _#_ mkfontscale <path-to-font-directory>,
7. _#_ mkfontdir <path-to-font-directory>, some references omit step 6 && 7 when installing fonts for single user at under home directory. However, when installing fonts system-wide, these two steps are necessary.

> Install system-wide fonts are similar except that the fonts directory is `/usr/share/fonts/`.

Refer to:

1. [Font paths](https://wiki.archlinux.org/index.php/Font_configuration#Font_paths)
2. [Manual Installation](https://wiki.archlinux.org/index.php/Fonts#Manual_installation)
3. [TeX Live Fonts](https://wiki.archlinux.org/index.php/TeX_Live#Fonts)
4. [Install Windows Chinese Fonts](http://www.fangxiang.tk/2015/02/03/TeXLive-2014-Ubuntu-Installation/)

# Concepts

Fontconfig depends on `configuration file` and `font files`.

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
