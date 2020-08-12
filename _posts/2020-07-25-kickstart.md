---
layout: post
title: Kickstart
---

1. toc
{:toc}

# Kickstart

## Outline

Kickstart 支持在内网设定装机服务器，所有需要装机的设备连接服务器，实现从网络启动，实现自动化装机。本文介绍简化版用例，基于 Kickstart 定制 ISO 文件，实现自动装机。

主要定制包括：

1. 自动装机。
2. 增、删、改内置 rpm 包。
3. 定制系统参数，如 DNS, 网络、启动服务等。

   这部分在主系统装完后操作。
4. 装完自动重启。

## Preparation

所有的操作在同版本 CentOS 下进行。例如要定制 CentOS 7.8 的 ISO, 则最好在 CentOS 7.8 系统下操作。

```bash
yum install rsync createrepo genisoimage isomd5sum pykickstart
```

1. genisoimage 替换老旧的 mkisofs.
2. isomd5sum (implantisomd5) 用于给 ISO 打 md5 值。
3. pykickstart (ksvalidator) 用于验证 ks.cfg 语法。

## Procedures

### 1 Mount Official 7.8 ISO image

挂载官方 ISO 文件。

```bash
lsblk
ll /dev/cdrom

mkdir -p /mnt{cdrom,iso}
mount /dev/cdrom /mnt/cdrom
```

### 2 Customize ISO Contents

定制 ISO 内容。

```bash
mkdir -p /mnt/iso
rsync -a /mnt/cdrom /mnt/iso

rm -rf /mnt/iso/repodata/*

cp /path/to/my-pkg.rpm /mnt/iso/Packages/
yum install -y --downloadonly --downloaddir=/mnt/iso/Packages/ pkg1 pkg2 pkg3
```

1. 清空 repodata 目录，后面会重建。
2. 本例主要添加额外安装包到 Packages 目录。

### 3 Customize comps.xml

对 ISO 改动后，需要重新生成 comps.xml. comps.xml 是 XML 文件，有 2 个大 tag, 即 'group' 与 'environment'. group 定义 rpm 包的集合，里面的 rpm 包紧密相关，而 environment 是 group 的集合，里面包含的 group 全部安装进系统。例如，CentOS 的 'minimal' environment 里，默认只有 core 这个 group. 在系统中通过 'dnf group install' 安装 group 与 environment 集合。

本例中，主要是往 minimal environment 里增加内置 rpm 包，所以需要把这些新的 rpm 包名加到 comps.xml 里。

```bash
mkdir -p /mnt/iso/douyu
cp /mnt/cdrom/repodata/*c7-minimal-x86_64-comps.xml /mnt/iso/douyu/c7-minimal-x86_64-comps.xml

# emacs /mnt/iso/douyu/c7-minimal-x86_64-comps.xml
#      <packagereq type="default">my-pkg</packagereq>
#      <packagereq type="default">pkg1</packagereq>
#      <packagereq type="default">pkg2</packagereq>
#      <packagereq type="optional">pkg3</packagereq>
```

comps.xml 定制说明：

1. 添加 pkg.rpm 包时，需同时添加其依赖包，否则无法安装。
2. 为添加的 rpm 包定义 group.
   1. 可以直接修改 'core' group.
   2. 可以仿照 core 自定义一个新 group.

   本例是直接修改 core. 如果是自定义 group, 则在 environment 环境后，添加 `<groupid>my-group</groupid>`, 表示把自己的 group 包括进对应的 environment 里，以便安装。没有添加进 environment 里的 group 是不会安装的。
3. 为新 rpm 包添加 'packagereq' tag.

   type 参数为 'default' 表示默认会安装（默认打勾），'mandatory' 表示必须要安装，不可取消打勾，'optional' 表示可选安装（默认末打勾）。

### 4 Generate ks.cfg

Kickstart 配制文件生成方案：

1. Kickstart 提供了图形界面的配制工具 system-config-kickstart, 可先用图形界面生成、更新 ks.cfg 模版，再作细节优化。
2. 先用图形界面手动安装一遍操作系统。安装完毕，anaconda 会在 /root 下生成 anaconda-ks.cfg 的文件，记录本次手动安装过程所对应的 Kickstart 配制文件。

两种方案中，前者可定制的参数比较全面，后者因为是实操作，更可靠。需要注意的是，如果第 2 种方法是在 VM 里手动安装，则可能存在一些参数与实际物理机安装不同。例如 /root 下的 anaconda-ks.cfg 文件里，往往在分区参数上，分区大小是固定的（`part / --fstype="xfs" --size=7167`），需要改成自适应模式。

生成 ks.cfg 后，先验证语法：

```bash
mkdir -p /mnt/iso/douyu
cp /path/to/ks.cfg /mnt/iso/douyu/

ksvalidator /mnt/iso/douyu/ks.cfg
```

1. ks.cfg 主要作用是代替手动安装的步骤，例如语言、键盘、时区、分区等。
2. `%post ... %end` 表示安装完毕，重启前的收尾工作。
3. `%packages ... %end` 指定安装需要安装的包。
   1. 以 `@^` 开头的表示安装 environment 里所有的 group.
   2. 以 `@` 并开表示单独安装某个 group 里的包。
   3. 直接以包名开头，表示安装某个包。

ks.cfg 模版参考 '7.8\7.8-ks_auto3.cfg'.

### 5 Customize Boot Menu

生成 ks.cfg 后，修改 lagacy BIOS isolinux.cfg 与 EFI grub.cfg, 定制 ISO 安装程序的启动参数，类似于 Grub 启动项。

Legacy BIOS 启动：

```
# isolinux/isolinux.cfg

timeout 60

label linux
  menu label ^KS Install CentOS 7.8
  kernel vmlinuz
  menu default
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS7 inst.ks=cdrom:/douyu/ks.cfg quiet
```

1. 超时改成 60 ms 而非默认的 600 ms.
2. 主要修改地方是，指定 ks.cfg 位置。
3. inst.stage2 是安装完毕后，硬盘启动位置 label

   'hd:LABEL=' 全部改成统一的字符串，可随意设定，但 __MUST be the SAME AS THAT of genisoimage -V BELLOW__.
4. inst.ks 是 Kickstart 加载配制文件的位置。

   本例中，ks.cfg 直接存于 ISO 文件，实际中可放于其它地方，如 U 盘、硬盘分区、网络地址等。
5. 把此 menu 改成默认的，即加 'menu default' 行。同时注释其它 menu 的 'menu default' 行。官方 ISO 的 'menu default' 在 'label check' menu 上。

参考 '7.8\isolinux.cfg'.

EFI grub.cfg 启动：

```
# EFI/BOOT/grub.cfg

set default="0"

menuentry 'KS Install CentOS 7' --class fedora --class gnu-linux --class gnu --class os {
	linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:LABEL=CentOS7 inst.ks=cdrom:/douyu/ks.cfg quiet
	initrdefi /images/pxeboot/initrd.img
}
```

1. KS 启动项放在菜单第 1 位，而 `set default=0` 表示默认启动第 1 个菜单。
2. EFI grub.cfg 部分与 legacy BIOS 部分没有什么区别，主要是增加 ks.cfg 位置。
3. 'hd:LABEL=' 同上。

这里虽然生成了 EFI 的 boot menu, 但是 ks.cfg 里没有创建对应的 ESP 分区，所以是实际无法以 EFI 启动安装。可以先手动用 UEFI 安装一次，再找修改，请参考 [UEFI Kickstart centos.org](https://forums.centos.org/viewtopic.php?t=71030) 和 [Kickstart Example for EFI-based system](https://docs.openvz.org/openvz_installation_using_pxe_guide.webhelp/_kickstart_file_example_for_installing_on_efi_based_systems.html)

### 7 Generate repodata

至此，各项准备工作完毕，开始执行关键命令。首先基于 comps.xml 生成 repodata 目录。repodata 目录包含 ISO 里所有文件的索引，comps.xml 等。

```bash
cd /mnt/iso

createrepo -g douyu/c7-minimal-x86_64-comps.xml .
ll repodata/

diff douyu/c7-minimal-x86_64-comps.xml repodata/*c7-minimal-x86_64-comps.xml
```

createrepo 会复制一份 comps.xml 进 repodata 目录，因此自定义的 comps.xml 不要放进 repodata.

### 8 Generate new ISO image

生成新 ISO 镜像。

```bash
cd /mnt/iso

genisoimage -v -joliet-long -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -R -J -cache-inodes -T -eltorito-alt-boot -e images/efiboot.img -no-emul-boot -V 'CentOS7' -o /home/jim/workspace/CentOS-7-dy.iso /mnt/iso/
```

1. genisoimage 的参数顺序非常重要，不能随变改动。
2. Legacy BIOS 启动参数，对应 isolinux.cfg.

   `-b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -R -J -cache-inodes -T`
3. UEFI 启动参数，对应 EFI grub.cfg.

   `-e images/efiboot.img -no-emul-boot`.
4. Legacy BIOS 和 UEFI 被 `-eltorito-alt-boot` 隔开。

   二者皆附带 `-no-emul-boot`.
5. 最重要的是 `-V` 参数后的标签必须与上面 isolinux.cfg 或 grub.cfg 里的 'hd:LABEL=' 相同。

### 9 Hybrid ISO

上面的制作的 ISO 只能以 CD 模式启动，Hybrid 模式 ISO 可同时用于制作 Usb stick 以硬盘模式启动。

The isohybrid utility modifies a an ISO 9660 image generated with mkisofs, genisoimage, or compatible utilities, to be bootable as a CD-ROM or as a hard disk.

```bash
isohybrid -v /home/jim/workspace/CentOS-7.8-dy.iso
```

### 10 Embed MD5 into ISO

ISO 文件里有一个 section 为空，没有有效内容，因此正好用来填充该 ISO 其它有效内容的 MD5 值。implantisomd5 把 MD5 值注入 ISO 中，会修改 ISO 的内容。对应地，可以用 checkisomd5 命令检验 ISO 文件完整性。

```bash
implantisomd5 /home/jim/workspace/CentOS-7.8-dy.iso

checkisomd5 /home/jim/workspace/CentOS-7.8-dy.iso
```

implantisomd5 只计算原始有效内容的 MD5 值，不是计算整个 ISO 的 MD5 值。同一 ISO 只能注入一次 MD5.

## Install New ISO Image

制作完毕 ISO 后开始安装。

1. 如果安装的初始界面出现 `+[drm:vmw_host_log [vmwgfx]] *ERROR* Failed to send host log message` 日志，是因为 VirtualBox 里 Display 的驱动设置问题，应从 VMSVGA 换成 VBoxVGA, 参考 https://unix.stackexchange.com/q/502540.
2. 安装过程中，可以用 Alt+F1 进主界面，Alt+F2 进 Shell 界面。在 Shell 界面，可以查看 /tmp/{packaging,anaconda}.log 日志，也可以从 /run/install/repo 下找到外部文件。

## Reference

1. https://o-my-chenjian.com/2017/11/20/DIY-A-CentOS7-System/

## Todos

1. https://superuser.com/a/359247 https://superuser.com/a/359247
2. openssl passwd /etc/shadow hash 原理。
3. ks.cfg 里如果包含 'auth', 需要 AppStream 里的 'authselect-compat' package. `BZ#1640697`. 查看 CentOS 8 的 Release Notes.