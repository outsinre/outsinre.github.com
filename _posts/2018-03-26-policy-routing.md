---
layout: post
title: Policy Routing
---

1. toc
{:toc}

This post is the successor of [WireGuard](/2018/03/20/wireguard/).

# Policy Routing

Policy routing could be accomplished based on bare IPs or domains. [WireGuard](/2018/03/20/wireguard/) post subscribes and updates IP sets periodically and adds relevant routes, which lacks flexibility as most IPs of the WireGuard set are not blocked by ISP. All traffic, by default, is routed to WireGuard except that of the CN set. The CN set can be called WireGuard blacklist. Another blacklist routing schema based on *dnsmasq* is [dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list).

Rather, WireGuard whitelist maintains a much smaller IP set of blocked domains - blocked IP set. Only route traffic of blocked set to WireGuard table while leaving the rest to the *main* table. All traffic, by default, is routed as usual.

Usually, each IP set corresponds to a domain set. For instance, the blocked IP set mainly derives from the blocked domain set though ISP may decide to add on individual IPs to the list occasionally.

In a nutshell, blacklist, whitelist and their sub-list can combine to realize policy routing as long decent routing rules and/or tables are configured. However, all the following configuration steps only involve whitelist set. To support other sets, just introduce extra routing tables and rules.

# Terminology

1. A routing rule decides which routing table to look up.
2. A routing table is a collection of routes that refer to entries that direct packets.
3. Iproute2, Iptables, Ipset and Dnsmasq together do smart routing, namely policy routing.

# Principle

1. Dnsmasq serves as a smart DNS service, maintaining domain sets: blocked domain set, non-blocked domain set, and sub-sets.
2. A relevant kernel Ipset is created in respond to each domain set.
3. Dnsmasq updates blocked Ipsets on the fly.
4. Iptables set a MARK on Ipsets.
5. Iproute2 rules and tables route packets based on Iptables MARK.

# Iproute2 Tutorial

1. *iproute2* is successor of the ancient *ifconfig*, providing userspace utilities for TCP / IP networking and traffic control in Linux, including routing, interfaces, tunnels, and traffic control.
2. Predefined identifiers (number and string name pairs) are located under */etc/iproute2/*. An item can be either referenced by its number or name. For example, table *254 main* is where we update routes mostly.
3. A *metric* or *preference* is to [set preference](https://serverfault.com/a/648279) among routes or rules with equal specificity.

Check routing rules and routes:

```bash
root@tux ~ # man [ ip-route | ip-rule ]
root@tux ~ # ip rule list
root@tux ~ # ip route [ list table main ]
```

Create routing rules and tables:

```bash
root@tux ~ # ip -4 rule add not fwmark 0xca6c pref 10 table 0xca6c
root@tux ~ # ip rule add to 223.255.236.0/22 pref 20 table 254
```

1. New routing tables are created on demand (i.e. not exist)
2. *pref* specifies rule preference.

Add routes:

```bash
root@tux ~ # ip route add 1.0.1.0/24 via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.100 metric 30
root@tux ~ # ip route add 8.8.8.8/32 dev wlan0 pref 40
```

1. *via* means the gateway.
2. *dev* followed by the outgoing interface.
3. *pref* and *metric* are the same thing: priority/preference.

# Launch WireGuard

*wg-quick* generates Iproute2 rules and tables to route all traffic through WireGuard except that of *endpoint*. To support different sets, we should carefully configure Iproute2.

The following setup is an example of WireGuard whitelist set. In spite of *wg-quick* method in prior post, manual configuration is preferred in this post.

*01-wg0.start*:

```bash
#!/bin/sh

ip link add wg0 mtu 1420 type wireguard
ip link set dev wg0 up
ip addr add 10.0.0.2/24 dev wg0
wg setconf wg0 /etc/wireguard/wg0.conf
```

The script is updated as this post progresses. Similarto the preceding post, set this as *local.d* Init service.

# Relation Between Ipset and Iptables

"IP set" is a framework inside the Linux kernel, which can be administered by userspace utility *ipset*. Depending on the type, an IP set may store IP addresses, networks, (TCP/UDP) port numbers, MAC addresses, interface names or combinations of them in a way, which ensures lightning speed when matching an entry against a set.

Linux kernel must enable IP set and then *net-firewall/ipset* is required. IP set can be enabled by `IP_SET` and relevant sub-options such as `IP_SET_HASH_IP`, `IP_SET_HASH_NET` and `IP_SET_LIST_SET`. Alternatively, enable *USE=modules* of *net-firewall/ipset*.

An IP set based on blocked domains will be updated on the fly. To make use of the IP set, please enable Netfilter (Iptables) SET match extension (`NETFILTER_XT_SET`) and MARK target (`NETFILTER_XT_MARK`).

# Create Ipset

Installation:

```bash
# USE=modules is not enabled.
root@tux ~ # emerge -avt ipset
root@tux ~ # rc-update add ipset default
```

Now we can create kernel IP set with *ipset* command:

```bash
root@tux ~ # ipset -! create gfwlist hash:net
root@tux ~ # ipset list [ gfwlist ]
```

By default, *ipset* will throw an error errors when creating sets or adding elements that do exist or when deleting elements that don't exist. `-!, -exist` will ignore the error.

An IP set called 'gfwlist' is created but it's an empty set without any IP elements. Let's add a few entries:

```bash
root@tux ~ # ipset -! add gfwlist 8.8.8.8; ipset list
root@tux ~ # ipset -! add gfwlist 8.8.4.4; ipset list
root@tux ~ # ipset -! del gfwlist 8.8.4.4; ipset list
```

We want all IPs associated with blocked domains be added to 'gfwlist' set, which is impossible without the help of *dnsmasq* as IPs vary frequently. That is to say, 'gfwlist' set should be updated frequently as well. Especially, we want to add clean DNS servers (i.e. *8.8.8.8*) to 'gfwlist' at first.

Save current state

```bash
root@tux ~ # rc-service ipset save
# -or-
root@tux ~ # ipset save > /var/lib/ipset/rules-save
# Launch
root@tux ~ # rc-service ipset start; ipset list
```

By default, *ipset* service will load prior set on startup while save current set on stop.

# Iptables Set MARK On IPset

Having Kernel IP set created and preliminarily updated, we assign MARK target to 'gfwlist' set with Iptables.

```bash
# Create a new chain
root@tux ~ # iptables -t mangle -N GFWLIST

# OUTPUT to GFWLIST
root@tux ~ # iptables -t mangle -C OUTPUT -j GFWLIST || iptables -t mangle -A OUTPUT -j GFWLIST
# Set mark 51820 on packets of 'gfwlist' IP set.
root@TUX ~ # iptables -t mangle -A GFWLIST -m set --match-set gfwlist dst -j MARK --set-mark 51820

# Only required when peer A serves as a gateway as well.
root@tux ~ # iptables -t mangle -C [ FORWARD | PREROUTING ]  -j GFWLIST || iptables -t mangle -A [ FORWARD | PREROUTING ] -j GFWLIST (opt)
root@tux ~ # sysctl -w net.ipv4.ip_forward=1 (opt)
```

1. A new chain GFWLIST is created for 'gfwlist' IP set for better isolation.
2. `-C` checks whether a rule exists or not.
3. When a packet hits FORWARD chain, it has been routed already. Instead, PREROUTING is hit before routing.

# Iproute2 Rule and Table

Now that 'gfwlist' IP set is marked by Iptables, we will create a routing rule and table for the mark. Before that we can define a string identifier for the table.

```bash
# Number and name identifier
root@tux ~ # echo "51820	gfwlist" >> /etc/iproute2/rt_tables

# Create routing rule and table: traffic of 'gfwlist' set is routed through 'gfwlist' table
root@tux ~ # ip rule add fwmark 51820 table gfwlist

# Set routing table: going out through WireGuard.
root@tux ~ # ip route add default dev wg0 table gfwlist
```

Identifier of Iproute2 table (*51820 gfwlist*) needn't to be the same as that of Iptables mark (51820).

Recall that, we've manually added *8.8.8.8* to 'gfwlist' set. Instead, add a new route in the *main* table though it is somewhat less mart.

```bash
root@tux ~ # ip route add 8.8.8.8 dev wg0 table main
```

*01-wg0.start*:

```bash
#!/bin/sh

# Bring up WireGuard link
ip link add wg0 type wireguad
ip link set mtu 1420 dev wg0
ip addr add 10.0.0.2/24 dev wg0
wg setconf wg0 /etc/wireguard/wg0_wg.conf
ip link set up dev wg0

# To route server's internal IP
ip route add 192.168.58.0/24 dev wg0
# -or-
#ip rule add 192.168.58.0/24 table gfwlist

# To route 'gfwlist' IP set
ip rule add fwmark 51820 table gfwlist
ip route add default dev wg0 table gfwlist
```

1. In this case, the internal IP of client and server fall into different network block, namely *10.0.0.0/24* and *192.168.58.0/24*. As mentioned in the prior post, explicit route should be added on both sides.
2. This is almost the final version. As *192.168.58.0/24* is an LAN block, we'd better leave it in the main table.

# Set Up Dnsmasq

The key success of policy routing depends on the accuracy of 'gfwlist' IP set. So far, it is almost empty besides a few clean DNS elements. This is where Dnsmasq comes to help!

Dnsmasq provides Domain Name System (DNS) forwarder, Dynamic Host Configuration Protocol (DHCP) server, router advertisement and network boot features for small computer networks, created as free software. Here, we use Dnsmasq to provide smart DNS service locally and update 'gfwlist' IP set on demand.

Installation:

```bash
root@tux ~ # echo 'net-dns/dnsmasq -dhcp dnssec' > /etc/portage/package.use/dnsmasq
root@tux ~ # emerge -avt net-dns/dnsmasq
root@tux ~ # rc-update add dnsmasq default
```

1. Dnsmasq is presented with predefined blocked domains. Dnsmasq resolves them into IPs upon receiving DNS query.
2. Dnsmasq now has builtin Ipset support and the resolved IPs can be added to 'gfwlist' IP set automatically.

## Configuration Files

All options within the default */etc/dnsmasq.conf* are commented out. We can leave that as a reference but load configuration from separate files.

Firstly, we add a command line `--conf-dir` to */etc/conf.d/dnsmasq*:

```
# Loads all files with the suffix .conf; ignore ther others

DNSMASQ_OPTS="--user=dnsmasq --group=dnsmasq --conf-dir=/etc/dnsmasq.d/,*.conf"
```

A better idea is appending that option to */etc/dnsmasq.conf* so that *resolvconf* will pick it up:

```bash
root@tux ~ # echo >> /etc/dnsmasq.conf
root@tux ~ # echo 'conf-dir=/etc/dnsmasq.d/,*.conf' >> /etc/dnsmasq.conf

root@tux ~ # mkdir -p /etc/dnsmasq.d
```

## Configure Dnsmasq

Dnsmasq will be the sole DNS server in the system in the following set up. Please prevent *dhcpcd* from overriding */etc/resolv.conf*:

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

# Upstream nameservers (i.e. from ISP)
resolv-file=/etc/dnsmasq.d/resolv.dnsmasq

# hosts addon
addn-hosts=/etc/dnsmasq.d/hosts.dnsmasq
```

*dnssec-check-unsigned* would drop DNS replies if upstream DNS servers does not support DNSSEC. Up to now, I cannot find any ISP DNS servers with DNSSEC support. Hence, *dnssec-check-unsigned* cannot resolve unblocked domains from upstream nameservers.

If we want Dnsmasq to support newly created interface on demand, we can use *bind-dynamic*:

```
# lo only
#bind-interfaces
#listen-address=127.0.0.1

# lo and eth2
bind-dynamic
interface=eth2
```

This would implicitly include *lo*.

## Update Blocked Domains and Ipset

As discussed earlier, we should firstly fuel Dnsmasq with blocked domains. Here is a predefined and updated blocked domain list: [gfwlist2dnsmasq](https://github.com/cokebar/gfwlist2dnsmasq).

```bash
root@tux ~ # git clone --depth=1 https://github.com/cokebar/gfwlist2dnsmasq
root@tux ~ # ./gfwlist2dnsmasq.sh -h
root@tux ~ # ./gfwlist2dnsmasq.sh -d 8.8.8.8 -p 53 -s gfwlist -o /etc/dnsmasq.d/gfwlist.conf
```

The accompanying script *gfwlist2dnsmasq.sh* automatically fetches newest blocked domains and generates corresponding Dnsmasq configuration file without user intervention. Then, set *cron* job to update 'gfwlist' set periodically:

```bash
#!/bin/sh

/opt/gfwlist2dnsmasq/gfwlist2dnsmasq.sh -d 8.8.8.8 -p 53 -s gfwlist -o /etc/dnsmasq.d/gfwlist.conf
```

A new *dnsmasq* configuration file which only contains *server* and *ipset* pairs like:

```
server=/030buy.com/8.8.8.8#53
ipset=/030buy.com/gfwlist

server=/0rz.tw/8.8.8.8#53
ipset=/0rz.tw/gfwlist
```

*dnsmasq* resolves *030buy.com* and *0rz.tw* through *8.8.8.8:53*, which would be routed through WireGuard.

So far, Dnsmasq is done! However, we can't even *dig* blocked domains.

# Iptables NAT by [MASQUERADE](https://serverfault.com/q/248841)

To be clear, we should enable MASQUERADE or SNAT as post [linux as gateway router](/2017/12/29/linux-as-gateway-router/) writes. Please be noted policy routing on the client side do not require IP forwarding or Iptables FORWARD.

Let us have a look at Iptables log on `-t nat OUTPUT` chain:

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

Please refer to the Netfilter flow chart:

![Packet flow in Netfilter and General Networking](http://inai.de/images/nf-packet-flow.svg)

1. The first line. The packet DST is *8.8.8.8* with MARK as 0xca6c, which means this is the original DNS query packet. The *routing decision* successfully matched the *default* route in *main* table. Afterwards, the packet was set MARK by `-t mangle OUTPUT` chain. Then *reroute check* routed the packet against *gfwlist* rule and table according the the MARK. Finally, a new packet is generated.
2. The second line. The new packet DST is *23.45.67.89* (server's public IP), which suggests the the first packet was routed through WireGuard by *reroute check*. The new packet was then sent to server.

However, the OUT and SRC of the first packet must be changed to *wg0* and *10.0.0.1* (internal IP of client A) after *reroute check* and before goint out, which is done with support from Iptables NAT. That is to say, the first packet should be transferred from:

```
# Before 'reroute check'.
OUT=wlan0 SRC=192.168.0.100 DST=8.8.8.8
```

to

```
# After 'reroute check'.
OUT=wg0 SRC=10.0.0.1 DST=8.8.8.8 MARK=0xca6c
```

Without Iptables NAT support, reply packets from the server would be abandoned. Suppose the VPN server receives the second line and replies with DST *192.168.0.100* and DPT 12345. The reply packet is then passed to *wg0* in accord with DPT where WireGuard listens for traffic. Upon receiving the packet, *wg0* abandons it immediately as DST does not match *10.0.0.1* - malformed packet.

We can also examine Iptable log on other Iptables tables and chains in accordance with the figure above. For example, before `-t mangle OUTPUT` the first packet is not marked.

Iptables NAT usually is executed in `-t nat POSTROUTING` chain with MASQUERADE or SNAT target.

```bash
root@tux ~ # iptables -t nat -A POSTROUTING -o wg0 -m mark --mark 51820 -j SNAT --to-source 10.0.0.1
# -or-
root@tux ~ # iptables -t nat -A POSTROUTING -o wg0 -m mark --mark 51820 -j MASQUERADE
```

Up to now, policy routing with WireGuard whitelist ('gfwlist' Ipset) is done! We can add extra Ipsets and relevant Iproute2 routing rules and tables. Of course, new Ipsets should also be marked by Iptables.

# Local Init Script

*01-wg0.start*:

```bash
#!/bin/sh

# Bring up WireGuard link
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

# Remove WireGuard
ip link del dev wg0 type wireguard
```

The VPN Proxy section in prior post, all traffic is routed through *wg0*. So it presents multiple methods (i.e. *suppress_prefixlength 0* from *wg-quick*) such that traffic to the server's public IP is routed through the *main* table instead.

The scripts above do not require any of those methods as 'gfwlist' does not include that IP. More specifically, as long as Ipset is used, we can put the IP into a set (i.e. CN set) routed by *main* table. At the same time, make sure it is not covered by Ipsets routed through *wg0*.

# Refs

1. [路由器自动分流科学上网原理和实现方式研究](http://harveyhu2012.webcrow.jp/wordpress/?p=184)
2. [Openwrt上使用dnsmasq和ipset实现域名分流](https://www.cnblogs.com/weifeng1463/p/6796140.html)
3. [OpenWrt VPN 按域名路由](https://blog.sorz.org/p/openwrt-outwall/)
4. [OpenWRT实现OpenVPN按ipset自动分流](https://zohead.com/archives/openwrt-openvpn-ipset/)
5. [使用 dnsmasq 和 ipset 的策略路由](https://bigeagle.me/2016/02/ipset-policy-routing/)
