---
layout: post
title: Wireshark
---

# Installation

```bash
root@tux / # echo 'net-analyzer/wireshark androiddump' >> /etc/portage/package.use/wireshark
root@tux / # emerge -avt wireshark
```

# Permission

Running Wireshark with *root* account is dangerous. We can add normal user account into *wireshark* group.

```bash
root@tux / # gpasswd -a username wireshark
```

Usually, you should log out and log in back to take effect. *newgrp* log in to a new group:

```bash
user@tux ~ # newgrp wireshark
user@tux ~ # wireshark-gtk
```

# Tutorials

to-dos

# Android

It's recommended to install terminal emulator app such as Termux.

### [androiddump](https://www.wireshark.org/docs/man-pages/androiddump.html)

*androiddump* as explained by the term name, is used to capture Android system *logcat* messages and networking flow. Before the capture carried out, you should provide the following:

1. Android SDK tools (i.e. *adb*) are added to PATH variable (for *logcat* capture);
2. [Android *tcpdump*](https://www.androidtcpdump.com/) on a *rooted* device (for networking flow). Device vendor may have enclosed *tcpdump* in Android image.

   *tcpdump* captures Android traffic and then pass it to Wireshark on your PC. *androiddump* serves as the bridge between *tcpdump* and Wireshark.
3. Obviously, your device should be in Developer Debugging mode and recognized by SDK tools.

*tcpdump* requires root permission (rooted device), exposing to maleware attack.

### [NAT gateway](http://how-to.wikia.com/wiki/How_to_set_up_a_NAT_router_on_a_Linux-based_computer)

Set PC as gateway router to your device. Then use Wireshark to capture PC Wi-Fi interface.

Here is my case. Linux and Android connect to the same Wi-Fi as follows:

Device | IP
--- | ---
Linux | 192.168.0.100
Android | 192.168.0.151
Router | 192.168.0.1
WAN | 57.194.2.1

By default Linux and Android uses gateway *192.168.0.1*. We can check route by:

```bash
user@tux ~ $ route -n
user@tux ~ $ ip route
```

Firstly, we set Android Wi-Fi gateway to Linux's IP *192.168.0.150*. Thus all Android Wi-Fi data routes through Linux.

Then on Linux, we should configure NAT for Android traffic, forwarding Android traffic to the real Wi-Fi gateway *192.168.0.1*.

```bash
user@tux ~ $ sysctl -w net.ipv4.ip_forward=1
user@tux ~ $ iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE (poor)
# or
user@tux ~ $ iptables -t nat -A POSTROUTING -o wlan0 -j SNAT --to-source 192.168.0.150 (better)
user@tux ~ $ iptables -A FORWARD -i wlan0 -j ACCEPT
```

1. NAT requires Ip forwarding that, by default, is disabled in Linux boxes.
2. MASQUERADE replaces the sender's (Android device *192.168.0.151* here) address by the router's (Linux *192.168.0.150* here) address.

   As the router here is Linux PC, we'd [better](https://unix.stackexchange.com/q/21967) use SNAT istead. Bascially, we use SNAT for static router IP, improving Iptables performance. However, we must opt MASQUERADE when the router connects to the Internet with PPoE dial-up (different WAN IP) each dial-up.
3. This is a special case of the reference as there is only one Linux interface (*wlan0*) involved.

To check the setup, make sure Android connect to network (ping in Termux). Further more, we can check Iptables counts:

```bash
user@tux ~ $ iptables -L -nv [-t nat]
```

Finally, Wireshark can capture Android traffic on PC. We can print out only Android traffic by filtering the Android IP:

>ip.addr == 192.168.0.151

### Proxy

Set up a proxy (i.e. OpenVPN, Privoxy HTTP etc.) on PC and proxy device traffic through.

### Alternatives

1. tPacketCapture app based on Android VPN service.
2. [AndroidHttpCapture](https://github.com/JZ-Darkal/AndroidHttpCapture) app for HTTP(s) capture.
3. We can use Charles and Fiddler for HTTP(s) capture.

Solutions based on VPN service can capture transmission layer packets. Most tools above only capture HTTP(s) layer traffic.
