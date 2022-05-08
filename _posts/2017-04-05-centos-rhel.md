---
layout: post
title: CentOS RHRL
---

1. toc
{:toc}

# Fedora RHEL CentOS

1. Fedora is a Linux distribution released under GPL.
2. Red Hat, Inc. copy the source code from Fedora, make modifications, release it as Red Hat Enterprise Linux (RHEL) distribution.

   Due to GPL, Red Hat, Inc. is obligated to disclose RHEL source code. But Red Hat can make money from its professional after-sale services.
3. CentOS (Community ENTerprise OS) is a Linux distribution built from the source of RHEL directly without any modification, with the intention of making CentOS binary compatible with RHEL such that library versions the same and binaries work on RHEL also work on CentOS.
   1. The main purpose of CentOS is to make RHEL available to pulbic users, benefitting from Red Hat's professional Linux service.
   2. Remove Red Hat, Inc. logo.
4. Fedora - RHEL - CentOS.

# Module

Module is a new packaging feature brought in from RHEL 8.

1. Package

   Traditional _.rpm_ file.
2. Group and Environment

   A large set of packages as a whole for specific system categories like X window, desktop, security, development, etc.
3. Module is a collection of a few packages, as a logical unit for a specific project, like Php and Nginx.
   1. Stream is the *upstream version*.
   2. Version is the *build version*.
   3. Profile is the sub-functionality of a module, similar to Gentoo's *tag*.

# rpm yum dnf

The relation between *rpm* (Redhat Package Manager) command and *yum* (Yellow dog Updater, Modified) command is analogous to *dpkg* and *apt-get*. *dnf* is the new version of *yum* with performance improvement and *module* feature. From RHEL 8 onwards, *dnf* replaces *yum* as the default manager.

Basically, *yum* is more intelligent than *rpm*. *rpm* wants to know the exact location of target *.rpm* file while *yum* only needs the package name. *yum* also takes care of dependencies. However, *rpm* is somehow a lightweight tool with concise output.

Check 'SPECIFYING PACKAGE NAMES' in the man page of *yum* to check how to pass globs arguments. The full name of a *.rpm* package is as follows:

```
# <name>-<version>-<release>.<os>.<arch>.rpm

vim-enhanced-7.4.160-4.el7.x86_64
# name:     vim-enhanced
# version:  7.4.160
# release:  4
# os:       el7
# arch:     x86_64
# suffix:   rpm

centos-release-7-4.1708.el7.centos.x86_64
# name:     centos-release
# version:  7
# release:  4.1708
# os:       el7.centos
# arch:     x86_64
```

1. Package *version* is also called *major version*, indicating the upstream source version from developers. Package *release* is also called *minor version*, indicating the final *.rpm* file by RHEL compliling/building/patching. Especially, RHEL may add customized patches or bug fixes to the major version, building a new *.rpm* package to the repository.
2. *os* may be el6 (rhel6), centos6, el5, suse11 etc.

By design, it is not possible to install difference versions of the same package alongside. The workaround is to make the new version a different package. For example, _python2_ and _python3_ are actually different package names.

## rpm

_rpm_ manages package file directly, maintaining a database of package information located under */var/lib/rpm/*. It does not care too much the package version but the RPM file you provided.

Query installed packages:

```bash
~ # rpm -qa name='globbing'
~ # rpm -qa | grep 'regex'

~ # rpm -qi pkg; rpm -qip pkg.rpm                     # query pkg information
~ # rpm -ql pkg; rpm -qlp pkg.rpm                     # list pkg files
~ # rpm -qR pkg; rpm -qRp pkg.rpm                     # list package it requires - dependencies

~ # rpm -qf /path/to/file                             # file owner
~ # rpm -qdf /path/to/file                            # file owner's man pages
```

Verify:

```bash
~ # rpm -V pkg; rpm -Vp pkg.rpm                       # verify
```

Verifying a package compares information about files installed locally with information about files taken from the package _metadata_ stored in the RPM database. Any discrepancies are displayed.

Install:

```bash
~ # rpm -ivvh --test /path/to/pkg.rpm      # dry run
~ # rpm -ivh /path/to/pkg.rpm              # install
~ # rpm --reinstall -vh /path/to/same.rom  # reinstall

~ # rpm -Uvh /path/to/pkg.rpm              # upgrade
~ # rpm -Fvh /path/to/pkg.rpm              # refresh
~ # rpm -Fvh --force /path/to/old.rpm      # downgrade
```

1. The path can be an ftp/http URL.
2. `-i, --install` is the general form of installing a package.
3. `-U, --upgrade` is the combination of `-i, --install` and `-e`. It firstly install a package and then erase earlier versions if present.

   `-U` does not require an earlier version is installed, so it can be a replacement of `-i`.
4. `-F, --freshen` is similar to `-U` but mandates an earlier version is already installed.
5. `--force`, `--replacepkgs`, `--replacefiles` and `--oldpackage` do the same thing.

Uninstall/Erase:

```bash
~ # rpm -evv pkg                     # uninstall a pkg
```

Kering:

```bash
~ # rpm --import /path/to/key.pub
~ # rpmkeys --import http://www.example.com/key.pub
```

Once imported, all public keys in the keyring can be managed as a package. Commands applied to a package also apply to a key like query, erase, information etc.

```bash
~ # ls /etc/pki/rpm-gpg/
~ # rpm -qa gpg-pubkey*
~ # rpm -q gpg-pubkey --queryformat "%{summary} ->%{version}-%{release}\n"
~ # rpm -qi gpg-pubkey-db42a60e
```

## yum

The *main* configuration of YUM is */etc/yum.conf* and specific repository configuration files are placed in */etc/yum.repos.d/*. Within the main configuration file, *cachedir* defines the location of cached *.rpm* packages (defaults to */var/cache/yum/$basearch/$releasever*).

By default, of the built-in repositories, only 'CentOS-Base.repo' is enabled.

1. `$releasever` indicates the main version. For example, the main version of CentOS 5.8 is 5.
2. `$arch` indicates the architecture like `x8_64`, `i386/i586/i686`.
3. `$basearch` is similar to `$arch`, but is *base* architecture: either `x86_64` or `i386`.

Cache:

```bash
~ $ yum help [sub-command]

~ # yum clean [ all | packages | metadata | expire-cache | rpmdb | plugins ]
~ # yum makecache
```

After any edits of _repo_ files, in order to clear any cached information, and make sure the changes are immediately recoginized, as root run `yum clean all`.

Query:

```bash
~ # yum history

~ # yum list [--all | --installed | --available | --upgrades] [pkg]
~ # yum info [--all | --installed | --available | --upgrades] [pkg] # more details
~ # yum search pkg

~ # yum repolist [--all | --enabled | --disabled]
~ # yum check-update
~ # yum upgrade [pkg]

~ # yum provides /path/to/file             # rpm -qf

~ # repoquery --whatprovides '*bin/grep'
~ # repoquery --list pkg
```

1. _update_ is synonym to _upgrade_.
2. *list --updates* is almost the same as *check-update*. As the command form implies, *check-update* is useful in Shell script while *list --updates* is for humans on the command line.

   Please pay attention, their exit status code difference.
3. 'yum provides' only search which package provides the pathname. 'rpm -qf' requires that the package is installed or existence of the *.rpm* file.

   Tp speed up the search, use 'repoquery' from 'yum-utils' package. Especially, it list all files provided by a package even it is not installed.

Install:

```bash
~ # yum [-y] install pkg1 pkg2

# will install dependency automatically
~ # yum install /path/to/pkg.rpm

~ # yum reinstall pkg1 pkg2

~ # yum upgrade pkg1 pkg2

~ # yum remove pkg1 pkg2
```

To install packages for _group_ or _environment_ set:

```bash
~ # yum group summary
~ # yum group [ list | info |install | upgrade | remove | mark ] [grp]
```

## dnf

_dnf_ is compatible with but more powerful than _yum_. One of the newest feature provided is _module_.

```bash
~ # dnf module -h
~ # dnf module list nginx
~ # dnf module info nginx

# NAME:STREAM:VERSION:CONTEXT:ARCH/PROFILE
~ # dnf module info nginx:1.16:8010020191122190044:cdc1202b:x86_64/common
~ # dnf module install nginx:1.14/common
```

The purpose of *module* is to manage package versions and profiles in an integrated way. The following two methods is equivalent:

```bash
~ # dnf module install nginx
~ # dnf install nginx
```

The only difference is that `dnf module` can provide better control when selecting pakage stream, version, arch etc.

The following example shows how to control repo with *dnf* (applying to *yum* as well):

```bash
~ $ dnf help repolist 
~ $ dnf repolist [--all | --enabled | --disabled]

~ $ sudo dnf config-manager --add-repo /etc/yum.repos.d/oracle-epel-ol8.repo

~ $ grep enabled /etc/yum.repos.d/oracle-epel-ol8.repo
~ $ sudo dnf config-manager --set-enabled ol8_developer_EPEL
~ $ duso dnf config-manager --set-disabled ol8_developer_EPEL

~ $ dnf --enablerepo=repo1,repo2 -disablerepo=repo3,repo4 install pkg
```

# Repositories

Of the [3rd party repositories](https://wiki.centos.org/AdditionalResources/Repositories), IUS and Remi (both depend on EPEL) is recommended over Webtatic.

```bash
~ # yum repolist [all|enabled|disabled]

~ # yum-config-manager --add-repo <repo-url>
~ # yum repolist [all|enabled|disabled]
~ # yum-config-manager --disable <repo-id>
~ # yum-config-manager --enable <repo-id>
```

To remove a repo, just remove the file within */etc/yum.repos.d*.

## EPEL not from RHEL ##

EPEL (Extra Packages for Enterprise Linux) is open source and free community based repository project from [Fedora](https://fedoraproject.org/wiki/EPEL) team which provides 100% high quality add-on software packages.

```bash
~ # yum info epel-release
~ # yum install epel-release
```

To specify repository when installing package:

```bash
~ # yum --enablerepo=epel install nginx
```

The `--enablerepo` option overides the permanent option setting in the */etc/yum/\*.repo* files for only the current command. `--disablerepo` does the opposite for enabled repos.

## [Inline with Upstream Stable (IUS)](https://ius.io/GettingStarted/)

```bash
~ # rpm -Uvh https://centos7.iuscommunity.org/ius-release.rpm
```

Read [IUS Usage](https://ius.io/Usage/) for more information.

## Webtatic (DEPRECATED)

The [Webtatic Yum repository](https://webtatic.com/projects/yum-repository/) is a CentOS/RHEL repository containing updated web-related packages like *git*.

```bash
~ # rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

## [CentOS-5.8](https://unix.stackexchange.com/q/359902)

CentOS-5 reached the end of its lifecycle as of March 31, 2017. The official [baseurl](http://mirror.centos.org/centos/5/readme) is deprecated and moved to a backup location [Vault](http://vault.centos.org/5.8/). Packages there is no longer maintained or upgraded, and may suffer from security risks.

However, we sometimes have to maintain ancient releases for whatever reasons. To install packages through *yum*, [point repositories to the new location](https://www.centos.org/forums/viewtopic.php?t=62130): either manually edit */etc/yum.repos.d/CentOS-Base.repo* or install *centos-release-5-8.el5.centos.x86_64.rpm* file.

1. For each section (i.e. Base) Comment out both the *mirrorlist=* and *baseurl=* lines.
2. Add a new baseurl line:

   ```
   baseurl=http://vault.centos.org/5.8/os/$basearch/
   ```

   1. Replace */os/* part approriately for other sections like */updates/*.
   2. It is highly recommended to replace 5.8 with 5.11, upgrading the system to CentOS 5.11

   Alternatively, install the corresponding repository package:

   ```
   root@tux ~ # wget http://vault.centos.org/5.11/os/x86_64/CentOS/centos-release-5-11.el5.centos.x86_64.rpm
   root@tux ~ # rpm -Uvh centos-release-5-11.el5.centos.x86_64.rpm
   ```

   For other repositories like EPEL, Webtatic etc., find their archive URL and repeat the same procedures.
3. Update

   ```bash
   root@tux ~ # rpm -q centos-release; cat /etc/redhat-release; cat /etc/centos-release
   root@tux ~ # yum clean all; yum makecache
   root@tux ~ # yum list updates; yum update
   root@tux ~ # reboot
   root@tux ~ # yum install epel-release; yum install git
   ```

Here is an example of manually edited Centos-Base.repo:

```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
baseurl=http://vault.centos.org/5.11/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#released updates 
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
baseurl=http://vault.centos.org/5.11/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
baseurl=http://vault.centos.org/5.11/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
baseurl=http://vault.centos.org/5.11/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
#baseurl=http://mirror.centos.org/centos/$releasever/contrib/$basearch/
baseurl=http://vault.centos.org/5.11/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
```

## Upgrade System ##

```bash
~ # dnf check-update
~ # dnf upgrade kernel
~ # dnf upgrade
~ # reboot
```

# rpmbuild

For how to build RPM package, refer to "logging/filebeat-zookeeper-kafka".

## Extracting RPM

Upon receving a *.rpm* file, we can extract the contents with *rpm2cpio* and *cipo*:

```bash
~ $ mkdir dst; cd dst
~ $ rpm2cpio /path/to/pkg.rpm | cpio -idmv
~ $ find .
```

1. `-i`: extract;
2. `-d`: make directories;
3. `-m`: preserve modification time;
4. `-v`: verbose.

# locale #

Redhad-based distributions places *locale* customization in */etc/locale.conf*. Check `man locale.conf`.

# SSH

```bash
~ # ssh-add
~ # ssh-copy-id -p 12345 root@12.23.56.78
# add an entry to ~/.ssh/config
```

# Create User Account

Check [useradd](/2015/03/25/gentoo-installation/).

```bash
~ # ssh-copy-id -p 12345 username@12.34.56.78
# add an entry to ~/.ssh/config
```

# Pip/Virtualenv

```bash
~ # yum install python-pip python-virtualenv
~ # pip install -U pip
```

# SS

```bash
~ # mkdir -p ~/opt/pyvenv2.6
~ # virtualenv --system-site-packages ~/opt/pyvenv2.6
~ # pip install git+https://github.com/shadowsocks/shadowsocks.git@master
# or
~ # wget https://github.com/shadowsocks/shadowsocks/archive/master.zip
~ # pip install master.zip
```

## Server Json

```bash
~ # mkdir -p /etc/shadowsocks
~ # vi /etc/shadowsocks/server.json
```

Fill in the fileds without any comments:

```json
{
    "server":"::0",
    "server_port":3900,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open":true,
    "workers":2
}
```

## TCP Fast Open

```bash
~ # vi /etc/sysctl.d/10-tcp-fast-open.conf
# net.ipv4.tcp_fastopen = 3
~ # sysctl -p /etc/sysctl.d/10-tcp-fast-open.conf
```

1. Unfortunately CentOS 7 does not satisfy lower kernel bound - 3.7.1.
2. It should be enabled on cleint and server simutaneously.

### A quick test

```bash
~ # ssserver -c /etc/shadowsocks/server.json --user nobody -d start -vv
```

## Systemd Unit ##

>Please reinstall Shadowsocks into system-wide location. The above *virtualenv* version was just a test.

```bash
# vi /etc/systemd/system/shadowsocks.service
```

### simple Type

```
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
Type=simple
PermissionsStartOnly=true
ExecStartPre=/usr/bin/mkdir -p /run/shadowsocks
ExecStartPre=/usr/bin/chown nobody:nobody /run/shadowsocks
ExecStartPre=/usr/bin/su -s /bin/bash -c "/usr/local/bin/kcptun-server -c /etc/shadowsocks/kcptun.json &" nobody
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks/server.json
Restart=on-abort
User=nobody
Group=nobody
UMask=0027

[Install]
WantedBy=multi-user.target
```

Please be noted that:

1. *simple* Type is used so that Shadowsocks does not *forking* itself so that daemon mode is handed over to *systemd*.
   1. Do *not* add `-d start/stop/restart` arguments to ExecStart
   2. No PIDFile and `--pid-file` required. Read more on [doesn't make sense to set PIDFile= by simple services](https://bugzilla.redhat.com/show_bug.cgi?id=723942#c4).
2. You may find a special line with *kcptun*. Neglect it now for it will be discussed later on.
3. [ref1](https://yuyii.com/2015/12/28/shadowsocks-systemd/) and [ref2](http://www.leyar.me/Compile-shadowsocks-libev-in-CentOS7/).

### forking Type

```
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
Type=forking
PermissionsStartOnly=true
PIDFile=/run/shadowsocks/ss.pid
ExecStartPre=/usr/bin/mkdir -p /run/shadowsocks
ExecStartPre=/usr/bin/chown nobody:nobody /run/shadowsocks
ExecStartPre=/usr/bin/su -s /bin/bash -c "/usr/local/bin/kcptun-server -c /etc/shadowsocks/kcptun.json &" nobody
ExecStart=/usr/bin/ssserver -d start -c /etc/shadowsocks/ss.json --pid-file /run/shadowsocks/ss.pid --log-file /run/shadowsocks/ss.log
Restart=on-abort
User=nobody
Group=nobody
UMask=0027

[Install]
WantedBy=multi-user.target
```

## [kcptun](https://github.com/xtaci/kcptun)

*kcptun* estabilishs a KCP tunnel to speed up TCP connection by encapsulating TC packets within UDP flooding. It may introduce twice or even triple traffic depending on arguments choosen. You are advised to use it only in WI-FI environment.

To add *kcptun* support, just insert a line into Systemd service file:

```
ExecStartPre=/usr/bin/su -s /bin/bash -c "/usr/local/bin/kcptun-server -c /etc/shadowsocks/kcptun.json &" nobody
```

Attention to trailing `&` of `-c` argument.

