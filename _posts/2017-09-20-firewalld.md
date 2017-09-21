---
layout: post
title: firewalld
---

# firewalld VS iptables

1. Stack layers

   1. firewalld/iptables service -> iptables command -> kernel packet filter (netfilter).
   2. We should know the difference between a *service* and shell *command* in terms of *iptables*.
   3. *firewalld* still requires *iptables* command underneath.

   Hence, no matter which service you prefer, *iptables command* is essential.

   ![firewall](/assets/firewall.png)

   At the very top, sits the GUI tool.
2. Check packages

   ```bash
   ~ # yum list iptables iptables-services firewalld
   ```

3. Specially, firewalld introduces *zone* to defines the level of trust for network connections, which resembles Microsoft Windows firewall.

   Rules are attached to a zone.

# mask iptables service

Switch to firewalld.

```bash
~ # systemctl stop iptables-services
~ # sysremctl disable iptables-services
~ # systemctl mask iptables-services
```

# Install

```bash
~ # yum install iptables firewalld
~ # systemctl enable firewalld
~ # systemctl start firewalld
```

>Don't break SSH connection as the default SSH in default zone is 22 that is usually changed by administrator, otherwise you could no longer SSH into server.

# Status

```bash
~ # systemctl status firewalld (or firewall-cmd --state)
~ # firewall-cmd --get-zones/get-services
~ # firewall-cmd --list-all/list-ports/list-services
~ # firewall-cmd --get-default-zone/get-active-zones
~ # firewall-cmd --reload
```

1. Default zone is *public* to which unmatched traffic would be directed.

   For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted. 
2. You activate a zone by binding a network interface or source IP address range(s) to it. Any firewall rules in the zone then apply to that network interface or IP address range(s).

# service and port

1. service is pre-defined well-known ports like http, https etc.

   Check */usr/lib/firewalld/services* XML definitions. You shouldn't edit these.
2. If we want a different service (i.e. ssh) port other than default (22), try:
   1. Copy */usr/lib/firewalld/services/ssh.xml* to */etc/firewalld/services*; change port there.
   2. Adding port directly to default zone.

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <service>
     <short>SSH</short>
     <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
     <port protocol="tcp" port="12345"/>
   </service>
   ```

# Adding ports

```bash
~ # firewall-cmd --zone=public --add-port=12345/tcp
~ # firewall-cmd  --permanent --zone=public --add-port=12345/tcp
~ # firewall-cmd --reload (opt)
```

1. Take effect immediately at rumtine without *reload*.
2. Take effect accross *reload*. Since we have the runtime command at first, no `--reload` is required.
3. `--zone` can be ommited unless you want to change other zones.

# Adding services

```bash
 ~ # firewall-cmd --permanent --zone=public --add-service={https,http}
```

# Create/modify services

Only a permanent service can be created.

```bash
~ # firewall-cmd --permanent --new-service-from-file=/path/to/service.xml --name=myservice (using an existing service)
# or
~ # firewall-cmd --permanent --new-service=myservice (create an empty service)
~ # firewall-cmd --info-service=myservice
```

Modify it after creation:

```bash
~ # firewall-cmd --permanent --service=myservice --set-description=description
~ # firewall-cmd --permanent --service=myservice --set-short=description
~ # firewall-cmd --permanent --service=myservice --add-port=portid[-portid]/protocol
~ # firewall-cmd --info-service=myservice
```

Adding the new service to a zone:

```bash
~ # firewall-cmd --permanent --zone=public --add-service=myservice
```

I think the easiest way is to copy an existing service XML to */etc/firewalld/services/myservice.xml* and edit that file directly.

# Drop a port

```bash
~ # firewall-cmd --permanent --zone=drop --add-port=12345/tcp
```

Then access to port 12345 would be dropped.

# Finally

```bash
~ # firewall-cmd --reload
```

# Refs

1. [firewalld](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7)
2. [Understanding Firewalld in Multi-Zone Configurations](https://www.linuxjournal.com/content/understanding-firewalld-multi-zone-configurations)
