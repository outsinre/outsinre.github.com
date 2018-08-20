---
layout: post
title: Intel Graphics
---

It's long-lasting panic to handle Intel HD3000 graphics corruption, tearing, judder, glitch etc. Desktop GUI get stuck without any responding. MPV playback spreads dots all over. Worsely, switches between virtual terminal and X freeze the desktop applications.

# Intel DDX

It is almost deprecated but has better performance. We should manually install the official relevant Intel driver:

```bash
root@tux ~ # emerge -avt x11-drivers/xf86-video-intel
```

Then configure Xorg to adopt it:

```
# /etc/X11/xorg.conf.d/20-intel.conf

Section "Device"
        Identifier        "Intel Graphics"
        Driver        "intel"
#       Option        "AccelMethod"        "uxa"
        Option        "AccelMethod"        "sna"
#       Option        "TearFree"        "true"
        Option        "DRI"        "2"
#       Option        "DRI"        "3"
EndSection
```

This driver is just maintained but no new features or bug fixes available.

# Modesetting DDX

This is now the default driver on newer Intel graphics chipsets for Gentoo. It is more generic and actively developed. As of *x11-base/xorg-drivers-1.19*, this has become the default for Gentoo.

Firstly, enable global *glamor* USE:

```
# /etc/portage/make.conf

USE="glamor"
VIDEO_CARDS="intel i965"
```

Then, Xorg configuration:

```
# /etc/X11/xorg.conf.d/20-modesetting.conf

Section "Device"
    Identifier  "Intel Graphics"
    Driver      "modesetting"
    Option      "AccelMethod"    "glamor"
    Option      "DRI"            "3"
EndSection
```

Note, if both *20-intel.conf* and *20-modesetting.conf* are defined in */etc/X11/xorg.conf.d/*, the X server will attempt to load the files in alpha-numeric order. 

# Xorg arguments

```shell
startx -- vt7
```

Arguments after the two dashes are passed to Xorg server. Default OpenRC Xinit configuration is located under */etc/X11/xinit*. However it fails to set the correct virtual terminal (i.e. vt7) to start X, resulting in X freezes upon switches between X and virtual terminal.

Alternatively, *vt7* can be passed to X server directly in *~/.xserverrc*:

```shell
exec /usr/bin/X -nolisten tcp "$@" vt7
```

# ThinkPad Buttons

https://wiki.gentoo.org/wiki/ACPI/ThinkPad-special-buttons

# Screen brightness

```
# /etc/default/grub

acpi_osi="!Windows 2012"
# -or-
acpi_osi="Linux"

acpi_backlight="video"
# -or-
acpi_backlight="vendor"
# -or-
acpi_backlight="native"
```

At the time of writing this post, correct combination is `acpi_osi="!Windows 2012" acpi_backlight="video"`.