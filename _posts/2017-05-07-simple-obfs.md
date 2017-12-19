---
layout: post
title: simple-obfs
---

# ABC

1. As [ptproxy](/2016/04/12/ptproxy/) is almost dead and shadowsocks advances further, we require a newer traffic obfuscation tool, where [simple-obfs](https://github.com/shadowsocks/simple-obfs) comes to sight.
2. *simple-obfs* is similar to Tor Pluggable Transport (i.e. obfs4proxy) in disguising abnormal traffic as daily HTTP(s), thus free from firewall interference.
3. This post prefers *standalone* mode.
4. The author does not provide information on configuration file format.

   Please check *jconf_t * read_jconf (const char *file)* function within *src/jconf.c*.

# Build

I will show how to build binary in Gentoo. Before that, make sure dependency packages are installed.

```bash
~ $ cd ~/opt/
~ $ git clone --depth=1 https://github.com/shadowsocks/simple-obfs
~ $ cd simple-obfs/
~ $ git submodule update --init --recursive
~ $ ./autogen.sh
~ $ ./configure [--prefix=/home/username/opt] && make
~ $ make install prefix=~/opt
# or
~ # cp src/{obfs-local,obfs-server} /opt/bin
```

1. By default, binaries, doc, man are installed into */usr/local/{bin,doc,man}*. `configure --prefix=/foo/bar` changes to */foo/bar*.

   Attention, do not put tailing slash to `--prefix=` argument.
2. To be simple, *obfs-local* and *obfs-server* binaries are enough. Just copy to `PATH`.

Finally, to clean letfovers:

```bash
~ $ cd ~/opt/simple-obfs
~ $ make clean
~ $ make distclean
~ $ git status
```

# Server unit

```
[Unit]
Description=simple-obfs server
Requires=shadowsocks-libev.service
After=network.target shadowsocks-libev.service

[Service]
Type=simple
PermissionsStartOnly=false
User=nobody
Group=nobody
LimitNOFILE=32768
ExecStart=/usr/local/bin/obfs-server -c /etc/shadowsocks-libev/obfs-server.json

[Install]
WantedBy=multi-user.target
```

## Server json

```json
{
    "server":"0.0.0.0",
    "server_port":1235,
    "dst_addr":"127.0.0.1:1236",
    "timeout":100,
    "obfs":"tls",
    "failover":"www.bing.com",
    "fast_open":true
}
```

1. Listen on port 1235 for *obfs-client* connection;
2. Forward traffic to *shadowsocks* on port 1237;
3. Obfuscation method is *http* or *tls*.
4. Fail over to public domain or personal web server.

# Client initd

```
#!/sbin/openrc-run
#
# 'start-stop-daemon' arguments resides before '--exec' while
# "${OBFS_COMMAND}" arguments after two succesive dashes '--'.
#
# Drop to 'nobody' user.

USER="nobody"

OBFS_COMMAND="/opt/bin/obfs-local"
OBFS_PIDFILE="/var/run/simple-obfs.pid"
OBFS_CONFILE="/etc/shadowsocks-libev/obfs-local.json"

depend() {
  after dhcpcd
}

start() {
  if [ "${RC_CMD}" = "restart" ];
  then
    ebegin "Restarting obfs-local"
    eend $?
  fi

  ebegin "Starting obfs-local"

  # start-stop-daemon -S -x "${OBFS_COMMAND}" -u "${USER}" -b -m -p "${OBFS_PIDFILE}" -- -c "${OBFS_CONFILE}"
  start-stop-daemon -S -x "${OBFS_COMMAND}" -- -c "${OBFS_CONFILE}" -a "${USER}" -f "${OBFS_PIDFILE}"

  eend $?
}

stop() {
  ebegin "Stopping obfs-local"

  start-stop-daemon -K -p "${OBFS_PIDFILE}"

  eend $?
}
```

## Client json

```json
{
    "server":"vps-ip",
    "server_port":1235,
    "local_address":"127.0.0.1",
    "local_port":1234,
    "timeout":100,
    "obfs":"tls",
    "obfs_host":"www.bing.com",
    "fast_open":true
}
```

1. Notice that *server* and *server_port* remain unchanged irrespective of client or server side.

# Notes

1. Make sure Iptables allows relevant ports and IPs.
2. On Android, first install *simple-obfuscation* app. Afterwards, Change profile port to that of *obfs-server*. Obviously, you should set *Configure* part to relevant values.

   You may got *[unknown](https://github.com/shadowsocks/shadowsocks-android/issues/1428) plugin obfs-local* error after the exact first run. Please turn on *autostart* permission.
