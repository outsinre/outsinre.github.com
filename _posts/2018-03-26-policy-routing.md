---
layout: post
title: iproute2, iptables, ipset, dnsmasq
---

1. toc
{:toc}

# Policy Routing

1. We can set a MARK on packet and/or CONNMARK on connection with iptables.
2. We can also set up iproute2 *rule* based on MARK.

Policy routing could be accomplished based on IPs or domains. In the [WireGuad](/2018/03/20/wireguard/), we collect IP set periodically and add relevant routes which is not smart enough as we most IPs of non-CN set are not blocked by ISP. All traffic, by default, is routed to WireGuard except those within CN IP set - WireGuad blacklist. Another WireGuad blacklist routing scheme over *dnsmasq* is [dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list). I will leave it on your own.

An improved method is generating a much smaller IP set of blocked domains. We then route packets to those IPs through proxy or VPN while the remaining through default gateway. Then *dnsmasq* and *ipset* come to help. All traffic, by default, is routed as normal while those blocked by ISP to WireGuad - whitelist.

# Terminology

1. A route refers to entries that direct packets flow. Route table is a collection of routes.
2. A rule decides which route table to look up.
3. Iproute2, iptables, dnsmasq, and ipset together do smart routing, namely policy routing.

# iproute2

*iproute2* is successor of the old *ifconfig*, providing userspace utilities for controlling TCP / IP networking and traffic control in Linux, including routing, interfaces, tunnels, and traffic control.

Predefined identifiers (number and name pairs) are located under */etc/iproute2/*. For example, table *254 main* is where we update routes mostly.

The routes *metric* (or the *preference* alias) is to [set preference](https://serverfault.com/a/648279) among routes with equal specificity. It's similar to rule's *preference*.

Check rules and routes:

```bash
root@tux ~ # man [ ip-route | ip-rule ]
root@tux ~ # ip rule list
root@tux ~ # ip route [ list table main ]
```

Add rules:

```bash
root@tux ~ # ip -4 rule add not fwmark 0xca6c pref 10 table 0xca6c
root@tux ~ # ip rule add to 223.255.236.0/22 pref 20 table 254
```

Add routes:

```bash
root@tux ~ # ip route add 1.0.1.0/24 via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.100 metric 30
root@tux ~ # ip route add 8.8.8.8/32 dev wlan0 pref 40
```

# WireGuad

To set up WireGuad whitelist, we must abandon *wg-quick* as it generates iproute2 rules to route traffic through WireGuad by default. Instead, we use script to launch WireGuad.

```bash
#!/bin/sh

ip link add wg0 mtu 1420 type wireguard
ip link set dev wg0 up
ip addr add 10.0.0.2/24 dev wg0
wg setconf wg0 /etc/wireguard/wg0.conf
```

I will update the script as this post progresses. As the previous post, add this to *local.d* startup service as *01-wg0.start*.

# ipset/iptables

"IP set" is a framework inside the Linux kernel, which can be administered by userspace utility *ipset*. Depending on the type, an IP set may store IP addresses, networks, (TCP/UDP) port numbers, MAC addresses, interface names or combinations of them in a way, which ensures lightning speed when matching an entry against a set. We set up an IP set based on blocked domains on the fly.

To make use of IP set, Netfilter (iptables) SET match extension (`NETFILTER_XT_SET`) and MARK target (`NETFILTER_XT_MARK`) are required. Meanwhile, enable IP set support as well, which can be done exclusively by kernel options `IP_SET` and submodules (`IP_SET_HASH_IP`, `IP_SET_HASH_NET` and `IP_SET_LIST_SET`) or enable `USE=modules`. Then install *net-firewall/ipset* package.

## ipset

Installation first:

```bash
root@tux ~ # emerge -avt ipset
root@tux ~ # rc-update add ipset default
```

Now we can create kernel IP set with *ipset* command:

```bash
root@tux ~ # ipset -! create gfwlist hash:net
root@tux ~ # ipset list [ gfwlist]
```

An IP set called *gfwlist* is created but it's an empty set without any IP elements. Let's update the set:

```bash
root@tux ~ # ipset -! add gfwlist 8.8.8.8; ipset list
root@tux ~ # ipset -! add gfwlist 8.8.4.4; ipset list
root@tux ~ # ipset -! del gfwlist 8.8.4.4; ipset list
```

By default, *ipset* will throw an error errors when creating or adding sets or elements that do exist or when deleting elements that don't exist. `-!, -exist` will ignore the error.

We Want all IPs associated with blocked domains be added to *gfwlist* set, which is impossible without the help of *dnsmasq* as IPs vary frequently. That is to say, *gfwlist* set should be updated frequently as well. Especially, we want to add public DNS server IP (i.e. *8.8.8.8*) to *gfwlist* at first.

Save current state

```bash
root@tux ~ # rc-service ipset save
# -or-
root@tux ~ # ipset save > /var/lib/ipset/rules-save
# launch
root@tux ~ # rc-service ipset start
```

By default, *ipset* service will load privous set on startup while save current set on stop.

## iptables

Now we assign MARK target to *gfwlist* set with iptables:

```bash
root@tux ~ # iptables -t mangle -N GFWLIST
root@tux ~ # iptables -t mangle -C FORWARD -j GFWLIST || iptables -t mangle -A FORWARD -j GFWLIST (opt)
root@tux ~ # sysctl -w net.ipv4.ip_forward=1 (opt)
root@tux ~ # iptables -t mangle -C OUTPUT -j GFWLIST || iptables -t mangle -A OUTPUT -j GFWLIST
root@TUX ~ # iptables -t mangle -A GFWLIST -m set --match-set gfwlist dst -j MARK --set-mark 51820
```

A new chain GFWLIST is created for *gfwlist* set for better isolation. `-C` checks whether a rule exists or not (ignore the command line errors). Replace FORWARD with PREROUTING (covers the former) if it's OpenWrt.

Locally generated traffic does not require IP forward feature. We add FORWARD here just in case of traffic coming from outside.

### [**IMPORTANT**](https://serverfault.com/q/248841)

To be consise, we should enable IP forwarding and/or MASQUERADE/SNAT as post [linux as gateway router](/2017/12/29/linux-as-gateway-router/) writes. Though that is a different story, they both routes traffic among different networks and/or different interfaces thereof.

I leave this part the end of the post: IP forwarding and MASQUERADE.

# ip rule

Now that, *gfwlist* set is marked in *iptables*, we will create a routing table for the mark. Before that we can define string name for the table.

```bash
root@tux ~ # echo "51820	gfwlist" >> /etc/iproute2/rt_tables
root@tux ~ # ip rule add fwmark 51820 table gfwlist
root@tux ~ # ip route add default dev wg0 table gfwlist
```

iptables mark value (51820) needn't to be the same as that of iproute2 table (*51820 gfwlist*).

Recall that, we manually added *8.8.8.8* to *gfwlist* set, otherwise, add a new route it in the *main* table

```bash
root@tux ~ # ip route add 8.8.8.8 dev wg0 table main
```

Up to now, *01-wg0.start* looks like:

```bash
#!/bin/sh

# Bring up WireGuad link
ip link add wg0 type wireguad
ip link set mtu 1420 dev wg0
ip addr add 10.0.0.2/24 dev wg0
wg setconf wg0 /etc/wireguard/wg0_wg.conf
ip link set up dev wg0

# ip route
ip route add 192.168.58.0/24 dev wg0
# -or-
#ip rule add 192.168.58.0/24 table gfwlist

# ip rule
ip rule add fwmark 51820 table gfwlist
ip route add default dev wg0 table gfwlist
```

This almost the final version. As *192.168.58.0/24* is an LAN block, we'd better leave it in the main table.

# dnsmasq

Dnsmasq provides Domain Name System (DNS) forwarder, Dynamic Host Configuration Protocol (DHCP) server, router advertisement and network boot features for small computer networks, created as free software. We focus on the DNS part to provide smart DNS query based on IP set.

```bash
root@tux ~ # echo 'net-dns/dnsmasq -dhcp dnssec' > /etc/portage/package.use/dnsmasq
root@tux ~ # emerge -avt net-dns/dnsmasq
root@tux ~ # rc-update add dnsmasq default
```

Presented with blocked domains, *dnsmasq* resolves them into IPs which in turn are added into the *gfwlist* set. *dnsmasq* now has *ipset* support and resolved IPs can be added to *gfwlist* automatically as long as we configure it properly.

## Configuration

### Files

All options within the default */etc/dnsmasq.conf* are commented out. We can leave that as a reference but load configuration from separate files.

Firstly, we add a command line `--conf-dir` to */etc/conf.d/dnsmasq*:

```
# Loads all files with the suffix .conf; ignore ther others

DNSMASQ_OPTS="--user=dnsmasq --group=dnsmasq --conf-dir=/etc/dnsmasq.d/,*.conf"
```

A better idea is appending that option to */etc/dnsmasq.conf* so that *resolvconf* will pick up:

```bash
root@tux ~ # echo >> /etc/dnsmasq.conf
root@tux ~ # mkdir -p /etc/dnsmasq.d
root@tux ~ # echo 'conf-dir=/etc/dnsmasq.d/,*.conf' >> /etc/dnsmasq.conf
```

### DNS

*dnsmasq* will be the sole DNS server in the system in the following set ups. Please prevent *dhcpcd* from overriding */etc/resolv.conf*:

```
# /etc/dhcpcd.conf

nohook resolv.conf
```

Then we create */etc/dnsmasq.d/dns.conf*:

```

# lo only
bind-interfaces
listen-address=127.0.0.1
# lo and eth2
#bind-dynamic
#interface=eth2

# DNS port
port=53

# dbus
enable-dbus

# dnssec
dnssec
conf-file=/usr/share/dnsmasq/trust-anchors.conf
#dnssec-check-unsigned

# Upstream nameservers
resolv-file=/etc/dnsmasq.d/resolv.dnsmasq

# hosts addon
addn-hosts=/etc/dnsmasq.d/hosts.dnsmasq
```

*dnssec-check-unsigned* would drop DNS replies if upstream DNS servers does not support DNSSEC. Up to now, I cannot find any China DNS servers with DNSSEC support. Hence, *dnssec-check-unsigned* cannot resolve domains out of GFWlist.

If we want *dnsmasq* support newly created interface on the fly, we can use *bind-dynamic*:

```
# lo only
#bind-interfaces
#listen-address=127.0.0.1

# lo and eth2
bind-dynamic
interface=eth2
```

### Update IP set

We use *dnsmasq* to fill up *gfwlist* set with [gfwlist2dnsmasq](https://github.com/cokebar/gfwlist2dnsmasq).

```bash
root@tux ~ # git clone --depth=1 https://github.com/cokebar/gfwlist2dnsmasq
root@tux ~ # ./gfwlist2dnsmasq.sh -h
root@tux ~ # ./gfwlist2dnsmasq.sh -d 8.8.8.8 -p 53 -s gfwlist -o /etc/dnsmasq.d/gfwlist.conf
```

We want to update *gfwlist* set periodically with *cron* job */etc/cron.daily/gfwlist2dnsmasq*:

```bash
#!/bin/sh

/opt/gfwlist2dnsmasq/gfwlist2dnsmasq.sh -d 8.8.8.8 -p 53 -s gfwlist -o /etc/dnsmasq.d/gfwlist.conf
```

A new *dnsmasq* configuration which only contains *server* and *ipset* pairs like:

```
server=/030buy.com/8.8.8.8#53
ipset=/030buy.com/gfwlist

server=/0rz.tw/8.8.8.8#53
ipset=/0rz.tw/gfwlist
```

*dnsmasq* resolves *030buy.com* and *0rz.tw* through *8.8.8.8:53*.

# init

*01-wg0.start*:

```bash

#!/bin/sh

# Bring up WireGuad link
ip link add wg0 mtu 1420 type wireguard
ip link set dev wg0 up
ip addr add 10.0.0.2/24 dev wg0
wg setconf wg0 /etc/wireguard/wg0.conf

# ip route
ip route add 192.168.58.0/24 dev wg0
# -or-
#ip rule add 192.168.58.0/24 table gfwlist

# ip rule
ip rule add fwmark 51820 table gfwlist
ip route add default dev wg0 table gfwlist
```

*01-wg0.stop*:

```bash
#!/bin/sh

# del gfwlist table
ip rule del table gfwlist

# ip route
ip route del 192.168.58.0/24 dev wg0
# -or-
#ip rule del 192.168.58.0/24 table gfwlist

# Remove WireGuad
ip link del dev wg0 type wireguard
```

# IP forwarding and MASQUERADE

>This follows the iptables section above.

After all the setup, we can't even *dig* blocked domains. To have a look at packet log:

```bash
root@tux ~ # iptables -t nat -A OUTPUT -m mark --mark 51820 -j LOG --log-level debug
root@tux ~ # dmesg -w
root@tux ~ # dig www.google.com
```

Here is an log sample:

```
[20083.739224] IN= OUT=wlan0 SRC=192.168.0.100 DST=8.8.8.8 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=10293 DF PROTO=UDP SPT=38374 DPT=53 LEN=40 MARK=0xca6c 
[20083.739408] IN= OUT=wlan0 SRC=192.168.0.100 DST=23.45.67.89 LEN=124 TOS=0x00 PREC=0x00 TTL=64 ID=14878 PROTO=UDP SPT=12345 DPT=54321 LEN=104 
```

The issue results from OUT and SRC parts of line 1.

![Packet flow in Netfilter and General Networking](http://inai.de/images/nf-packet-flow.svg)

Fromt his figure, on the right side, a packet (DNS) from upper layer is routed (**routing decision**) by *main* table before entering *iptables*. At this stage, source IP is determined to be that of *wlan0*.

Then *mangle output* marks the packet as 51820 and pass it to *ip rule* for 2nd routing (**reroute check**). However, *ip rule* won't touch OUT and SRC, which explains the first log line. Line 2 means packet is correctly passed to VPN server by WireGuad (kernel module).

Suppose the VPN server gets the DNS query and send feedback packet with DST *192.168.0.100* and DPT 12345 which is then passed to *wg0* based on DPT where WireGuad listens for traffic. Upon receiving the packet, *wg0* abandons it immediately as DST does not match *10.0.0.1*.

Therefore, we should add MASQUERADE/SNAT NAT support in *nat* table after **reroute check**:

```bash
root@tux ~ # iptables -t nat -A POSTROUTING -o wg0 -m mark --mark 51820 -j SNAT --to-source 10.0.0.1
# -or-
root@tux ~ # iptables -t nat -A POSTROUTING -o wg0 -m mark --mark 51820 -j MASQUERADE
```

If it's OpenWrt (LAN interface and WAN interface), we should also enable IP forwarding:

```bash
root@tux ~ # sysctl -w net.ipv4.ip_forward=1
root@tux ~ # iptables -A FORWARD -i wg0 -o eth0 -m mark --mark 51820 -j ACCEPT
```

In this post, we don't require IP forwarding as locally generated packets does not traverse FORWARD chain.

# Refs

1. [路由器自动分流科学上网原理和实现方式研究](http://harveyhu2012.webcrow.jp/wordpress/?p=184)
2. [Openwrt上使用dnsmasq和ipset实现域名分流](https://www.cnblogs.com/weifeng1463/p/6796140.html)
3. [OpenWrt VPN 按域名路由](https://blog.sorz.org/p/openwrt-outwall/)
4. [OpenWRT实现OpenVPN按ipset自动分流](https://zohead.com/archives/openwrt-openvpn-ipset/)
5. [使用 dnsmasq 和 ipset 的策略路由](https://bigeagle.me/2016/02/ipset-policy-routing/)
