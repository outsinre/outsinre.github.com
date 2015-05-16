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

You need to wait for a few minutes for VPS system initialization. The default VPS system is `CentOS6 x86`. You can also reinstall OS after which a new root passowrd and port for SSH are generated as well.

1. [bandwagonhost](https://bandwagonhost.com/clientarea.php): the web portal login. The most important page is `Services -> My Services`.
    1. You can also click on `KiviVM Control Panel` to get to the 2nd step.
2. [KiviVM Control Panel](https://kiwivm.it7.net): VPS management page. Briefly go through the management panel.
    1. The first tool I avail of is `two-factor authentication` thus another temporary code is required for each login into KiviVM.
    2. Since CentOS6 x86 root password is not send through email any more, reset root password by `root password modification`. You can now SSH into your VPS Centos with clients like `Putty` and `MobaXterm`.
	3. Under `KiviVM password modification`, set password for [KiviVM Control Panel](https://kiwivm.it7.net).

## ss server Python version
1. At the bottom of `KiviVM Control Panel` lies `KiviVM Extras` from which you find `Shadowsocks Server`. What a relief! You no longer are bothered installing ss server manually. The default is Shadowsocks Python version. After installing finished, click `Go back`. Instructions on setting ss client for Windows system is illustrated. ss server will run automatically after the automatic installation.
    1. Command `ps -ef | grep ssserver` will print the ss server command:

        ```
root       415     1  0 01:21 ?        00:00:00 /usr/bin/python /usr/bin/ssserver -p 443 -k PASSWORD -m aes-256-cfb --user nobody --workers 2 -d start
nobody     417   415  0 01:21 ?        00:00:02 /usr/bin/python /usr/bin/ssserver -p 443 -k PASSWORD -m aes-256-cfb --user nobody --workers 2 -d start
nobody     418   415  0 01:21 ?        00:00:02 /usr/bin/python /usr/bin/ssserver -p 443 -k PASSWORD -m aes-256-cfb --user nobody --workers 2 -d start
root       822   708  0 07:03 pts/0    00:00:00 grep ssserver	
        ```
    2. `ssserver -h` will show help information
    3. `-d` means run as a daemon in the background.
    4. `--workers 2` will generate two processes belong to user `nobody`
    5. This automatic method does not run `ssserver` with a configuration file, but with bare command line arguments.
    6. ss server was set to run at boot. Let's check:

{% highlight linenos %}
    [root@localhost ~]# ls /etc/rc.local -al
    lrwxrwxrwx 1 root root 13 May 16 00:48 /etc/rc.local -    > rc.d/rc.local
    
    [root@localhost ~]# cat /etc/rc.d/rc.local
    \#!/bin/sh
    \#
    \# This script will be executed *after* all the other init scripts.
    \# You can put your own initialization stuff in here if you don't
    \# want to do the full Sys V style init stuff.
    
    touch /var/lock/subsys/local
    
    /usr/bin/ssserver -p `cat /root/.kiwivm-shadowsocks-port` -k `cat /root/.kiwivm-shadowsocks-password` -m `cat /root/.kiwivm-shadowsocks-encryption` --user nobody --workers 2 -d start
{% endhighlight %}
	Pay attention to the command line arguments are stored in `/root/.kiwivm-shadowsocks-*` files.
2. Of course, you can also install ss server manually. Among the others, there mainly four versions of ss server: Python version, C libev version, Go version, and C++ with Qt version. Take the Python version as an example:

    ```
yum install python-setuptools && easy_install pip
pip install shadowsocks
    ```
    1. `/usr/bin/python /usr/bin/ssserver -p 443 -k PASSWORD -m aes-256-cfb --user nobody --workers 2 -d start` as step 1.
    2. `ssserver` can also run with `-c` parameter to specify a configuration file rather than command parameters.
    3. `ssserver -c /path/to/shadowsocks.json -d start` for example.
	4. A simple configuration file `/etc/shadowsocks.json`:

        ```
{
“server”:”0.0.0.0″,
“local_address”: “127.0.0.1”,
“local_port”:1080,
“server_port”: 8388,
“password”: “password”,
“timeout”:60,
“method”:”aes-256-cfb”,
“fast_open”: false,
“workers”: 2
}
        ```
    4. Support multiple users:
	
        ```
{
“server”:”0.0.0.0″,
“local_address”: “127.0.0.1”,
“local_port”:1080,
“port_password”:
{
    “8388”: “password8388″,
    “8398”: “password8398″,
    “8418”: “password8418″
  },
“timeout”:60,
“method”:”aes-256-cfb”,
“fast_open”: false,
“workers”: 2
}
        ```
3. Shadowsocks server `ssserver`并没有加入到开机启动，如果需要则要创建一个启动脚本，使其开机启动。
3. Refer to [shadowsocks 2.6.8](https://pypi.python.org/pypi/shadowsocks); [VPS之自建shadowsocks服务器（Centos及Ubuntu方法）](http://www.vtestvps.tk/?p=18)

## ss client
There are many clients available, my current windows 8.1 client is [shadowsocks-csharp](https://github.com/shadowsocks/shadowsocks-csharp).

Fill in the `encryption method`, `server port`, `password`, and `proxy port` for client. The default proxy mode is `PAC` (Proxy auto-config) not `Global`.

### PAC VS Global
pac是只对被墙的使用ss，全局就是无论什么网站都用ss。

pac可以自己修改，添加任意网站。当然也可以用网上网友维护的文件，最有名的是就是GFWlist列表。windows下右键点击ss client，出现一个菜单"Update PAC from GFWlist" 。

说的简单点，就是一个被Q网址收集汇总，只要配对上就会走代理，没有配对上就不走代理，这样子就节省了一些流量，包括上网速度等问题。

参考[ShadowSocks教程:SS软件中的pac自动代理模式是什么？](http://shadowsocks.info/shadowsocks-pac/)

# VPS credentials
1. [Official web portal client area](https://bandwagonhost.com/clientarea.php)
2. KiviVM Control Panel two-factor authentication:
    1. [KiviVM password](https://kiwivm.it7.net)
    2. Temporary code from `Google Authenticator`.
3. CentOS6 x86 root password: ssh into CentOS for OS-specific management.
4. ss server password for ss client connectoin on `Shadowsocks Server` of KiviVM control panel.

# References
1. [shadowsocks.org](http://shadowsocks.org)
2. [shadowsocks github](https://github.com/shadowsocks/shadowsocks)
3. [shadowsocks搭建教程](http://shadowsocks.blogspot.com/2015/01/shadowsocks.html)
4. [ShadowSocks教程:Bandwagonhost搬瓦工一键安装Shadowsocks新手小白教程](http://shadowsocks.info/shadowsocks-bandwagonhost/)

# Notes
1. SwichSharp is no longer needed.
2. If VPS CentOS restarted, then make sure to run `ssserver` again as a daemon if you don't set it run on boot.

# Issues remained
3. VPN
4. ss优化
