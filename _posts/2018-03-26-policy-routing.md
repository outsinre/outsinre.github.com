---
layout: post
title: iproute2, iptables, dnsmasq
---

# iproute2

*iproute2* is successor of the old *ifconfig*, providing userspace utilities for controlling TCP / IP networking and traffic control in Linux, including routing, interfaces, tunnels, and traffic control.

Predefined identifiers (number and name pairs) are located under */etc/iproute2/*.

```bash
root@tux ~ # ip route
root@tux ~ # ip rule
```

Routes are divided into *table*s, default of which is the *254 main* table.

# dnsmasq

# ipset

# Policy Routing

mark by iptables
ip rule fwmark

# Refs
