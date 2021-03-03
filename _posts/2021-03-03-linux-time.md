---
layout: post
title: Linux Time
---

>VirtualBox guest OS relies on VirtualBox guest additions to synchronizes time with host. Hence leave this part after VirtualBox guest additions.

In an operating system, the time (clock) is determined by four parts: time value, time standard, time zone, and Daylight Saving Time (DST) if applicable. Especially, RTC is just a bare value on board without any other information attached. It's the time managment tools that control time standard, time zone and DST.

*timedatectl* is used to control system date and time while *hwclock* is for hwardware time (RTC). The use of *timedatectl* requires an active dbus. Therefore, it may not be possible to use this command under a *chroot* (i.e. during installation). In such cases, you can revert back to the *hwclock* command or wait for booting into the new system.

As stated before, dual boot with Windows should set Linux system to treat RTC as *localtime*. If this installation resides in virtual machine, then leave it default.

Check time:

```bash
[root@host ~ #] timedatectl set-ntp true
[root@host ~ #] timedatectl status
```

Here is an example of *timedatectl* output:

```
                      Local time: Thu 2017-09-21 16:08:56 CEST
                  Universal time: Thu 2017-09-21 14:08:56 UTC
                        RTC time: Thu 2017-09-21 14:08:56
                       Time zone: Europe/Warsaw (CEST, +0200)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```

Don't be confused. Both *Local time* and *Universal time* are system time but with different time standards. The third one (RTC) is hardware time value. Obviously, RTC in this example is treated as Universal Time, which can be verified by the last line - RTC in local TZ: no

Set time:

```bash
[root@host ~ #] hwclock [--systohc | --hctosys] [--utc | --localtime]
[root@host ~ #] timedatectl set-local-rtc 0/1
```
