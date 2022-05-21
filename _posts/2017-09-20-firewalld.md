---
layout: post
title: firewalld
---

# firewalld VS iptables

1. Stack layers

   1. firewalld/iptables service -> iptables command -> kernel packet filter (netfilter).
   2. We should know the difference between a *service* and shell *command* in terms of *iptables*.
   3. *firewalld* still requires *iptables command* underneath.

   Hence, no matter which service you prefer, *iptables command* is essential.

   ![firewall](/assets/firewall.png)

   At the very top, sits the GUI tool.
2. Check packages

   ```bash
   ~ # yum list iptables iptables-services firewalld
   ```

3. Specially, firewalld introduces *zone* to defines the level of trust for a connection, IP source or interface, which resembles Microsoft Windows firewall. Rules are created within a zone. A zone is bound to one or more interfaces, IP sources or interfaces, but a connection, interface or IP source can *only* be part of one zone.

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
~ # systemctl status firewalld
```

>Don't break SSH connection as the default SSH port in default zone is 22 that is usually changed by administrator, otherwise you could no longer SSH into server.

# Status

```bash
~ # firewall-cmd --reload
~ # firewall-cmd --state

# print zone names
~ # firewall-cmd --get-zones
# print all zone details
~ # firewall-cmd --list-all-zones
# default is 'pulbic'
~ # firewall-cmd --get-default-zone
# zones that has bindings
~ # firewall-cmd --get-active-zone

# print all service names
~ # firewall-cmd --get-services
# print a service detail
~ # firewall-cmd --info-service https

# print objects in a zone
~ # firewall-cmd --zone=public --list-services
~ # firewall-cmd --zone=public --list-ports
~ # firewall-cmd --zone=public --list-interfaces
~ # firewall-cmd --zone=public --list-sources
# print all objects in a zone
~ # firewall-cmd --zone=public --list-all
~ # firewall-cmd --info-zone public
```

1. Default zone is *public* to which unmatched traffic would be applied.

   For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted. 
2. We *activate* a zone (`--get-active-zones`) by binding a network interface, source IP address range(s) or ports to it.

As *firewalld* sites on top of *iptables*, we can manipulate underlying *iptables* rules with *firewall-cmd* directly with the `--direct` option.

```bash
~ # firewall-cmd --direct --get-all-chains
~ # firewall-cmd --direct --get-chains nat

~ # firewall-cmd --direct --get-all-rules
~ # firewall-cmd --direct --get-rules nat
```

Attention please; do not use *iptables* command to manipulate *firewalld* rules as that would make things complicated.

# service and port

1. Services are pre-defined well-known ports like http, https etc.

   Check */usr/lib/firewalld/services* XML definitions. You shouldn't edit those files.
2. To edit a servce (i.e. change ssh port):

   Copy */usr/lib/firewalld/services/ssh.xml* to */etc/firewalld/services/*; change port there.

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <service>
     <short>SSH</short>
     <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
     <port protocol="tcp" port="12345"/>
   </service>
   ```

   Alternatively, adding port directly to default zone.

# Adding ports

```bash
~ # firewall-cmd --zone=public --add-port=12345/tcp
~ # firewall-cmd --reload (opt)

~ # firewall-cmd --permanent --zone=public --add-port=12345/tcp
```

1. Take effect immediately at rumtine without *reload*.
2. As we have the runtime command at first, no `--reload` is required.
3. Take effect accross *reload*.
4. `--zone` can be ommited unless you want to change other zones.

# Adding services

```bash
 ~ # firewall-cmd --permanent --zone=public --add-service=http --add-service=https
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
~ # firewall-cmd --permanent --service=myservice --remove-port=portid[-portid]/protocol
~ # firewall-cmd --info-service=myservice
```

Adding the new service to a zone:

```bash
~ # firewall-cmd --permanent --zone=public --add-service=myservice
```

I think the easiest way is to copy an existing service XML to */etc/firewalld/services/myservice.xml* and edit that file directly.

# Disallow a port

```bash
~ # firewall-cmd --permanent --zone=drop --add-port=12345/tcp
```

Then access to port 12345 would be dropped.

# Finally

```bash
~ # firewall-cmd --reload
```

# Refs

1. [firewalld centos 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7)
2. [Understanding Firewalld in Multi-Zone Configurations](https://www.linuxjournal.com/content/understanding-firewalld-multi-zone-configurations)
