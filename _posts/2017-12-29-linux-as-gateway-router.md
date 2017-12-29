---
layout: post
title: Linux as Gateway Router
---

# Senario

Say you have a Linux with two networking cards (i.e. *eth0* and *eth1*). One card (i.e. *eth0*) connects to the Internet by whatever means like PPoE dial-up, ADSL, static IP etc. The other one (i.e. *eth1*) stays within an internal network with many devices.

>Nowadays, it's common that a networking card supports two inferfaces.

# Goal

The interface with WAN networking can serve as a gateway router for other devices in internal network with Iptables support.

# Outline

1. Set internal devices' gateway to Linux's *eth1* IP.

   For Linux device, this can be done with *ip route* or *route* command line.

   ```bash
   root@tux / # ip route add 192.168.10.0/24 via 192.168.10.101 dev eth0
   root@tux / # route add 192.168.10.0 netmask 255.255.255.0 gw 192.168.10.101 dev eth0
   ```

   *192.168.10.0/24* is the inernal network IP block while *192.168.10.101* is IP of *eth1* from gateway Linux.
2. Turn on IP forwarding on gateway Linux.

   ```bash
   root@tux / # sysctl -w net.ipv4.ip_forward=1
   # or
   root@tux / # echo 1 > /proc/sys/net/ipv4/ip_forward
   root@tux / # iptables -A FORWARD -i eth1 -j ACCEPT
   ```

   The FORWARD rule allows forwarding internal networking traffic to *eth0*.
3. Turn on NAT on gateway Linux

   ```bash
   root@tux / # iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE (poor NAT)
   # or
   root@tux / # iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source XXX.XXX.XXX.XXX (better NAT)
   ```

   *XXX.XXX.XXX.XXX* is IP of *eth0* from gateway Linux.
4. Read more on Wireshark post.

# Reference

1. [NAT gateway](http://how-to.wikia.com/wiki/How_to_set_up_a_NAT_router_on_a_Linux-based_computer)
2. [LinuxTutorialIptablesNetworkGateway](http://www.yolinux.com/TUTORIALS/LinuxTutorialIptablesNetworkGateway.html).