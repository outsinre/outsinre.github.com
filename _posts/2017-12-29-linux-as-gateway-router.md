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
   root@tux / # ip route add default 192.168.10.0/24 via 192.168.10.101 dev eth1
   root@tux / # route add 192.168.10.0 netmask 255.255.255.0 gw 192.168.10.101 dev eth1
   ```

   *192.168.10.0/24* is the inernal network IP block while *192.168.10.101* is IP of *eth1* from gateway Linux.
2. Turn on IP forwarding on gateway Linux.

   Read [IP Forwarding = when and why is this required?](https://serverfault.com/q/248841) and [What is kernel ip forwarding?](https://unix.stackexchange.com/q/14056). It is basically for inter-network (except for VLAN router) packet transfer and not for inter-interface transfers. By this I mean that if two interfaces are on the same network then we dont need to enable IP forwarding on the server. Also, if one is virtual interface, then we don't need IP forwarding either.

   It simply determines behaviour when receiving a packet that is NOT destined for the receiving host. Without forwarding, the packet is dropped. With forwarding, a routing decision determines which interface it should be forwarded to.

   Locally generated packets does not traverse FORWARD chain.

   ```bash
   root@tux / # sysctl -w net.ipv4.ip_forward=1
   root@tux / # cat /proc/sys/net/ipv4/ip_forward
   root@tux / # iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
   ```

   The FORWARD rule allows forwarding internal networking traffic to *eth0*.
3. Turn on [MASQUERADE NAT](http://www.tldp.org/HOWTO/IP-Masquerade-HOWTO/index.html) on gateway Linux

   Generally, MASQUERADE is required when many LAN private IP's traffic sits behind and goes through a (WAN public) single IP, namely a many-to-one mapping.
   
   ```bash
   root@tux / # iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE (poor NAT)
   # or
   root@tux / # iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source XXX.XXX.XXX.XXX (better NAT)
   ```

   *XXX.XXX.XXX.XXX* is IP of *eth0* from gateway Linux.
4. Read more on [Wireshark post](/2017/12/12/wireshark/).

# Reference

1. [NAT gateway](http://how-to.wikia.com/wiki/How_to_set_up_a_NAT_router_on_a_Linux-based_computer)
2. [LinuxTutorialIptablesNetworkGateway](http://www.yolinux.com/TUTORIALS/LinuxTutorialIptablesNetworkGateway.html).