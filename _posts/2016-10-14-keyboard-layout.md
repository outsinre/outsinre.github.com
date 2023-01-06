---
layout: post
title: Keyboard layout
comments: true
---

# ABCs

1. Emacs users are subject to [Emacs pinky](https://www.emacswiki.org/emacs/RepeatedStrainInjury). Common solution is to  *switch Caps and Ctrl* or *turn off Caps*.

   Caps is recommended to turned off as it's only used occasionally. What's more, we can use Shift to *shout*.
2. Setting for virtual termnial is different from that of X, and should be set respectively.
3. In this post, I assume Intel PC and QWERTY keybord.

# Virtual Terminal (OpenRC)

> In Virtual Terminal, without specific clarification, term *layout* refers to *keymap*.

1. Keyboard layout can be found under */usr/share/keymaps*. Keymap compressed in *.gz* format. Depending on the hardware, they are organized in sub-directories.

   For Intel QWERTY keyboard, the default keymap is *i386/qwerty/us.map.gz*.

   Among other keymaps, there is a special *include* directory offer supplemental keymap *extensions*.
2. There are two import parameters in */etc/conf.d/keymaps*, namely *keymap* and *extended_keymaps*.

   *keymap=us* sets keyboard layout to *us.map.gz* while *extended_keymaps* is a comma-listed *extension*s.

   In this post, we will ignore *extended_keymaps*.
3. To switch Caps/Ctrl or turn off Caps, we can modiy the *keycode* in keymap (decompressed). Before that, let's examine the *us.map* format:

   ```
   include "linux-with-alt-and-altgr"
   keycode  29 = Control
   keycode  58 = Caps_Lock
   keycode  97 = Control
   ```
   
   1. *include* *extend*s the current keymap.
   2. *keycode 29 = Control* means the Ctrl key is pressed which also applies to 58 and 97.
   3. Specially 97 means the right Ctrl key while 29 is left Ctrl key.
4. Switch Caps and left Ctrl:

   ```
   keycode  29 = Caps_Lock
   keycode  58 = Control
   ```
   
   Just swap the keycode for Control and Caps_Lock. Yeah, right, you can swap 29 and 97 (right Control) as well.
5. Turn off Caps:

   ```
   keycode  58 = Control
   ```

6. Although modification the original default *us.map.gz* is handy, create a new keymap is preferrable.

   Create *caps-off.map*:
   
   ```
   include "us.map"
   keycode  58 = Control
   ```

   Remember to compress the new keymap:

   ```bash
   $ gzip caps-off.map
   # mv caps-off.map.gz /usr/share/keymaps/i386/qwerty/
   ```

   Then set *keymap=caps-off* in */etc/conf.d/keymaps*.
7. Finally restart *keymaps* service:

   ```bash
   # rc-service keymaps restart
   ```

8. Interestingly, someone already compose a customized */usr/share/keymaps/i386/qwerty/emacs.map.gz*.

   I have chosen to use this one *keymap=emacs*. It includes much more customization for Emacs.
9. Attention: changing */etc/conf.d/keymaps* directly affects only Virtual Terminal.

# X

1. Aside from changing *keymap*s, X supports XKB in */usr/share/X11/xkb/rules/base.lst* instead of raw *keycode*.

   X KeyBoard extension, or XKB, defines the way keyboards codes are handled in X, and provides access to internal translation tables. It is the basic mechanism that allows using multiple keyboard layouts in X.
3. (deprecated) *x11-apps/xmodmap* is a somewhat low level tool, similar to OpenRC */etc/conf.d/keymaps*.

   *xmodmap* can temporarily change keymaps on command line (Terminal Emulator). However, for persistent effect,  create a configuration file *~/.Xmodmap*:

   ```
   !
   ! Swap Caps_Lock and Control_L
   !
   remove Lock = Caps_Lock
   remove Control = Control_L
   keysym Control_L = Caps_Lock
   keysym Caps_Lock = Control_L
   add Lock = Caps_Lock
   add Control = Control_L
   ```

   Insert *xmodmap ~/.Xmodmap* command to *~/.xinitrc*,  *~/.Xresources*, *~/.xprofile* (Gnome Display Manager) or whatever. Alternatively, we can add it to desktop's *autostart* list. Most desktops (Xfce4, Gnome, or KDE) supports such setting.
4. XKB command line tool *x11-apps/setxkbmap* is a relativelly high level tool. If present, it resets */etc/conf.d/keymap* and *xmodmap* settings.

   Edit *~/.xinitrc* or *~/.xprofile* and call *setxkbmap* from there.

   ```
   /usr/bin/setxkbmap -option "ctrl:swapcaps"
   or
   /usr/bin/setxkbmap -option "ctrl:nocaps"
   ```

5. XKB in Xorg configuration

   Without installing *setxkbmap* package, we can use XKB rules in */etc/X11/xorg.conf.d/10-keyboard.conf*:

   ```
   Section "InputClass"
	   Identifier      "keyboard-all"
	   MatchIsKeyboard "on"
	   #Option  "XkbOptions"    "ctrl:swapcaps"
	   #Option  "XkbOptions"    "terminate:ctrl_alt_bksp"
	   Option  "XkbOptions"    "ctrl:nocaps"
   EndSection
   ```

   Restart X to take effect.

# Refs

1. [Keyboard layout switching](https://wiki.gentoo.org/wiki/Keyboard_layout_switching)
2. [Moving the Ctrl key](https://www.emacswiki.org/emacs/MovingTheCtrlKey)
3. [keybord configuration in Xorg](https://wiki.archlinux.org/index.php/Keyboard_configuration_in_Xorg)
4. [x keybord extension XKB](https://wiki.archlinux.org/index.php/X_KeyBoard_extension)
