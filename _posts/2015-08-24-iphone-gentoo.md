---
layout: post
title: Iphone Gentoo
---
How to mount iOS and transfer files to Iphone?

1. # echo "gnome-base/gvfs ios" >> /etc/portage/package.use/gvfs
2. # emerge -av gvfs usbmuxd
3. # usbmuxd
3. Restart X and login as a normal user account.
4. Thunar will remind you to input passcode in Iphone. Iphone might ask you to trust the computer. Just do it.
5. Now We can just transfer files to Iphone.
6. Refs:
    1. https://wiki.gentoo.org/wiki/Apple_iPod,_iPad,_iPhone
    2. https://wiki.archlinux.org/index.php/IPod

