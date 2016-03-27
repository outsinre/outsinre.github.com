---
layout: post
title: Gentoo Networking
---

>Discuss networking configuration in Gentoo system, covering *net.\** config, dhcpcd, wpa_supplciant, mac spoofing etc.

# ABCs

1. Wireless, PPP etc. special networking needs *extra configuration* before DHCP or static IP.

   For example, before the wireless interface request IP from DHCP server, it must first authenticate username and password with wireless router. Otherwise, it was not even attched to a physical link. This functionality can be achieved by tools like *wpa\_supplicant* and/or *wireless-tools*.
2. The Gentoo traditional *net config* is */etc/init.d/net.\** and */etc/conf.d/net*.
3. A functionality is not constrined to a specific tool.

   For instance, the wireless authentication functionality can be also achieved by *wcid*, *networkmanager* etc. as they all draw in *wpa\_supplicant* package.
4. A tool is not constrained to a specific funciontality. Most of the time, we only need a few of them.

   For example, *dhcpcd* can handle all networking functinalities.
5. If using *net config* or *dhcpcd*, we should install *wpa\_supplicant* manually to satisfy wireless authentication.

# Simple config

The simplest config on my system is *dhcpcd* + *wpa\_supplicant*, though *net config + wpa\_supplicant* is fine. For Ethernet, *dhcpcd* is enough! The extra *wpa\_supplicant* fullfils wireless authentication.

1. Emerge both packages;
2. Add *dhcpcd* to *default* runlevel; Don't add *wap\_supplicant* to any runlevel.
3. Conigure *wpa_\supplicant.conf* for authentication.

Bingo~

## *dhcpcd*

*dhcpcd* brings along its own *hooks* (*/lib/dhcpcd/dhcpcd-hooks/* and */usr/share/dhcpcd/hooks/*), like *resolv.conf*, *wpa\_supplicant* etc. The hook tells *dhcpcd* to launch a relevant service. For example, *wpa\_supplicant* hooks instruct *dhcpcd* to launch *wpa_supplicant* functionality before DHCP session.

Now, *dhcpcd* won't enable some hooks (i.e. *wpa_\supplicant* ) by default, we need to copy the hook from */usr/share/dhcpcd/hooks/* to */lib/dhcpcd/dhcpcd-hooks/*. To disable a hook is trivial - either remove the hook under */lib/dhcpcd/dhcpcd-hooks/* or append `nohook hook-name1, hook-name2` to */etc/dhcpcd.conf*.

Here is an excerpt of *dhcpcd* config:

```
# stop resolv.conf hook to alter /etc/resolv.conf file since we will use TorDNS
# Another method is to remove /lib/dhcpcd/dhcpcd-hooks/20-reslve.conf hook
#
# Let net.wlan0 handle wpa_supplicant
nohook resolv.conf, wpa_supplicant

# Speed up DHCP by disabling ARP probing. This is useful in home network where IP collision
# rarely happens
noarp

# dhcpcd will release the lease prior to stopping the interface
# I use macchanger to randomize mac address. If without this option,
# the lease pool will burn out - NO IPs available!
release
```

The example above disables *wpa\_supplicant* hook, which implies *dhcpcd* won't launch *wpa\_supplicant* functionality.

# MAC Spoofing

Now we do some trick on networking control. Anonymity on network is important. We can hide our IP/identity on the Internet by *Tor*, *NAT* etc. However that does not work within LAN due to MAC address!

Though MAC address won't go out of LAN, but it's plain within that scope. Enveryone in LAN can identify you by MAC address, thus motivating MAC Spoofing. Each time, we spoof a MAC address, the DHCP servers treat it as a new joint device and asign a new IP from the pool.

*Notice*: xx:xx:xx:xx:xx means permanent address while yy:yy:yy:yy:yy:yy is fake address.

## Manually

Manually means change MAC address on command line. Pior to spoofing, we should make sure the interface id down. Basically, we resort to:

1. ifconfig

   ```bash
   ifconfig wlan0 down
   ifconfig wlan0 hw ether XX:XX:XX:XX:XX:XX
   ifconfig wlan0 up
   ```

   Some drivers might not support `hw` option.
2. iproute2

   ```
   ip link set dev wlan0 down
   ip link set dev wlan0 address XX:XX:XX:XX:XX:XX
   ip link set dev wlan0 up
   ```
   
3. macchanger

   The former two are almost the same except the syntax, while *macchanger* is smarter with fine control. *macchanger* is a package capable of complex MAC spoofing like *reserve vendor type*, *any vendor*, *fully random* etc.

   ```
   emerge -avt macchanger
   emerge -avt --oneshot >=netifrc-0.2.3
   ```

   *macchanger* has bumped to verion 1.7.0 which returns different string on exit. However, *<netifrc-0.2.3* still examines the old exit string, error-prone.

   ```
   ip link set dev wlan0 down
   machanger -A wlan0
   ip link set dev wlan0 up
   ```

   Details refer to man page.
   
## Automation

We prefer autmatic spoofing on system boot without user intervence, saving much trouble. Openrc *init script* meets the requirement. We wrap *ifconfig*, *ip* or *macchanger* command in init script.

We assume the networking scheme is *dhcpcd + wpa\_supplicant*.

*Notice*: we should test our init script, `rc-service script-name stop` and `rc default` (or `rc`). Don't reboot - a waste of time!

1. standard init script

   ```
   #!/sbin/runscript
   #
   # Spoof wireless interface mac address on boot
   # Add other interfaces if needed
   #
   # `macchanger' supports complex syntax compared
   # to `ip'

   ETH0_PERMANET_MAC="xx:xx:xx:xx:xx:xx"
   ETH0_FAKE_MAC="yy:yy:yy:yy:yy:yy"

   WLAN0_PERMANENT_MAC="xx:xx:xx:xx:xx:xx"
   WLAN0_FAKE_MAC="yy:yy:yy:yy:yy:yy"

   depend() {
     after udev
     before dhcpcd
   }

   start() {
     if [ "${RC_CMD}" = "restart" ];
     then
       ebegin "Spoofing again"
       eend $?
     fi

     ebegin "Spoofing MAC Address"

     ip link set dev eth0 down
     #ip link set dev eth0 address $ETH0_FAKE_MAC
     macchanger -A eth0
     ip link set dev eth0 up

     ip link set dev wlan0 down
     #ip link set dev wlan0 address $WLAN0_FAKE_MAC
     macchanger -A wlan0
     ip link set dev wlan0 up

     eend $?
   }

   stop() {
     ebegin "Oops! Not a daemon and do nothing"
     eend $?
   }
   ```

   Since we specify *before dhcpcd* in *dpend*, the *down/up* wrapper can be removed unless you need manually `rc-service macspoofing restart`.
2. local init script

   Similarly, MAC spoofing can be accomplished in a ordinary local script in */etc/local.d/*. Actually MAC spoofing is NOT a daemon service but a few shell commands. *local init script* fits better.

   */etc/local.d/macspoofing.start*:
   
   ```
   #!/bin/sh

   ETH0_PERMANET_MAC="xx:xx:xx:xx:xx:xx"
   ETH0_FAKE_MAC="yy:yy:yy:yy:yy:yy"

   WLAN0_PERMANENT_MAC="xx:xx:xx:xx:xx:xx"
   WLAN0_FAKE_MAC="yy:yy:yy:yy:yy:yy"

   echo "Spoofing eth0 MAC Address"
   ip link set dev eth0 down
   #ip link set dev eth0 address $ETH0_FAKE_MAC
   macchanger -A eth0
   ip link set dev eth0 up

   echo "Spoofing wlan0 MAC Address"
   ip link set dev wlan0 down
   #ip link set dev wlan0 address $WLAN0_FAKE_MAC
   macchanger -A wlan0
   ip link set dev wlan0 up
   ```

   1. Please make sure *local* init is added to *default* runlevel: `rc-update add local default`.
   2. Also mark the script as executable.
   3. Must provide the *down/up* wrapper since we can not guarantee the interface is down when local script is executed.
   4. We can also revoke the script by `rc-service local start` or `rc`.

3. udev init script

   Udev allows to perform MAC address spoofing by creating the udev rule. Use *address* attribute to match the interface by its original MAC address and change it using the *ifconfig/ip/macchanger* command:

   ```
   # /etc/udev/rules.d/75-mac-spoof.rules

   ACTION=="add", SUBSYSTEM=="net", ATTR{address}=="XX:XX:XX:XX:XX:XX", RUN+="/usr/bin/ip link set dev %k address YY:YY:YY:YY:YY:YY"
   or
   ACTION=="add", SUBSYSTEM=="net", ATTR{address}=="XX:XX:XX:XX:XX:XX", RUN+="/usr/bin/macchanger -A %k"
   ```

   Make sure Udev is added to a init runlevel. On Gentoo, it's by default at *sysinit* runlevel.

   Udev is a low level but powerful device control module, we can do more. For example, create another rule file to rename the interface.
4. net config - discusses later

## Release

When requiring IP by DHCP, we MUST tell DHCP client to *release previous lease when it stops* for IP re-use. Otherwise, eventually all available IPs are reserved by our single interface - running out of IP!

How to release? If use *dhcpcd* to serve DHCP, add `release` option to *dhcpcd.conf*. For traditional *net* config, add `dhcpc_eth0="release"` to */etc/conf.d/net*.

## net config

The above spoofing method assumes *dhcpcd + wpa\_supplicant* networking scheme. Actually the old *net config* now supports MAC spoofing with *macchanger*. This is new networking scheme:

   Now *net config* + *dhcpcd* + *wpa\_supplicant* new networking scheme.

1. Symbolic

   ```
   pushd /etc/init.d/
   ln -sv net.lo net.wlan0
   rc-update add net.wlan0 default
   ```
   
2. */etc/conf.d/net*

   ```
   ## global modules preference

   # iproute2 over ifconfig
   # both are for static IP setting
   modules="iproute2"

   # DHCP: over 'dhclient' / 'pump' / 'udhcpc' etc.
   # For dynamic IP setting
   modules="dhcpcd"

   # Wireless: over 'wireless-tools' etc.
   # For wireless authentication
   modules="wpa_supplicant"


   ## ethernet - eth0

   # modules preference
   modules_eth0="dhcpcd"

   # null - no IP
   config_eth0="null dhcp"
   dhcp_eth0="release"

   # mac spoofing
   # To randomize between the same physical type of connection (e.g. fibre,
   # copper, wireless) , all vendors
   mac_eth0="random-anykind"


   ## wireless - wlan0

   # modules preference
   modules_wlan0="wpa_supplicant"

   # dhcp
   config_wlan0="dhcp"
   dhcp_wlan0="release"

   # mac spoofing
   # To randomize between the same physical type of connection (e.g. fibre,
   # copper, wireless) , all vendors
   mac_wlan0="random-anykind"

   ```

   `mac_wlan0="random-anykind" actually revokes `macchanger -A wlan0`.
3. Update *dhcpcd* config
   1. Keep *dhcpcd* at *default* runlevel though *net config* can launch *dhcpcd* process automatically!

      But this lanuching is not a *daemon* but a normal process. If revoked as a normal process, then `rc-service net.wlp3s0 restart` won't work due to failure stop of the *dhcpcd* process.

      The key is we still ned *dhcpcd* to serve DHCP functionality.
   2. Disable *wpa\_supplicant* hook since *net config* will lanuch *wpa\_supplicant* functinality automatically.

      *net config* will definitely launch *wpa\_supplicant* but *dhcpcd* will launch *wpa\_supplicant* as well. Thus *wpa\_supplicant* is revoked twice, causing error.
   3. Keep the *release* option in *dhcpcd.conf*.
4. It seems that *net config* does NOT spoof MAC address if is no connection (i.e. no wired cable), which is a preferred way.
5. More

   I can set an interface down by default. For example, to let Ethernet eth0 down default, in *net config*, we can add *config_eth0="null"*. And in *dhcpcd.conf*, add *denyinterfaces eth0*.
## Summary

The *Udev init script* is the easiest method.

## Refs

1. [archi wiki](https://wiki.archlinux.org/index.php/MAC_address_spoofing)
2. [what why how](https://perot.me/mac-spoofing-what-why-how-and-something-about-coffee)
3. [gentoo forum](https://forums.gentoo.org/viewtopic-p-7737632.html)

# Refs

1. [gentoo wiki](https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Networking)
