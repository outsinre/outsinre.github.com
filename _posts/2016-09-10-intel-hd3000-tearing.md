---
layout: post
title: Intel HD3000 Tearing/Corruption/Glitch
---

> It's long-lasting panic to handle Intel HD3000 graphics corruption, tearing, judder, glitch etc. Desktop GUI get stuck without any responding. MPV playback spreads dots all over...

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
2. *DON'T* bump to lastest.

   *xf86-video-intel*, *mesa* etc.
3. Upgrade kernel to lastest.

