---
layout: post
title: Init script
---

1. Most daemon (service) brings along init script on installation, like LVM, Tor, Dhcpcd etc.
2. User defines their own daemon. In this post, I show procedures to generate a Gentoo init script of *ss* proxy. Make sure it's started on boot.
3. There are two methods to add personal init script.
   1. Write a init script directly (*/etc/init.d/*).

      Init script is mainly for *daemon* in the background.
   2. Write a simple script (*/etc/local.d/*) revoked by a special init script *local* (*/etc/init.d/local*). Of course, we make sure `rc-update add local default`.

      Local script focuses on simple foreground script that executes only once. For example, write a value to a file on boot.
4. [wiki initscript](https://wiki.gentoo.org/wiki/Handbook:X86/Working/Initscripts) and [local.d](https://wiki.gentoo.org/wiki//etc/local.d).

```sh
#!/sbin/runscript
# To launch shadowsocks proxy at default runlevel
#
# https://wiki.gentoo.org/wiki/Handbook:AMD64/Working/Initscripts
#
# Tor uses Shadowsocks as frontend proxy. So launch
# it 'before tor'
#
# Put $DAEMON arguments after two succesive dashes '--'
#
# 'sslocal' create $PIDFILE by default. We just tell
# 'start-stop-daemon' the location by '--pidfile'.
#
# 'start-stop-daemon' arguments resides before '--exec'.

DAEMON="/usr/bin/sslocal"
PIDFILE="/var/run/shadowsocks.pid"

depend() {
  before tor
}

start() {
  DAEMON_OPTS="-c /etc/shadowsocks.json -q -d start --log-file /var/log/shadowsocks.log"

  if [ "${RC_CMD}" = "restart" ];
  then
    ebegin "Restarting"
    eend $?
  fi
  
  ebegin "Starting Shadowsocks"
  start-stop-daemon --start --pidfile $PIDFILE --exec $DAEMON -- $DAEMON_OPTS
  eend $?
}

stop() {
  ebegin "Stopping Shadowsocks"
  start-stop-daemon --stop --pidfile $PIDFILE --exec $DAEMON
  eend $?
}
```
