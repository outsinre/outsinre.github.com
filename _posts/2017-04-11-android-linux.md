---
layout: post
title: Android Linux
---

# Outline

1. Adb

   PC tool that sends commands to Android system over cable or Wi-Fi. Commands are:

   1. Adb sub-commands like *shell*, *install*, *push*, *sideload* etc.
   2. Android system commands like *pm*, *am* etc. by *adb shell*.
2. Fastboot

   Similar to Adb, Fastboot connects to Android bootloader - Fastboot by cable. As Android bootloader does not provide built-in commands, Fastboot supports only sub-commands of itself like *flash* etc.

   >Fastboot tool VS Fastboot bootloader.

3. TWRP

   Android system includes the tiny Recovery and complete Android ROM (i.e. MIUI). TWRP is an extension to stock Recovery with more functionalities. Mostly, we use it to flash 3rd party ROMs.

   Possible TWRP sub-commands are *wipe*, *install* etc. by *adb shell twrp*.

   >You could change TWRP version if flashing failed.

# Thunar

```bash
~ # echo "gnome-base/gvfs mtp" >> /etc/portage/package.use/gvfs
~ # emerge -av1 gvfs
```

>Logout and login to take effect.

# [TWRP](https://twrp.me)

1. Set `rm -rf` to avoid re-formating EXT4 filesystem unpon wiping.
2. After wiping and before flashing, keep all (at least */system*) partitions *umount*ed and disable MTP, which otherwise would report *system*, *data* etc. partitions *busy* errors.
3. Error. If you receive error like:

   ```
   Unable to mount /data
   ```

   Resort to:

   1. Adanced Wipe, Repair or Change File System;
   2. Reboot TWRP.
   3. bootloader emergency recovery wipe;
   4. adb sideload.
4. Instead of flashing TWRP to device, we can temporarily *boot* device with TWRP image.

# ROM

1. If previous ROM is official Android 6.0 and new ROM is 3rd party Android 7.0/7.1, you'd better firstly flash an official 7.0/7.1 development version (a big so-called "底包").
2. To support F2FS data partition:
   1. 3rd party recovery (i.e. TWRP) should support creating F2FS filesystem (formating *data* and *cache* partitions). Don't format *system* partition as F2FS.

       Formatting *data* partition to F2FS would wipe the *internal storage* too.
   2. ROM must run on F2FS systems.

# [Android Debug Bridge](https://developer.android.com/studio/command-line/adb.html) (adb)

Android Debug Bridge (adb) is a versatile command-line tool that lets you communicate with a device (an *emulator* or a connected Android device).

The adb command facilitates a variety of device actions, such as installing and debugging apps, and it provides access to a Unix shell that you can use to run a variety of commands on a device.

It is a client-server program that includes three components:

1. A client, which sends commands. The client runs on your development machine. You can invoke a client from a command-line terminal by issuing an adb command.
2. A daemon (adbd), which runs commands on an Android device. The daemon runs as a background process on each device.
3. A server, which manages communication between the client and the daemon. The server runs as a background process on your development machine.

## Installation

From Gentoo portage:

```bash
~ # emerge -avt dev-util/android-tools
# Android platform tools (adb, fastboot, and mkbootimg)
# Logout and login back to take effect
```

Current stable is 1.0.32. For an up-to-date version (recommended), [download here](https://developer.android.com/studio/releases/platform-tools.html), which is a *standalone* version ready for terminal (just extract the zip):

```bash
~ $ cd ~/opt
~ $ unzip ~/Downloads/platform-tools-latest-linux.zip
~ $ ./adb
~ $ ./fastboot
```

## Connection to device

There are two ways to pair an Android device with your PC, namely over USB cable and over Wi-Fi.

In order to let computer recognizes the Android device, we should enable:

1. [Developer options](https://developer.android.com/studio/debug/dev-options.html);
2. USB debugging;
3. Fastboot Unlock (i.e. Xiaomi virgo).

### Over USB cable

```bash
~ $ ./adb start-server/kill-server
~ $ ./adb devices [-l]
~ $ ss/netstat -npelt
```

*adb devices* launches *adb start-server* on demand. When the server starts, it binds to local TCP port 5037 and listens for commands sent from adb clients - all adb clients use port 5037 to communicate with the adb server.

>adb client - adb server - adb daemon on Android

Example output:

```
List of devices attached
afc75758               device usb:1-1.2 product:NX531J model:NX531J device:NX531J
```

The serial number is *afc75758* while *device* represents connection state.

If *adb* prints:

```
List of devices attached
afc75758	unauthorized
```

Unlock your device and confirm PC RSA key (for first connection). This security mechanism protects user devices because it ensures that USB debugging and other adb commands cannot be executed unless you're able to unlock the device and acknowledge the dialog.

If *adb* complains:

```
List of devices attached
afc75758               no permissions (udev requires plugdev group membership); see [http://developer.android.com/tools/device.html] usb:1-1.2
```

Possible methods:

1. Add current user account into *plugdev* group.
2. Run *adb* under *root* account.
3. Switch the USB connection type from *Charge only* to *Transmit files* (MTP) or *Transmit photos* (PTP) from device drop-down menu. Alternatively, set *Select USB configuration* in *Developer options*.
4. Follow the given link to create a **udev** rule your device:

   ```
   # lsusb
   Bus 001 Device 035: ID 19d2:abcd ZTE WCDMA Technologies MSM
   ```

   *lsub* prints the device vendor ID (19d2) and product ID (abcd). If */lib/udev/rules.d/69-libmtp.rules* (*media-libs/libmtp*) does NOT include your device, then

   ```
   # /etc/udev/rules.d/51-android.rules
   SUBSYSTEM=="usb", ATTR{idVendor}=="19d2", ATTRS{idProduct}=="abcd", GROUP="plugdev", MODE="0660", SYMLINK+="NX531J-%k"
   # or
   SUBSYSTEM=="usb", ATTRS{idVendor}=="19d2", ATTRS{idProduct}=="abcd", OWNER="jim", MODE="0600", SYMLINK+="NX531J-%k"
   ```

   Finally, *trigger* the rules:

   ```bash
   ~ # gpasswd -a username plugdev (optional)
   ~ # udevadm control -R/--reload (optional)
   ~ # udevadm trigger
   ```

   Pay attention to how GROUP and OWNER relates to MODE.
4. We might create a new rule for *fastboot* (bootloader) uses a different *idProduct* number.
5. Different ROM gives differents *idProduct* number.

### Over Wi-Fi

Connection over Wi-Fi needs some initial setup over USB. Connect your Android device and adb host computer to a common Wi-Fi network accessible to both (i.e. WLAN).

```bash
~ $ ./adb tcpip 5037
~ $ ./adb connect device_ip_address
~ $ ./adb devices -l
```

>Connection over Wi-Fi is slow in speed and less secure subjecting to sniffer.

## adb sub-commands

```bash
~ $ adb [-d | -e | -s serial_number] command
```

One of the most special is *shell*:

```bash
~ $ ./adb shell
# get into Android Linux system
shell@NX531J:/ $ ls /system/bin
shell@NX531J:/ $ netstat -lt
```

If you are connecting over Wi-Fi, *netstat* will show the TCP socket which is owned by *adb shell*. To tell the ADB daemon return to listening over USB and closing the socket:

```bash
~ $ ./adb usb
```

Other commands are *pull/push/backup* for file transmission and backup/restore. Pull/push is pretty fast to transmit files but may the device a hot potato - high temprature.

You should also pay attention to *remount/install/uninstall/shell* for file updating. Apart from traditional shell commands, there are two special ones, namely **am** (activity manager) and **pm** (package manager).

Lastly, the most common command is:

```bash
~ $ reboot [bootloader|recovery|sideload|sideload-auto-reboot]
```

As stated in the Outline section, we can use *adb shell twrp install /path/to/zip/ROM/on/android* to flash a new ROM.

### logcat

*logcat* is a pretty useful tool to collect system logs which can be filtered by parameters to focus on specific app.

```bash
~ $ ./adb shell
shell@NX531J:/ $ logcat --help
# or
~ $ ./adb logcat --help
```

For example, we want logs of Amaze:

```bash
~ $ ./adb shell ps | grep -i amaze
# get the PID
~ $ ./adb logcat | grep -F $PID
~ $ ./adb logcat -s *:E
```

# Flashing

1. Fastboot is Android device's bootloader that resembles Grub in loading kernel.
   1. A system can be a tiny Recovery system or a complete ROM system. That is to say, Recovery and ROM are of equal objects in viewpoint of Fastboot.
   2. Device and PC must be connected cable line. This is the reason we call it "线刷" (i.e. fastboot update update.zip). Here is a reference [命令行下的fastboot刷机](http://bbs.xiaomi.cn/t-12560364).
2. Recovery is a tiny runnable Android system. As explained above, we can connect PC to device recovery over cable line or Wi-Fi.

   Usually Recovery is used to do "卡刷" (PC involvement).
3. *fastboot* and *adb* commands communicate with *fastboot* bootloader and *recovery/ROM* system respectively.
4. The ROM (zip) format of "线刷" and "卡刷" is probably different.

   Check your vendor's instruction.
5. After wiping partitions (data, cache, system, etc.), you'd better reboot TWRP.

## fastboot TWRP

fastboot can flash sparse *img* (recovery, data, system, and even self-made ones) to relevant Android partitions. To make fastboot recognize your device,

1. Let the device get into fastboot mode.

   ```bash
   # get into fastboot mode
   ~ $ ./adb reboot bootloader
   ```

   Another way is to use combination keys mostly by pressing power key and volume down key simultaneously. Details refer to your device documentation.
2. Create a udev rule.

   When the device is at fastboot mode, the product ID changes accordingly. We should add another udev rule like for adb.
3. boot/flash

   ```bash
   ~ $ ./fastboot devices -l
   ~ $ ./fastboot boot recovery /path/to/TWRP.img (recommended)
   ~ $ ./fastboot [-i 0x19D2 ] flash recovery /path/to/TWRP.img
   ```

4. getvar

   ```bash
   ~ $ ./fastboot getvar all
   ````

The [difference](http://c.mi.com/thread-31082-1-1.html) between *flash* and *boot* is that:

1. *flash* do flash the TWRP to recovery parition for *permanent* effect.
2. *boot* *temporarily* boots the device with TWRP, which resembles a LiveCD environment.

## [adb sideload](https://twrp.me/faq/ADBSideload.html) ROM

As of version 2.3, TWRP now supports *adb sideload* mode. ADB sideload is a different ADB mode that you can use to push and install a zip using one command from your computer. Don't mix *adb sideload* with *fasboot flash* ("线刷"), though they both work over cable connection.

*adb sideload* requires reliable USB cable connection.

1. Let your device be in sideload. You either do it in TWRP menu or by adb command line from your computer:

   ```bash
   ~ $ ./adb reboot sideload/sideload-auto-reboot
   ~ $ ./adb devices
   ```

   You will get:

   ```
   List of devices attached
   afc75758	sideload
   ```

2. Flash the zip ROM

   ```bash
   ~ $ ./adb sideload /path/to/rom.zip
   ```

# ROM custiomization

Before flashing official zip ROM, delete builit-in promotional APKs to save storage. Alternatively, you have to delete them by *root* apps after flashing.

1. To add back a previously deleted app, *push* both the apk and *odex*. *install* does NOT help in case of apk with separate *odex*.

   ```bash
   ~ $ ./adb remount
   ~ $ ./adb push /path/apk /system/app/
   ```

   1. You may *remount* before *push*ing to */system/app*.
   2. Check newly pushed files permission mode.
   3. You can also make a zip ROM to install just one app. Details refer to [努比亚z11精简包+系统程序中英文对照表](http://bbs.nubia.cn/thread-786029-1-1.html).
2. To further delete a built-in apk,

   ```bash
   ~ $ ./adb shell pm list packages | grep -i example
   ~ $ ./adb pm uninstall nubia.example.cn
   ~ $ ./adb reboot recovery
   ~ $ ./adb shell
   shell@NX531J:/ $ find / -iname '*example*'
   shell@NX531J:/ $ find / -iname '*example*' -delete
   ```

   Sometimes deleted apps reappear somehow. They are restored by Android cache partition. Wipe cache gets rid of that.
3. Add personal apps to ROM
   1. Put your apk (i.e. *Example*) into */system/app/Example*.
   2. If apk depends on system lib (check /META-INF/com/google/android/updater-script), do a *symlink* to */system/lib*.
   3. Lastly, check files modes.

   ```
   以下再以另一个例子来说明如何内置带有库（LIB）的软件。

   我以来电通为例子

   在电脑上用RAR打开“来电通.APK”。发现它是带有LIB目录的。

   进入它，并把那两个SO文件拉出来，是的就是拉。然后放到手机的TF卡上。我一般喜欢放到个文件夹中。

   这里记得把来电通的中文名改成一个你喜欢的英文名。比如说我的改成LDT1024.apk(如果文件名是英文，可以不不改掉。)

   在RE中按住MENU 多选，选两个SO文件。复制它们到SYSTEM/LIB中。

   同理，再回去把LDT1024. apk 拷到SYSTEM/APP中。

   重启。接着你进去会发现，程序那里多出来了两个来电通的东东。一个是主程另一个来电通拔号。
   ```

# [apktool](https://ibotpeaches.github.io/Apktool/)

To install apktool, at least java 1.7 is installed. Afterwards, just follow [Install Instructions](https://ibotpeaches.github.io/Apktool/install/).

1. Put the two downloaded files into home directory *~/opt/apktool* instead of */usr/bin*.

   ```bash
   ~ $ cd ~/opt/apktool
   ~ $ chmod +x apktool*
   ~ $ ./apktool
   ```

2. Linux wrapper script is *optional* but we have to type a long command:

   ```bash
   ~ $ cd ~/opt/apktool
   ~ $ java -jar apktool.jar
   ```
   
3. Framework resources

   Every Apktool release contains internally the most up to date AOSP framework at the time of the release. This allows you to decode and build most apks without a problem. However, manufacturers add their own framework files in addition to the regular AOSP ones. To use apktool against these manufacturer apks you must first install the manufacturer framework files.

   Framework files are usually a few built-in apks under */system/framework*. Nubia has two built-in framework files, namely *framework-nubia-res.apk* and *framework-res.apk*. We just need the former one for Apktool brings along an up-to-date *framework-res.apk*.

   ```bash
   ~ $ ./apktool if -t nubia /path/to/framework-nubia-res.apk 
   ```

   1. `-t nubia` sets a tag for the installed framwork which should be added to *decode*, which otherwise would report *Can't find framework resources for package of id*.
   2. By default, it's installed to *~/.local/share/apktool/framework*.
4. [Decompile/decode and build back](https://ibotpeaches.github.io/Apktool/documentation/)

   After decoding, we can find useful information on the app under *res/values-zh-rCN/string.xml*.
5. By default */tmp* is mounted as *noexec*, which would cause *apktool build* error like:

   ```
   brut.common.BrutException: could not exec: [/tmp/brut_util_Jar_1093671213752336150.tmp
   ```

   We either remount */tmp* as *exec* or refer to [aapt](https://github.com/iBotPeaches/Apktool/issues/1011). Firstly, extract directory *prebuilt/aapt* from *apktool.jar* to somewhere (i.e. *~/opt/apktool*). Then use `-a / --aapt` parameter.

   ```bash
   ~ $ ./apktool b -a ./aapt/linux/64/aapt Test-apk/
   ```

# Apps

1. ConnectBot

   Connectbot freezes after login? No response with [black screen](https://github.com/dragonlinux/connectbot/issues/493)? Please enable *Start shell session* in host edit.
2. LatinIME

   By default, I have removed default AOSP input method */system/app/LatinIME* from official ROM in favor of Google Pinyin Input. Meanwhile I choose *require PIN/password to start device* when setting lock screen PIN/password. In such case, I cannot boot the device since there is no stock input method to fill in PIN/password.

   When confronted with the issue, I can temporarily *adb push* AOSP LatinIM into system or use physical OTG keyboard.

   After booting the device, change lock screen method to *pattern* instead of *PIN/password*, which does not depend on stock LatinIME.

   **To be verified**: may be solved by adding *google-pinyin.apk* into ROM */system/app* or */system/priv-app/*, thus flashing *google-pinyin* as stock IME.
3. OsmAnd+

   *obf* maps data hidden link: [obf link 1](https://download.osmand.net/list.php), [obf link 2](https://osmand.net/list.php), and [obf link 3](http://download.osmand.net/rawindexes/). Generated download link is as:

   >http://dl5.osmand.net/download.php?standard=yes&file=Hong-kong_asia_2.obf.zip

   Hidden [Faq link](https://osmand.net/help/).

# boot/recovery img manipulation

Read [mkbootimg/unpackbootimg](https://android.stackexchange.com/a/154621). You'd better use *android_system_core* version.

# Refs

1. [adb backup](https://android.stackexchange.com/a/28315)
2. [Gentoo adb](https://wiki.gentoo.org/wiki/Android/adb)
3. [XiNGRZ](http://bbs.mfunz.com/space-uid-1339507.html)
4. [mkbootimg/unpackbootimg](https://android.stackexchange.com/a/154621)