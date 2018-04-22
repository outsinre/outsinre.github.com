---
layout: post
title: WireGuard
---

* ToC
{:toc}

[WireGuard](https://www.wireguard.com/) claims to utilizes state-of-the-art cryptography (Noise protocol framework, Curve25519, ChaCha20, Poly1305, BLAKE2, SipHash24, HKDF etc.) while provide a simple construction.

It adds a virtual network interface and IP thereof. The interface is associated with a public/private key pair of which the public part serves as that interface's identity within WireGuard network. WireGuard encapsulates IP packets (application data payload) over UDP and forwards packets based on Cryptokey Routing.

Originally, WireGuard is integrated into Linux kernel (module or built-in tree). Nowadays, it's under heady development to be cross-platform and widely deployable.

>The 6th reference (NAT-to-NAT VPN with WireGuard) is worth of reading.

# Installation

To install as an external module:

```bash
root@tux ~ # emerge -avt wireguard openresolv
root@tux ~ # modinfo wireguard
```

Optionally, we can build WireGuard as an [internal kernel module or as built-in](https://www.wireguard.com/install/) with `module-src` USE.

*wireguard* (by *wg-quick*) depends on *openresolv* when we define custom DNS for WireGuard tunnel.

# Conceptual Overview

Before everything else, we should grasp the conceptual overview.

Peer | A | B
---|---|---
External IP | 192.168.0.123 | 23.45.67.89
UDP port | 48574 | 39814
Internal IP | 10.0.0.1/24 | 10.0.0.2/24

External IP (i.e. *eth0*) is given by ISP or Wi-Fi router, without which you cannot reach the Internet. On the other hand, internal IP manually assigned to WireGuard interface is only privately valid within WireGuard network. UDP port associated with external IP is where WireGuard service listens for traffic.

WireGuard does not assume server side or client side. They differ in which initiates traffic sending. For consistent terminology, I regard peer (B) with public IP as server side while peer (A) censorshipped by firewall as client side.

Clearly, Peer A, on the initiaive, opens connections to peer B establishing a point-to-point tunnel. With a few arguments adjustment, peer A reaches the whole *10.0.0.0/24* LAN blocks (even machines without WireGuard) via B - the *gateway*. A step further, peer A can route all the Internet traffic through B.

Firstly, we start off by point-to-point link. Without explicit notice, the following steps should be executed on both sides.

# Interface

Create Wireguard interface with *ip* that would automatically load *wireguard* module:

```bash
root@tux ~ # modprobe -v wireguard (opt)
root@tux ~ # ip link add dev wg0 type wireguard
root@tux ~ # ip link
```

Internal IP assignment:

Unless you have a really good reason, please assign internal IPs within the same subnet for client and server sides. Otherwise, you want extra route on both sides.

```bash
root@tux ~ # ip addr add 10.0.0.1/24 dev wg0
root@tux ~ # ip addr
```

# Key pair

Permission:

```bash
root@tux ~ $ umask 077
root@tux ~ $ mkdir -p /etc/wireguard/keys; cd /etc/wireguard/keys/
```

Private/public keys:

```bash
root@tux ~ $ wg genkey > private-key; cat private-key
root@tux ~ $ wg pubkey < private-key > public-key; cat public-key
```

Optionally, this can be done all at once:

```bash
root@tux ~ $ wg genkey | tee private-key | wg pubkey > public-key
```

# WireGuard setup

Arguments can be loaded from file.

```bash
root@tux ~ # wg setconf wg0 /etc/wireguard/wg0.conf
# -or-
root@tux ~ # wg set wg0 listen-port 48574 private-key /etc/wireguard/private-key
root@tux ~ # wg set wg0 peer <peer B public key> [persistent-keepalive 25] [endpoint 23.45.67.89:39814] allowed-ips 10.0.0.2/32,10.0.100.2/24
root@tux ~ # wg [showconf wg0]
```

1. *peer B public key* should be *base64* format like *V7g3kxzLATJ6edBybau1IrE3FOgLHajxxFfMZ+QOUyE=*.
2. To traverse NAT or firewall, we need *persistent-keepalive*.

   It's the peer behind NAT or firewall that associates *persistent-keepalive* value with its peer setting.

   For instance, A sites behind NAT (i.e. Wi-Fi router), *persistent-keepalive* sends a packet to its peers periodically to keep NAT mapping open, without which, peers cannot send packets A actively though it is not the common case.
3. *endpoint* is peer B's external yet public IP and UDP port.

   It's recommended to leave this argument out on server side because the server updates the endpoint of its peers by examining from where correctly authenticated packets originates. Only client side that makes the initial connection requires *endpoint*.

   As peer A sits behind Wi-Fi router, relevant *endpoint* should be WAN IP instead!
4. *allowed-ips* is a list of comma-separated IP ranges to which WireGuard traffic can be sent and from where WireGuard traffic can be recevied.

   For simple point-to-point connection, it should be peer B's internal IP.

   The catch-all *0.0.0.0/0* may be specified for matching all IPv4 addresses, and *::/0* may be specified for matching all IPv6 addresses, allowing and route *all* traffic on the client through the VPN tunnel. This is useful if want to set a VPN proxy.

   This can be narrowed down if you only want some traffic (i.e. remote LAN network *10.0.0.0/24*) to go over VPN.

   >Peer B's internal IP must be covered by *allowed-ips*.

5. Multiple peers can be set up.

## Persistent Configuration

Previously *wg* commands are valid within current login session. To be permanent across reboots, arguments could be loaded from file which, by convention, locates under */etc/wireguard/*.

To save and load configuration:

```bash
root@tux ~ # wg showconf wg0 > /etc/wireguard/wg0.conf
root@tux ~ # wg setconf wg0 /etc/wireguard/wg0.conf
```

Here is the example:

```
[Interface]
ListenPort = 48574
PrivateKey = EOpLmUJ08uwH2z2NpBwJ2upWzB5Tn36nhQvlXccAFnk

[Peer]
PublicKey = 7nXZYaqyXuVdW0RwHauMJuW81axAM9hSavY9JIJsZFU=
AllowedIPs = 10.0.0.2/32
Endpoint = 23.45.67.89:39814
```

# Final *ping*

## Link up

```bash
root@tux ~ # ip link set up dev wg0
root@tux ~ # ip link; ip addr
root@tux ~ # ss -npelu
```

You can find UDP sockets created.

## Route

When internal IP is assigned, a system route is added automatically for that IP range like:

> 10.0.0.0/24 dev wg0 proto kernel scope link src 10.0.0.1

If *allowed-ips* (i.e. peer B's internal IP *192.168.58.1/32*) belongs to different subnets (and vice versa). They cannot connect to each other unless extra routes for *allowed-ips* on both sides are created.

```bash
# client side
root@tux ~ # ip route add 192.168.58.0/24 dev wg0 proto kernel scope link
# server side
root@tux ~ # ip route add 10.0.0.0/24 dev wg0 proto kernel scope link
```

If you would like a strict route, the IP range could be narrowed down to IP address like:

```bash
root@tux ~ # ip route add 192.168.58.1/32 dev wg0 proto kernel scope link
```

## Iptables

Please make sure relevant UDP ports and IPs are accepted by firewall. By far, a point-to-point link is established. We can check by *ping* each other's internal IP.

```bash
root@tux ~ # iptables -A OUTPUT -d 23.45.67.89/32 -p udp -m udp --dport 39814 -m conntrack --ctstate NEW -j ACCEPT
root@tux ~ # iptables -A INPUT -s 23.45.67.89/32 -p udp -m udp --sport 39814 -m conntrack --ctstate NEW -j ACCEPT
```

# [LAN access](https://www.ericlight.com/wireguard-part-two-vpn-routing.html)

Up to now, we have successfully established point-to-point connection between two peers. Roughly speaking, we can reach only peers' internal IPs. To reach the whole remote LAN network, we just tune a few arguments.

## Remote arguments

On the remote peer, turn on IPv4 packet forwarding on remote peer:

```bash
root@tux ~ # sysctl -w net.ipv4.ip_forward=1
# consistent across reboot
root@tux ~ # echo 'net.ipv4.ip_forward=1' > /etc/sysctl.d/20-ip_forward.conf
root@tux ~ # sysctl -p /etc/sysctl.d/20-ip_forward.conf
```

Afterwards, we enable ProxyARP on *wg0*:

```bash
root@tux ~ # sysctl -w net.ipv4.conf.all.proxy_arp=1
# -or-
root@tux ~ # sysctl -w net.ipv4.conf.wg0.proxy_arp=1
# consistent across reboot
root@tux ~ # echo 'net.ipv4.conf.wg0.proxy_arp=1' > /etc/sysctl.d/25-proxy_arp.conf
root@tux ~ # sysctl -p /etc/sysctl.d/25-proxy_arp.conf
```

## Local arguments

All we need to midify is *allowed-ips* to cover the whole LAN "10.0.0.0/24* instead of *10.0.0.2/32*.

Then we can connect to the remote LAN through WireGuard.

# [VPN proxy](https://www.wireguard.com/netns/)

Based on LAN access, we can route all local traffic through WireGuard tunnel for whatever reasons. Similarly, we want to tune a few arguments.

## Remote arguments

Mostly, we should add two *iptables* rules to allow FORWARDING from *wg0* to *eth0* interface and set up NAT. That is to say, we set remote peer as [gateway](/2017/12/29/linux-as-gateway-router/) to the Internet.

```bash
root@tux ~ # iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT
root@tux ~ # iptables -t nat -A POSTROUTING -i wg0 -o eth0 -s 10.0.0.0/24 -j MASQUERADE
# -or-
root@tux ~ # iptables -t nat -A POSTROUTING -i wg0 -o eth0 -s 10.0.0.0/24 -j SNAT --to-source <ip-of-eth0>
```

We assume *eth0* is the remote interface that has Internet access.

>Please make sure *net.ipv4.ip_forward* is turned as stated above.

## Local arguments

To route all local traffic through the tunnel, we mainly resort to system routes. Specifically, we should route traffic through *wg0* instead of the default except traffic to server peer's external IP.

The key is to bypass *default* in the first place and route traffic to *wg0* first.

### Replacing the default route

```bash
user@root ~ # ip route del default
user@root ~ # ip route add default dev wg0
user@root ~ # ip route add 23.45.67.89/32 via 192.168.0.1 dev wlan0
```

### Override the default route

Override the default with two more specific rules that *add up in sum* to the default, but match before the default. We split the default into *0.0.0.0/1* and *128.0.0.0/1*.

```bash
user@root ~ # ip route add 0.0.0.0/1 dev wg0
user@root ~ # ip route add 128.0.0.0/1 dev wg0
user@root ~ # ip route add 23.45.67.89/32 via 192.168.0.1 dev wlan0
```

### Rule-based routing

>Without explicit *priority* value, new rules are **prepended** the old ones.

A new route table is created for *wg0*.

```bash
user@root ~ # ip rule add to 23.45.67.89/32 lookup main pref 30
user@root ~ # ip rule add to all lookup 80 pref 40
user@root ~ # ip route add default dev wg0 table 80
```

#### wg-quick method

A variant version used by *wg-quick*:

```
user@root ~ # wg set wg0 fwmark 51820
user@root ~ # ip -4 route add 0.0.0.0/0 dev wg0 table 51820
user@root ~ # ip -4 rule add not fwmark 51820 table 51820
user@root ~ # ip -4 rule add table main suppress_prefixlength 0
```

The *ip rule* looks likes:

```
0:	from all lookup local 
32764:	from all lookup main suppress_prefixlength 0
32765:	not from all fwmark 0xca6c lookup 51820 
32766:	from all lookup main 
32767:	from all lookup default
```

`suppress_prefixlength 0` bypass the *default* entry (i.e. *default via 192.168.0.1 dev*) in *main* (254) table because *default* prefix is 0. Say we have a rule *from all lookup table 42 suppress_prefixlength 16*. When network stack hits the rule and looks up route table 42 against packet destination IP as it would do to rule *from all lookup table 42*. When the longest match found, it checks the route entry's prefix length. If the prefix is longer than 16 bits, it proceeds as usual. Otherwise (shorter than or equal to 16 bits), network stack go to check the next rule.

Let's say, table 42 contains two entries:

```
10.0.0.0/8 dev eth0
10.1.1.0/24 dev eth1
```

And the destination IP is *10.1.1.1*. Obviously, the 2nd entry matches (24 > 16 > 8) and routes the packet to *eth1*. If *10.2.1.1*, then the 1st entry has the longest prefix that, however, is less than 16. Table 42 does not route the packet. From the analysis, `suppress_prefixlength` puts a strict on prefix threshold. Specially, 0 threshold just excludes the *default* entry of a table.

Now let's go through an example to see how a DNS packet is routed through rules by *wg-quick*:

1. Suppose DNS query on *www.bing.com* is sent to server is *8.8.8.8*.
2. Network stack checks rule 32764, namely the *main* table.

   The *default* entry matches *8.8.8.8* but with prefix 0. So it's ignored.

   Assume there does not exist other explicit entries for *8.8.8.8* in *main* table.
3. Network stack checks the next rule 32765.

   The DNS packet is not marked as 51820 and successfully routed to 51820 table.
4. WireGuard received the DNS query packet.

   It encrypts the packet and forms a new UDP packet with destination IP as *endpoint* (server public IP). This newly generated packet has *fwmark 51820*.
5. Network checks rule 32764 to route the new UDP packet.

   Similary the *default* entry is ignored.
5. Network stack checks rule 32765.

   Due to *fwmark 51820*, table *51820* is ignored either.
5. Network stack checks rule 32766.

   It's the *main* table again. This time there is no `suppress_prefixlength 0` limit. The WireGuard UDP packet is routed to *default* entry of *main* table, going out!

>Of all the methods mentioned above, *wg-quick* is robust to peer roaming (endpoint change) as it does not require explicit endpoint routing to be added.

# Smart routing

In prevous section, we can route all traffic through WireGuard. The all-in-one routing is somewhat stupid. For example, we only want some traffic (i.e. that blocked by ISP) through WireGuard while the remaining through the *main* table.

## IP set

Firstly, we divide IPs into two sets. traffic of one goes to WireGuard while the other not. IPs are from [APNIC Delegated List](http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest).

CN set:

```bash
user@tux ~ # wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > cnip.txt
```

non-CN set:

```bash
user@tux ~ # wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/\|ipv4\|/ && ! /\|CN\|/ && ! /\|\*\|/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > wgip.txt
```

We can set a *cron* job to update IP set in the background.

## Static Route

One obvious method is adding all IPs of WireGuard set to server peer's *allowed-ips* while removing the catch-all *0.0.0.0/0*. *wg-quick* will create the relevant route entries for them. It looks like:

```
1.0.128.0/17 dev wg0 proto kernel scope link src 10.0.0.1
```

Similarly we can also create route entries in *main* table for IPs of CN set:

```
1.0.32.0/19 via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304
```

I will choose the 2nd method as:

```bash
root@tux ~ # xargs -a /etc/wireguard/cnip.txt -I'{}' ip route add '{}' via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304
# -or-
root@tux ~ # wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' | xargs -a /etc/wireguard/cnip.txt -I'{}' ip route add '{}' via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304
```

1. Don't worry about route entries if you manually bring down WireGuad.
2. We have *src* in the route entries, which would fail if *wlan0* or *wg0* do not have a IP the time we add routes (i.e. Wi-Fi router down).

   To simple things, just remove *src* part. If you'd like, remove *metric* and *proto* as well.

Read more on [chnroutes](https://github.com/fivesheep/chnroutes) and [chinaroute路由表更新命令](https://gist.github.com/lixingcong/286144b3a521add58d8dcf045700963f)

## Geoip method

Instead of IP set, we can make use of *geopip* extension of *iptables*. This is much more simpler.

```
root@tux ~ # iptables -t mangle -A OUTPUT -o wlan0 -m geoip --dst-cc CN -j MARK --set-mark 51820
```

Recall the *ip rule* above, we packets with *51820* mark will routed by *main* table as usual.

That's all!

http://worldend.logdown.com/posts/304263-gentoo-install-iptables-geoip
https://gist.github.com/woods/25ef91a95da85bf10974
https://daenney.github.io/2017/01/07/geoip-filtering-iptables.html

# Smart DNS

With Smart routing above, we route traffic based on IP set. However, that depends on correct DNS resolving when visit a domain. Hence, we usually want a custom DNS server (i.e. *8.8.8.8*) other than the one provided by ISP which (DNS poisoning).

This in turn, would degrade performance when visiting pages within the ISP as custom DNS usually returns IP of geographically far. Therefore, we should enable set up smart DNS service which resolves domains within ISP as usual (geographically nearby) but others censorshipped correctly. If you could find such a public DNS server, just make sure traffic to it goes to WireGuard.

The goal is to set a DNS which resolve CN domains into CN IP set while non-CN domains into non-CN IP set.

Alternatively, we could set up local DNS server like [ChinaDNS](https://github.com/shadowsocks/ChinaDNS). An better setup is illustrated in [iproute2, iptables, dnsmasq](/2018/03/26/policy-routing/)

# wg-quick

*wg-quick* is just a wrapper script of commands of *ip* and *wg* above to faciliate the interface and WireGuard setup.

```bash
# wg-quick [ up | down | save ] [ CONFIG_FILE | INTERFACE ]
root@tux ~ # wg-quick save wg0
root@tux ~ # wg-quick up wg0
```

Please make sure */etc/wireguard/INTERFACE.conf* file exist.

**ATTENTION**: The configuration file of *wg-quick* introduces a few extra items (i.e. internal IP, DNS) to format understood by *wg* in order to configure additional attribute of WireGuard interface. It handles the values that it understands, and then it passes the remaining ones directly to *wg* for further processing. That is to say, *wg-quick* and *wg* share the same configuration file but read only part of it.

Especially, *wg-quick* update system route table automatically.

Here is an client example:

```
[Interface]
ListenPort = 48574
Address = 10.0.0.1/24
DNS = 8.8.8.8
SaveConfig = true
PrivateKey = EOpLmUJ08uwH2z2NpBwJ2upWzB5Tn36nhQvlXccAFnk

[Peer]
PublicKey = 7nXZYaqyXuVdW0RwHauMJuW81axAM9hSavY9JIJsZFU=
AllowedIPs = 10.0.0.2/32
Endpoint = 23.45.67.89:39814
```

You can see that Address, DNS and SaveConfig items are added. The DNS defined by *wg-quick* will replace that of system default by *resolvconf* of *net-dns/openresolv*.

Here is an server example:

```
[Interface]
Address = 10.0.0.1/24
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 39814
PrivateKey = [SERVER PRIVATE KEY]

[Peer]
PublicKey = [CLIENT PUBLIC KEY]
AllowedIPs = 10.0.0.1/32  # This denotes the clients IP.
```

# Booting

## Gentoo wg-quick

Here I use *local.d* startup script. Please make sure the scripts are executable.

Up: */etc/local.d/01-wireguard.start*:

```bash
#!/bin/sh

echo "Starting WireGuard ..."
/usr/bin/wg-quick up wg0

echo "Adding cnip routes in background ..."
xargs -a /etc/wireguard/cnip.txt -I'{}' ip route add '{}' via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304 >/dev/null 2>&1 &
```

The route part can be moved to */etc/dhcpcd.exit-hook* as well. But on my system, *dhcpcd* reports:

>Error: Device for nexthop is not up

From what the message, *wlan0* is not up before route entries added.

Down: */etc/local.d/01-wireguard.stop*:

```bash
#!/bin/sh

echo "Stopping WireGuard ..."
/usr/bin/wg-quick down wg0

echo "Removing cnip routes in background ..."
xargs -a /etc/wireguard/cnip.txt -I'{}' ip route del '{}' via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304 >/dev/null 2>&1 &
```

By default, the *local* service will silence all output. Setting `rc_verbose=yes` will cause it to show which scripts were run and their output, if any.

```
# /etc/conf.d/local
rc_verbose=yes
```

Check `rc_verbose` in */etc/rc.conf*.

## systemd wg-quick

```bash
root@tux ~ # systemctl enable wg-quick@wg0
root@tux ~ # systemctl start wg-quick@wg0
```

## cron job

Set a cron job to update IP set daily.

```bash
root@tux ~ # crontab -u root -e
#
9  14  * * *     root    wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/wireguard/cnip.txt
```

Alternatively, we can set a script file (executable) into */etc/cron.daily/*.

```bash
#!/bin/sh

#/etc/cron.daily/cnip

wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/wireguard/cnip.txt
```

# Notes

*wg-quick up* process:

```
[#] ip link add wg-quick type wireguard
[#] wg setconf wg-quick /dev/fd/63
[#] ip address add 192.168.57.1/24 dev wg-quick
[#] ip link set mtu 1420 dev wg-quick
[#] ip link set wg-quick up
[#] resolvconf -a wg-quick -m 0 -x
[#] wg set wg-quick fwmark 51820
[#] ip -4 route add 0.0.0.0/0 dev wg-quick table 51820
[#] ip -4 rule add not fwmark 51820 table 51820
[#] ip -4 rule add table main suppress_prefixlength 0
```

*wg-quick down* process:

```
[#] wg showconf wg-quick
[#] ip -4 rule delete table main suppress_prefixlength 0
RTNETLINK answers: Address family not supported by protocol
Dump terminated
RTNETLINK answers: Address family not supported by protocol
Dump terminated
[#] ip link delete dev wg-quick
[#] resolvconf -d wg-quick
```

# Reference

1. [Wireguard - Part Two (VPN routing)](https://www.ericlight.com/wireguard-part-two-vpn-routing.html)
2. [WireGuard: 简单好用的 VPN](https://blog.lilydjwg.me/2017/10/10/wireguard.210886.html)
3. [Virtual private networks with WireGuard](https://lwn.net/Articles/748582/)
4. [Quick Start](https://www.wireguard.com/quickstart/)
5. [wireguard-vpn-typical-setup](https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/)
6. [NAT-to-NAT VPN with WireGuard](https://staaldraad.github.io/2017/04/17/nat-to-nat-with-wireguard/)
7. [WireGuard VPN Walkthrough](https://nbsoftsolutions.com/blog/wireguard-vpn-walkthrough)
