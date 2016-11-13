---
layout: post
title: Intel HD3000 Tearing/Corruption/Glitch
---

> It's long-lasting panic to handle Intel HD3000 graphics corruption, tearing, judder, glitch etc. Desktop GUI get stuck without any responding. MPV playback spreads dots all over. Worsely, switches between virtual terminal and X freeze the desktop applications.

1. */etc/X11/xorg.conf.d/20-intel.conf*:

   ```
   Section "Device"
           Identifier        "Intel Graphics"
           Driver        "intel"
   #       Option        "AccelMethod"        "uxa"
           Option        "AccelMethod"        "sna"
           Option        "TearFree"        "true"
   #       Option        "DRI"        "2"
           Option        "DRI"        "3"
   EndSection
   ```

   Turn on *TearFree* for *sna*.
2. Bump to lastest (even 9999).

   *xf86-video-intel* and *mesa* (even *xorg-server* and *xorg-drivers*) etc.
3. Upgrade kernel to lastest.
4. Extra Xorg arguments

   ```bash
   # startx -- vt7
   ```
   
   Arguments after the two dashes are passed to Xorg server. Default OpenRC Xinit configuration is located under */etc/X11/xinit*. However it fails to set the correct virtual terminal (i.e. vt7) to start X, resulting in X freezes upon switches between X and virtual terminal.

   Alternatively, *vt7* can be passed to X server directly in *~/.xserverrc*:

   >exec /usr/bin/X -nolisten tcp "$@" vt7