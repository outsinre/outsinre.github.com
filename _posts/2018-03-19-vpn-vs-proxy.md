---
layout: post
title: VPN versus proxy
---

VPN and proxy both can, more or less, serve as a man in the middle to target network. VPN is meant to establish private (encryption) connection to target network over the Internet. Proxy is just a traffic relay though developers may decide to implement security enhancement. Therefore, VPN can be an alternative to proxy for security concern.

Here is a simple flow illustration:

>VPN: application - VPN client (encrypt traffic) - the Internet - VPN server (decrypt traffic) on target network

>Proxy: application - proxy client (opt) - proxy server - target network

>VPN as proxy: application - VPN client (encrypt traffic) - the Internet - VPN server (decrypt traffic) - target network

# Proxy

1. With proxy, traffic appears to come from somewhere else hiding your IP from target network.
2. HTTP proxy is only designed for web traffic while SOCKS proxy is indifferent to the type of traffic (HTTP, FTP, BT etc.) that passes through it.

   SOCKS is slower than HTTP as it require more overhead.
3. HTTP proxying has a different usage model in mind, the CONNECT method allows for forwarding TCP connections like SOCKS proxy.

   HTTPS proxy requires the CONNECT method.
4. SOCKS works on TCP mostly. But SOCKS5 provides a means for UDP packets to be forwarded.

## Security

1. As mentioned above, proxy by nature, does not protect traffic security.
2. BT over proxy would reveal your IP. Try BTGuard instead.

# VPN

1. VPN encrypt connection to provide private communication by design.
2. VPN creats a virtual interface at the operating system level, and the VPN connection captures the entire network connection of the device it is configured on.

   Proxy, on the other hand, needs enabled on application basis.
3. Encryption and decryption incur much more overhead than SOCKS proxy, let alone HTTP proxy.
4. UDP VPN is faster than TCP VPN but unreliable.
