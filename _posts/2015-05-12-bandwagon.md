---
layout: post
title: Firewall
---
# Concepts
1. The paramount is to get your own VPS (virtual private server) - a server located outside of the firewall. VPS can be bought from commercial supplier. If you possess a PC outside of the firewall, that can be treated as your own VPS. Most users don't own a physical PC outside the firewall, which is an awkward situation urging them buying VPS.
2. Then you can deploy VPN or Shadowsocks (ss) servers on your own VPS.
3. Up to now, a local VPN or ss needs installed.
4. After that, you can penetrate the firewall through VPN or ss.
5. VPN and ss are parallel services. You only need one of them unless you would like to switch between these two services for better QoS.
    1. ss支持区分国内外流量，传统VPN在翻出墙外后访问国内站点会变慢.
    2. ss use small number of memory.
6. Choose ss instead of VPN.

# VPS
There are many VPS suppliers of which I chose `bandwagonhost.com` or the so-called "搬瓦工". `bandwagonhost` is easy to manage by web portal including installing VPN or ss server, offering different kinds of pricing package specifying _RAM, DISK, BANDWIDTH etc_.

My choice is [$9.99 USD annually](https://bandwagonhost.com/cart.php?a=confproduct&i=1). Of course, it will reminds you to register your web account before paying through paypal.

Pay attention to the pricing link which is an _inviting_ link. If you buy VPS through [bandwagonhost.com](https://bandwagonhost.comm), you might not locate the _$9.99 USD annually_ pricing package.

You need to wait for a few minutes for VPS system initialization. The default VPS system is `Centos6 x86`. Your root password is not sent through email any more.
1. [bandwagonhost](https://bandwagonhost.com/clientarea.php): the web portal login. The most important page is `Services -> My Services`.
    1. You can also click on `KiviVM Control Panel` to get to the 2nd step.
2. [KiviVM Control Panel](https://kiwivm.it7.net): VPS management page. Briefly go through the management panel.
    1. The first tool I avail of is `two-factor authentication` thus another temporary code is required for each login into KiviVM.
    2. Since root password is not send through email, reset root password by `root password modification`. You can now SSH into your VPS Centos with clients like `Putty` and `MobaXterm`.

## ss server
At the bottom of `KiviVM Control Panel` lies `KiviVM Extras` from which you find `Shadowsocks Server`. What a relief! You no longer are bothered installing ss server manually. The default is Shadowsocks Python version. After installing finished, click `Go back`. Instructions on setting ss client for Windows system is illustrated.

Of course, you can also install ss server manually. Among the others, there mainly four versions of ss server: Python version, C libev version, Go version, and C++ with Qt version. Take the Python version as an example:

    ```
yum install python-setuptools && easy_install pip
pip install shadowsocks
    ```

## ss client
There are many clients available, my current windows 8.1 client is [shadowsocks-csharp](https://github.com/shadowsocks/shadowsocks-csharp).

# VPS credentials
1. [Official web portal client area](https://bandwagonhost.com/clientarea.php)
2. KiviVM Control Panel two-factor authentication:
    1. [KiviVM password](https://kiwivm.it7.net)
    2. Temporary code from `Google Authenticator`.
3. CentOS6 x86 root password
4. ss server password for ss client connectoin on `Shadowsocks Server` of KiviVM control panel.

# References
1. [server](http://shadowsocks.org/en/download/servers.html)
2. [shadowsocks](https://github.com/shadowsocks/shadowsocks)
3. [shadowsocks搭建教程](http://shadowsocks.blogspot.com/2015/01/shadowsocks.html)
4. [ShadowSocks教程:Bandwagonhost搬瓦工一键安装Shadowsocks新手小白教程](http://shadowsocks.info/shadowsocks-bandwagonhost/)
