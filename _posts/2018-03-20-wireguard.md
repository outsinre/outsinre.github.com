---
layout: post
title: WireGuard
---

1. toc
{:toc}

# Preamble

[WireGuard](https://www.wireguard.com/) claim to utilizes state-of-the-art cryptography (Noise protocol framework, Curve25519, ChaCha20, Poly1305, BLAKE2, SipHash24, HKDF etc.) while provide a simple construction.

It requires a virtual network interface and an IP assigned. The interface is associated with a public/private key pair of which the public part serves as the identity within WireGuard network. WireGuard encapsulates IP packets (application data payload) over UDP and forwards packets based on Cryptokey Routing- a UDP VPN.

Originally, WireGuard is integrated into Linux kernel (module or built-in tree). Nowadays, it's under heady development to be cross-platform and widely deployable. The 6th reference (NAT-to-NAT VPN with WireGuard) is worth reading.

# Installation

To install as an external module:

```bash
root@tux ~ # emerge -avt wireguard openresolv
root@tux ~ # modinfo wireguard
```

Optionally, we can build WireGuard as an [internal kernel module or as built-in](https://www.wireguard.com/install/) with `module-src` USE.

*openresolv* is needed if *wireguard* (by *wg-quick*) defines custom DNS.

# Schema

Before everything else, we should grasp the conceptual overview.

Peer | Client A | Server B
---|---|---
External IP | 192.168.0.123 | 23.45.67.89
UDP port | 48574 | 39814
Internal IP | 10.0.0.1/24 | 10.0.0.2/24

External IP (i.e. *eth0*) is given by ISP or Wi-Fi router, without which you cannot reach the Internet. On the other hand, internal IP manually assigned to WireGuard interface is privately valid only within WireGuard network. UDP port associated with external IP is where WireGuard service listens for traffic.

WireGuard does not assume server side or client side. The peer initiates connections is regarded as the client. For terminology consistency, peer B with public IP is the server while peer A censorshipped by ISP is the client. In this post, peer A sites behind a Wi-Fi router (NAT).

Clearly, Peer A, on the initiative, opens connections to peer B establishing a point-to-point tunnel. With a few arguments adjustment, peer A reaches the whole remote block *10.0.0.0/24* such as devices without WireGuard but IP falling into that range. A step further, peer A can route all its traffic through B to the Internet.

Firstly, we start off by establishing a point-to-point link. Without explicit notice, the configuration steps should be executed on both sides.

# Create WireGuard Interface

Create Wireguard interface with Iproute2, which automatically loads *wireguard* module:

```bash
root@tux ~ # modprobe -v wireguard (opt)
root@tux ~ # ip link add dev wg0 type wireguard
root@tux ~ # ip link
```

## Assign WireGuard IP

Unless you have a really good reason, please assign internal IPs within the same subnet for client and server sides. Otherwise, you want extra routes on both sides.

```bash
root@tux ~ # ip addr add 10.0.0.1/24 dev wg0
root@tux ~ # ip addr
```

# WireGuard Key pair

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

# Configure WireGuard Interface

Arguments can be loaded from file.

```bash
root@tux ~ # wg setconf wg0 /etc/wireguard/wg0.conf
# -or-
root@tux ~ # wg set wg0 listen-port 48574 private-key /etc/wireguard/private-key
root@tux ~ # wg set wg0 peer <public-key of peer> [persistent-keepalive 25] [endpoint 23.45.67.89:39814] allowed-ips 10.0.0.2/32,10.0.100.2/24
root@tux ~ # wg [showconf wg0]
```

1. Public key should be *base64* format like *V7g3kxzLATJ6edBybau1IrE3FOgLHajxxFfMZ+QOUyE=*.
2. To traverse NAT or firewall, *persistent-keepalive* is a must for peers behind NAT or firewall.

   For instance, client A sites behind NAT (i.e. Wi-Fi router), *persistent-keepalive* sends packets to other peers periodically to keep NAT mapping open.
3. *endpoint* is the remote peer's external yet public IP and UDP port.

   Only client sides that makes the initial connection require *endpoint*. It's recommended to leave this argument out when setting server sides because they update *endpoint* values by examining from where correctly authenticated packets originates. Meanwhile, clients behind NAT or dial-up do not even have fixed public IPs.
4. *allowed-ips* is a list of comma-separated IP ranges to which WireGuard traffic can be sent and from which WireGuard traffic can be recevied.

   For a simple point-to-point connection, it should be a peer's internal IP. To reach the whole LAN network, we set it to *10.0.0.0/24*.

   The catch-all *0.0.0.0/0* and *::/0* match all IPv4/6 addresses, routing all local traffic through WireGuard. This is useful if we want to set up transparent proxy on client sides.

   Whatever network ranges we choose, the peer's internal IP must be covered.
5. Multiple peers can be added.

# Persistent Configuration

Setup above is versatile across reboots. To be permanent, arguments can be loaded from file on boot. By default, configuration file locates under */etc/wireguard/*.

To save and load configuration:

```bash
root@tux ~ # wg showconf wg0 > /etc/wireguard/wg0.conf
root@tux ~ # wg setconf wg0 /etc/wireguard/wg0.conf
```

Here is an example of point-to-point VPN link:

```
[Interface]
ListenPort = 48574
PrivateKey = EOpLmUJ08uwH2z2NpBwJ2upWzB5Tn36nhQvlXccAFnk

[Peer]
PublicKey = 7nXZYaqyXuVdW0RwHauMJuW81axAM9hSavY9JIJsZFU=
AllowedIPs = 10.0.0.2/32
Endpoint = 23.45.67.89:39814
```

# WireGuard Link Up

```bash
root@tux ~ # ip link set up dev wg0
root@tux ~ # ip link; ip addr
root@tux ~ # ss -npelu
```

A WireGuard UDP socket is created.

# Check WireGuard Route

When internal IP is assigned, a system route is added automatically for that IP range like:

> 10.0.0.0/24 dev wg0 proto kernel scope link src 10.0.0.1

If a remote peer's internal IP (be covered by *allowed-ips*) belongs to a different subnet (i.e. B's is *192.168.58.1/32*), they cannot connect to each other unless a special route is created on both sides.

```bash
# client A
root@tux ~ # ip route add 192.168.58.0/24 dev wg0 proto kernel scope link
# server B
root@tux ~ # ip route add 10.0.0.0/24 dev wg0 proto kernel scope link
```

For strict control, the range could be narrowed down to the internal IP address like:

```bash
# client A
root@tux ~ # ip route add 192.168.58.1/32 dev wg0 proto kernel scope link
```

# Allow endpoint in Iptables

Please make sure the *endpoint* is accepted by firewall.

```bash
root@tux ~ # iptables -A OUTPUT -d 23.45.67.89/32 -p udp -m udp --dport 39814 -m conntrack --ctstate NEW -j ACCEPT
root@tux ~ # iptables -A INPUT -s 23.45.67.89/32 -p udp -m udp --sport 39814 -m conntrack --ctstate NEW -j ACCEPT
```

## Final ping

By far, a point-to-point link is established. Peer A and B can communicate with each other.

# Set Server B as a [Gateway](/2017/12/29/linux-as-gateway-router/)

For client A's visit to remote LAN or even the Internet, server B should be set as a gateway involving the following steps:

1. IP forwarding and *optional* Proxy ARP.
2. Iptables FORWARD.
3. Iptables NAT.

## IP Forwarding

```bash
root@tux / # sysctl -w net.ipv4.ip_forward=1
```

Usually, IP forwarding is not needed for communication within LAN. However, if peer A want to reach *10.0.0.123* without WireGuard setup, it must depend on peer B's forwarding functionality. This is because A does not have direct Ethernet connection with *10.0.0.123*.

Needless to say, to reach the Internet through server B, this is a must.

## [Proxy ARP](https://www.ericlight.com/wireguard-part-two-vpn-routing.html) on Server Side

Optionally, enable Proxy ARP on *wg0*. However, it seems this step could be omitted.

```bash
# Permanent across reboots
root@tux ~ # echo 'net.ipv4.conf.wg0.proxy_arp=1' > /etc/sysctl.d/25-proxy_arp.conf
```

For the current session:

```bash
root@tux ~ # sysctl -p /etc/sysctl.d/25-proxy_arp.conf
# -or-
root@tux ~ # sysctl -w net.ipv4.conf.all.proxy_arp=1
```

## Allow IP Forwarding through Iptables 

```bash
root@tux ~ # iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT
```

1. We assume *eth0* is the remote interface that has Internet access.
2. You may also choose to ACCEPT `-i eth0 -o wg0`. For example, server B initiates connection to client A.

## Set up Iptables NAT

```bash
root@tux ~ # iptables -t nat -A POSTROUTING -i wg0 -o eth0 -s 10.0.0.0/24 -j SNAT --to-source <IP-of-eth0>
```

1. Similarly, you may also enable `-i eth0 -o wg0`.

# [LAN Access](https://www.ericlight.com/wireguard-part-two-vpn-routing.html)

In order to reach the whole remote block "10.0.0.0/24* including devices without WireGuard, client A should set *allowed-ips* to cover the LAN range.

# [VPN Proxy](https://www.wireguard.com/netns/)

All client A's traffic goes through the WireGuard tunnel: to set up VPN routing as a proxy.

Please be noted that all configurations in this section are done on client A.

## *allowed-ips* Catch All

Similar to LAN Access section above, client A should firstly set its *allowed-ips* to cover the catch-all block: *0.0.0.0/0* and/or *::/0*.

## Update Client A Routes

To route all local traffic through the tunnel, local routes should be updated. Specifically, all traffic except that of *endpoint* (server B's public IP) goes to *wg0*. The key is to route traffic to *wg0* before the *default* route entry, such that only traffic to the server is routed through the *main* table.

Of all the methods mentioned below, *wg-quick* is robust to peer roaming (endpoint change) as it does not require explicit endpoint routing to be added.

### Replace the default Route

```bash
user@root ~ # ip route del default
user@root ~ # ip route add default dev wg0
user@root ~ # ip route add 23.45.67.89/32 via 192.168.0.1 dev wlan0
```

### Override the default Route

Override the default with two more specific rules that add up to the default but match before the default. We split the default into *0.0.0.0/1* and *128.0.0.0/1*.

```bash
user@root ~ # ip route del default (optional)
user@root ~ # ip route add 0.0.0.0/1 dev wg0
user@root ~ # ip route add 128.0.0.0/1 dev wg0
user@root ~ # ip route add 23.45.67.89/32 via 192.168.0.1 dev wlan0
```

### Rule-based routing

The two methods above operates on route entries of the *main* routing table. Here, new routing rules and tables are created.

Notice: a routing rule is set for a specific routing table, namely to determine which routing table is consulted when routing. Without explicit *priority* value, new rules are *prepended* to old ones.

```bash
# Traffic to server's public IP is routed on the main table.
user@root ~ # ip rule add to 23.45.67.89/32 lookup main pref 30

# Create a new routing table for WireGuard to route all other traffic.
user@root ~ # ip rule add to all lookup 51820 pref 40

# Create the default route for the new table.
user@root ~ # ip route add default dev wg0 table 51820
# -or-
user@root ~ # ip route add 0.0.0.0/0 dev wg0 table 51820
```

1. Routing table 51820 is created for *wg0*. A routing table can be referenced by its number or string name.
2. *pref* means priority of a routing table.
3. The keyword *default* can be replaced with *0.0.0.0/0* that resembles the catch-all block of *allowed-ips*.

### *wg-quick* method

All the three methods above requires explicit route for server's public IP. However, such configuration is subject to peer roaming where public IP varies without prior notice. *wg-quick* utilizes an improved but obscure rule-based method: *fwmark* and *suppress_prefixlength*.

```
# Mark all traffic of *wg0*.
user@root ~ # wg set wg0 fwmark 51820

# Create a new routing table for WireGuard to route not marked traffic.
user@root ~ # ip -4 rule add not fwmark 51820 table 51820

# Create the default route for the new table.
user@root ~ # ip -4 route add 0.0.0.0/0 dev wg0 table 51820

# Suppress routing tables that the prefix length of the longest match
# is <= 0 in the main table.
# Namely skip the default entry (prefix length is 0).
user@root ~ # ip -4 rule add table main suppress_prefixlength 0
```

The resulting routing rules look like:

```
0:	from all lookup local 
32764:	from all lookup main suppress_prefixlength 0
32765:	not from all fwmark 0xca6c lookup 51820 
32766:	from all lookup main 
32767:	from all lookup default
```

1. 32764: specific routes in the *main* table except the *default* is respected.
2. 32765: all the rest goes to table 51820.
3. 32766: traffic to the server goes to the *main* table including the *default*.

#### suppress_prefixlength

Recall that an IP can be divided into the *network part* and the *host part*. Prefix length means the length of the network part. For example, the prefix length of *192.168.0.100/24* is 24 while that of *12.34.56.78/32* is 32.

Say we have a rule *from all lookup table 42 suppress_prefixlength 16*. When network stack hits the rule, it looks up routes in table 42 against destination IP as it would do to rule *from all lookup table 42*. When the longest match is found in table 42, it checks the route entry's prefix length. If the prefix is longer than 16 bits, it proceeds as usual. Otherwise (shorter than or equal to 16 bits), network stack go to check the next routing rule. That is to say, prefix length (of longest match) less than or equal to 16 is ignored (*suppressed*).

Here is a more detailed explanation. Suppose table 42 contains two entries:

```
# prefix 8
10.0.0.0/8 dev eth0
# prefix 24
10.1.1.0/24 dev eth1
```

And the destination IP is *10.1.1.1*. Obviously, the 2nd entry matches (24 > 16 > 8) and routes the packet to *eth1*. If *10.2.1.1*, then the 1st entry has the longest match but prefix is less than or equal to 16 bits. Hence table 42 does not route the packet.

`suppress_prefixlength` wants to set a minimal network prefix threshold for routes! The longer a prefix is, the more favored the route entry is: to avoid routes that are too general. Specially, threshold 0 just excludes the *default* entry (i.e. *default via 192.168.0.1 dev eth0*) as the prefix length is 0 (*0.0.0.0/0*). Therefore *suppress_prefixlength 0* above suppresses or skips the *default* entry in the *main* (254) table.

## Procedures Illustrated

Now let's go through an example to see how a packet is routed through rules of *wg-quick*:

1. *dig @8.8.8.8 www.bing.com* sends DNS query to *8.8.8.8*.
2. Network stack hits rule 32764 and checks *main* table.

   The *default* entry matches *8.8.8.8/32* but its prefix length is 0. So it's ignored.

   Suppose there are not any other routes.
3. Network stack checks the next rule 32765.

   The DNS packet is not marked as 51820 and successfully routed to 51820 table, going to *wg0*.
4. WireGuard received the DNS query packet.

   It encrypts the packet and forms a new UDP packet with destination IP as *endpoint* (server's public IP). This newly generated packet has *fwmark 51820*.
5. Network checks rule 32764 to route the new UDP packet.

   Similary the *default* entry is ignored.
5. Network stack checks rule 32765.

   Due to *fwmark 51820*, table *51820* is ignored either.
5. Network stack checks rule 32766.

   It's the *main* table again. This time there is no *suppress_prefixlength 0* limit. The WireGuard UDP packet is routed to *default* entry of *main* table, going out!

# Policy Routing Based on Bare IPs

In VPN Proxy section, traffic with *fwmark* is routed through the *main* table while that without the mark is routed through *wg0* table. Therefore, all traffic by default goes through WireGuard. Such all-in-one routing method is somewhat inefficient.

For example, we can route some traffic (i.e. to *www.baidu.com*) through the main table while some (i.e. to *www.block.com*) through *wg0* table. To achieve smart routing, we can mark former part while leaving the rest alone.

To set *fwmark* for a packet, we can resort to Iptables and IP set with the help of *dnsmasq* like [this post](/2018/03/26/policy-routing/). Obviously, there is no need to modify the routing rules and tables above.

The two methods followed are somewhat less elegant but deserves attention. They are mainly based on bare IPs.

## IP set

Firstly, we divide IPs into two sets. Traffic of one goes to WireGuard while the other not. IPs are from [APNIC Delegated List](http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest).

CN set:

```bash
user@tux ~ # wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > CNip.txt
```

non-CN set:

```bash
user@tux ~ # wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/\|ipv4\|/ && ! /\|CN\|/ && ! /\|\*\|/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > nonCNip.txt
```

We can set a *cron* job to update IP set.

### Static Route Method

One obvious method is adding all IPs of non-CN set to server peer's *allowed-ips* while removing the catch-all *0.0.0.0/0*. *wg-quick* will create the relevant routes in *main* table like:

```
1.0.128.0/17 dev wg0 proto kernel scope link src 10.0.0.1
```

However, that would fluff the *allowed-ips* list and increase maintainance burden.

Similarly we can also create routes in *main* table for IPs of CN set like:

```
1.0.32.0/19 via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304
```

I will choose the 2nd method as:

```bash
root@tux ~ # xargs -a /etc/wireguard/CNip.txt -I'{}' ip route add '{}' via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304
# -or-
root@tux ~ # wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' | xargs -I'{}' ip route add '{}' via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304
```

1. Don't worry about route entries if you manually bring down WireGuard.
2. We have *src* in the route entries, which would fail if *wlan0* or *wg0* do not have a IP the time we add routes (i.e. Wi-Fi router down).

   To simplify things, just remove *src* part. If you'd like, remove *metric* and *proto* as well.

Read more on [chnroutes](https://github.com/fivesheep/chnroutes) and [chinaroute路由表更新命令](https://gist.github.com/lixingcong/286144b3a521add58d8dcf045700963f)

### Geoip method

Instead of IP set, we can make use of *geopip* extension of *iptables*. This is much more simpler.

```
root@tux ~ # iptables -t mangle -A OUTPUT -o wlan0 -m geoip --dst-cc CN -j MARK --set-mark 51820
```

Recall the *ip rule* above, packets with *51820* mark will be routed by *main* table as usual.

1. [Gentoo Geoip](http://worldend.logdown.com/posts/304263-gentoo-install-iptables-geoip)
2. [woods/geoip.sh](https://gist.github.com/woods/25ef91a95da85bf10974)
3. [GeoIP based filtering with iptables](https://daenney.github.io/2017/01/07/geoip-filtering-iptables.html)

## Improve Local DNS Servers

Policy routing based on IPs highly depends on correct DNS resolving when visiting a domain. Accordingly, a custom clean DNS server (i.e. *8.8.8.8*) other than the one provided by ISP (prone to DNS poisoning) is required. This, in turn, would degrade performance when visiting domestic domains as custom DNS servers such as *8.8.8.8* usually return IPs geographically far.

Consequentially, smart DNS service that resolves domestic domains as usual (geographically nearby) but censorshipped domains correctly. There exist such smart DNS servers around. Just make sure traffic to it goes to WireGuard! In other words, organize IPs of smart DNS servers into the non-CN set. The goal is to set system DNS servers such that they resolve domestic domains into IPs of CN set while foreign domains into IPs of non-CN set.

Alternatively, a smart DNS server can be set locally such like [ChinaDNS](https://github.com/shadowsocks/ChinaDNS). An far better method is illustrated in [iproute2, iptables, dnsmasq](/2018/03/26/policy-routing/)

# wg-quick

*wg-quick* is just a wrapper script of commands of *ip* and *wg* above to faciliate the interface and WireGuard setup.

```bash
# wg-quick [ up | down | save ] [ CONFIG_FILE | INTERFACE ]
root@tux ~ # wg-quick save wg0
root@tux ~ # wg-quick up wg0
```

Please make sure */etc/wireguard/INTERFACE.conf* file exist.

*Attention*: the configuration file of *wg-quick* introduces a few extra arguments (i.e. internal IP, DNS) to format understood by *wg* in order to configure additional attributes. *wg-quick* handles the values that it understands, and then passes the rest directly to *wg setconf*. Especially, *wg-quick* update system route table automatically.

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

You can see that "Address", "DNS" and "SaveConfig" items are added. The DNS defined by *wg-quick* will replace local ones with help of *resolvconf*.

Here is an server example:

```
[Interface]
Address = 10.0.0.2/24
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 39814
PrivateKey = [SERVER PRIVATE KEY]

[Peer]
PublicKey = [CLIENT PUBLIC KEY]
AllowedIPs = 10.0.0.1/32  # Client's internal IP.
```

# Booting

## Gentoo wg-quick

Here I use *local.d* startup script. Please make sure the scripts are executable.

Up: */etc/local.d/01-wireguard.start*:

```bash
#!/bin/sh

echo "Starting WireGuard ..."
/usr/bin/wg-quick up wg0

echo "Adding CNip routes in background ..."
xargs -a /etc/wireguard/CNip.txt -I'{}' ip route add '{}' via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304 >/dev/null 2>&1 &
```

The route part can be moved to */etc/dhcpcd.exit-hook* as well. But on my system, *dhcpcd* reports:

>Error: Device for nexthop is not up

From the message, *wlan0* is not up before route entries added.

Down: */etc/local.d/01-wireguard.stop*:

```bash
#!/bin/sh

echo "Stopping WireGuard ..."
/usr/bin/wg-quick down wg0

echo "Removing CNip routes in background ..."
xargs -a /etc/wireguard/CNip.txt -I'{}' ip route del '{}' via 192.168.0.1 dev wlan0 proto dhcp src 192.168.0.123 metric 304 >/dev/null 2>&1 &
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
9  14  * * *     root    wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/wireguard/CNip.txt
```

Alternatively, we can set a script file (executable) into */etc/cron.daily/*.

```bash
#!/bin/sh

#/etc/cron.daily/CNip

wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/wireguard/CNip.txt
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
