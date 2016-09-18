---
layout: post
title: Intel HD3000 graphics corruption
---

> It's long-lasting panic to handle Intel HD3000 graphics corruption, tearing, judder etc. Desktop GUI get stuck without any responding. MPV playback spreads dots all over...

1. [Some ways to fix tearing on Linux Mint XFCE + Intel HD Graphics](https://komorebinomichi.wordpress.com/2015/08/24/some-ways-to-fix-tearing-on-linux-mint-xfce-intel-hd-graphics/) suggests *enable* XFCE setting *Window manager Tweaks* -> *Compositor* -> *Synchronize drawing to Vertical Blank*. Or even *disable* compositor *Enable Display Compositing*.

   I think *untick* the *Enable display compositing* is useful in making XFCE4 more beautiful.
   
   But I doubt it help the tearing issue.
2. Kernel options `IOMMU_SUPPORT`, `INTEL_IOMMU`, `INTEL_IOMMU_DEFAULT_ON` set to 'Y'.

   IOMMU Hardware Support, Support for Intel IOMMU using DMA Remapping Devices, Enable Intel DMA Remapping Devices by Default.
3. */etc/X11/xorg.conf.d/20-intel.conf*:

   ```
   Section "Device"
	   Identifier      "intel"
	   Driver          "intel"
	   Option          "AccelMethod"   "sna"
	   Option          "TearFree"      "true"
   EndSection
   ```

   Turn on *TearFree* for *sna*.
4. *DON'T* bump to lastest versions of

   *xf86-video-intel, libva-video-driver, libva* etc. unless you are sure what you are doing.
5. Try to `emerge -av1 @x11-module-rebuild`.
