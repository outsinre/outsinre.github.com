---
layout: post
title: GTK recently-used.xbel
---

GTK2/3 library creates *~/.local/share/recently-used.xbel* for applications to keep their recently visited files, revealing user privacy! To disable *recently-used.xbel*:

1. Create directory with the same name.

   ```bash
   ~ $ rm .local/share/recently-used.xbel*
   ~ $ mkdir .local/share/recently-used.xbel
   ```

   This is enough! The next configurations are alternatives by changing GTK2/3 configurations.

2. GTK3 *setting.ini*

   ```bash
   ~ $ echo -e "[Settings]\ngtk-recent-files-max-age=0\ngtk-recent-files-limit=0" >> ~/.config/gtk-3.0/settings.ini
   ```

3. GTK2 *gtkrc-2.0*

   ```bash
   ~ $ echo "gtk-recent-files-max-age=0" >> ~/.gtkrc-2.0
   ```

4. Refs:
   1. [disabling-gnomes-recently-used-file-list-the-better-way](https://alexcabal.com/disabling-gnomes-recently-used-file-list-the-better-way/)
   2. [How do I prevent the file 'recently-used.xbel' from being created](https://askubuntu.com/a/269874)
   3. [GTK configuration](https://wiki.archlinux.org/index.php/GTK%2B#Configuration)
