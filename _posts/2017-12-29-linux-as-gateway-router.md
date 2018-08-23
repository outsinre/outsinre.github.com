---
layout: post
title: Linux as Gateway Router
---

1. toc
{:toc}

# Principal

Suppose a Linux box such as a personal computer has two networking cards (i.e. *eth0* and *eth1*). One card (i.e. *eth0*) connects to the Internet by whatever means like PPoE dial-up, ADSL, static IP etc. The other one (i.e. *eth1*) stays within a LAN block.

>Nowadays, it's common that a networking card supports two inferfaces.

The Linux box can serve as a gateway/router for other devices (i.e. virtual machines using NAT mode) LAN with Iptables support. To [achieve this functionality](https://serverfault.com/a/431607):

1. Enable packet forwarding.
2. Iptables FORWARD support.
3. Iptables NAT support.

Actually, what this post does is to set up the routing functionality on Linux boxes, simulating a router.

1. A layer 3 router maintains a table mapping IPs to ports while a layer 2 switch maintains a table mapping mac addresses to ports. However, a switch learns mac address and port pair by broadcasting packets at first while a router usually needs manual configuration. After a table entry is learned, the switch formards upcoming packet the exact port instead of broadcasting.
2. A Wi-Fi router comprises of two parts, namely a wireless access point and a routing table. The access point handles traffic among WLAN devices while the routing table fowards traffic in and out.

## [IP Forwarding](https://unix.stackexchange.com/a/14058) at Layer 3

To server as a gateway, the Linux box must forward a packet not destined for local interfaces (i.e. neither *eth0* nor eth1*). In this post, such packets are named as "foreign*.

If the box receives a packet with destination address not configured locally, it could either drop or forward it. It is intuitive to drop a strange packet. Alternatively, the box (i.e. router) may choose to forward the packet to some interface letting the packet traverse it as a router usually resides across between multiple network blocks.

*forwarding* is a synonym for *routing* meaning forwarding a foreign packet according to *routing tables* similar to routing locally generated packets. In other words, packet forwarding is routing tables' business and has nothing to do with Iptables.

Read more at [IP Forwarding = when and why is this required?](https://serverfault.com/q/248841). It is basically for inter-network (except for VLAN router) packet transmission and not for that of inter-interface. If two interfaces are on the same network then IP forwarding is not involved (rare case though). Additionally, if one is virtual interface, IP forwarding is not needed either.

For example, traffic between two devices on the same network does not require forwarding/routing at layer 3. Instead, it is handled at layer 2 by switches, access points of Wi-Fi router, or direct Ethernet wire.

The box should enable `net.ipv4.ip_forward`.

## Iptables FORWARD

Once forwarded, the foreign packet will be examined by FORWARD chain of Iptables. Please be noted, keyword FORWARD is not meant to forward a packet but apply Iptables rules against the forwarded packets from routing tables.

The box should allow forwarded packets to go outside by `-j ACCEPT`.

## Iptables NAT

Before a foreign packet is forwarded outside, the box should also modify the source IP to that of the outgoing interface (i.e. *eth0*), namely to set up NAT by Iptables.

After examined by Iptables FORWARD, the source IP remains of the original one (that of the foreign device). If it is left alone, the destination device would attempt to send back packets directly to the foreign device without intervention of the box.

Iptables NAT support will maintain an kernel mapping between the source IP and that of outgoing interface, such that destination device can reply to the box instead.

`-j MASQUERADE` or `-j SNAT` should be enabled.

# Set Device Gateway

Set LAN devices' gateway to IP of *eth1* on the box.

For Linux device, this can be done with *ip route* or *route* command line. Say LAN block is *192.168.10.0/24* while *192.168.10.1* is IP of *eth1* of the box.

```bash
root@tux / # ip route add default 192.168.10.0/24 via 192.168.10.1 dev eth1
# -or-
root@tux / # route add 192.168.10.0 netmask 255.255.255.0 gw 192.168.10.1 dev eth1
```

It is not unusual that home devices' (i.e. smartphone) default gateway is set to *192.168.0.1* (IP of Wi-Fi router).

# Set the Box

1. Enable IP forwarding.

   It simply determines behaviour when receiving a packet for another network. Without forwarding, the packet is dropped. With forwarding, routing tables are consulted to determine to which interface it should be forwarded.

   Permanent across reboots:

   ```bash
   root@tux / # echo 'net.ipv4.ip_forward=1' > /etc/sysctl.d/20-ip_forward.conf
   ```

   For current session:

   ```bash
   root@tux / # sysctl -w net.ipv4.ip_forward=1
   # -or-
   root@tux / # sysctl -p /etc/sysctl.d/20-ip_forward.conf
   # Check
   root@tux / # cat /proc/sys/net/ipv4/ip_forward
   ```

2. Enable Iptables FORWARD

   Please be noted locally generated packets are not examined by FORWARD chains.

   ```bash
   root@tux / # iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
   ```

   This FORWARD rule lets forwarded packets go outside.
3. Enable Iptables NAT

   Turn on [MASQUERADE NAT](http://www.tldp.org/HOWTO/IP-Masquerade-HOWTO/index.html) on the box. Generally, MASQUERADE is required when many LAN private IPs' traffic sits behind and goes through a public single IP (i.e. Wi-Fi router), namely a many-to-one relationship.
   
   ```bash
   root@tux / # iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE (poor NAT)
   # or
   root@tux / # iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source <IP of eth0> (better NAT)
   ```

4. Read more on [Wireshark post](/2017/12/12/wireshark/).

# Reference

1. [NAT gateway](http://how-to.wikia.com/wiki/How_to_set_up_a_NAT_router_on_a_Linux-based_computer)
2. [LinuxTutorialIptablesNetworkGateway](http://www.yolinux.com/TUTORIALS/LinuxTutorialIptablesNetworkGateway.html).
