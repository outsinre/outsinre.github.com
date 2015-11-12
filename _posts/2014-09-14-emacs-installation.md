---
layout: post
title: Emacs installation
---

# Windows 安装

Refer to [从零开始——Emacs 安装配置使用教程 2015](http://www.jianshu.com/p/b4cf683c25f3).

# Linux 安装

下面关于 Linux 的部分主要以 Gentoo 为主。

## 输入法

1. 如果是 Gentoo，一定要安装 font-adobe-75dpi 和 font-adobe-100dpi 字体。
2. `LC_CTYPE` 要设置称 `zh_CN.UTF-8`.
3. 安装 Fcitx 输入法。

参考 [Gentoo Installation](http://jimgray.tk/2015/03/25/gentoo-installation/).

## Emacs 启动太慢

> Emacs = Emacs Makes A Computer Slow.

Emacs 由于要加载好多脚本，特别是 .emacs 或 init.el 里的内容很多时，太慢，是 Emacs 一大诟病。不过我们可以利用 Emacs 的 C/S 模式，脚本的加载让 server 来完成。然后再用客户端 emacsclient 连接 Emacs server，方法如下：

### Emacs 23 之前的版本：

可以在启动 Emacs 的时候，顺便选择开启 server-start。

1. 一种方法是在 init.el 里面设定加入 `(server-start)` 或 `(server-mode)`。
2. 另一种是直接在 Emacs 的 mini-buffer 输入命令`M-x: server-start/server-mode`。

Emacs 23 之前的那种方法有个缺点，即所开启的 server mode 只属于当前的 Emacs frame，如果这个 frame 关闭了，那么 server 就随之关闭。再运行 Eamcs 时，又要重新加载 server mode。所以为了能够利用这个 C/S 模式，开启 server mode 的那个 frame/instance/session 不能关闭！ 只能在另一个 terminal 启动 emacsclient 来链接复用这个 server。
### Emacs 23 开始的版本

幸好 Emacs 23 引入 `emacs --daemon`，那么 daemon 与 server 有细微区别，可以常驻系统（内存）中，与某个 Emacs frame/instance 无关。关闭当前的 Emacs frame，daemon 服务依然存在，这时关闭的仅仅是 client 端。

下面开始不区分 server 和 daemon 两个词汇。看看 client 端如何链接 daemon/server。

### eamcsclient

运行客户端`emacsclient`，连接 daemon/server，对文件进行处理。eamcsclient有几个关键参数：

1. -t

    -t 表示在 Terminal 中打开字符界面的 frame。
2. -c

    -c表示创建 create 一个 X11 界面的 frame，也就是通常所说的 GUI。
3. -a

    -a, --alternate-editor, 表示如果 server mode 没有开启，那么选择一个替代编辑器，譬如 vim/gedit 等。

    通常我们设置成空：""。如 `-a ""` 或 `--alternate-editor=""`。设置成空表示，如果 server mode 没有加载（通常是开机第一次运行），由于替代编辑器为空，emacsclient 则会先加载 server。

    但是如果每次运行 emacsclient 命令时，都带上 `-a` 很麻烦。更好的办法是设置系统变量 `ALTERNATE_EDITOR=""`,下面会涉及到。

### 终端下运行 emacsclient

```
_$_ emacs --daemon
_$_ emacsclient -t -a "" [file names]
_$_ emacsclient -c -a "" [file names]
```
这里 file names 是可省略的参数。如果每次都这样运行，输入的命令太长来，作如下改进：

1. emacs --daemon 可以加入 default runlevel，开机启动。不过我不认为这样是个好主意，毕竟不是每次开机都铁定运行 Emacs，而且还影响开机速度。我们按需启动 Emacs server。
2. 给这几个命令取别名，或创建脚本，简化命令的长度。脚本文件放在 `/usr/local/bin` 下：

    ```
    # /usr/local/bin/ect
    #! /bin/bash
    emacsclient -t -a "" "$@"
    ```

    ```
    # /usr/local/bin/ecx
    #! /bin/bash
    emacsclient -c -a "" "$@"
    ```
    `"$@"` 表示接受命令行的所有参数，主要就是要编辑的文件名列表。默认情况下，`/usr/local/bin` 已经加入到了 `PATH` 中：

    >$ type ecx ect
    
    >$ ect/ecx [file names]
    
3. 为了省略脚本中 `-a` 参数,在 */etc/env.d/* 下创建文件 *99local*，用于存放 system-wide environment variable，内容如下：

        export ALTERNATE_EDITOR=""
4. 修改默认编辑器为 ect。在 */etc/env.d/99editor* 添加：

        EDITOR=/usr/local/bin/ect

    这种直接编辑 *99editor* 的方法不方便！在 root 下运行:

        # eselect editor list
        # eselect editor set "/usr/local/bin/ect"

    如果是在普通用户下运行 eselect 则设置只对当前用户生效。
5. \# env-update && source /etc/profile

    更新系统环境变量，让上面两步生效！
6. 上面的步骤是为 system-wide 设置的，也可以单独为用户自己设置 private owned scripts：
    1. 脚本放在 *~/bin* 下面。
    2. *~/bin* 加入到当前用户的 `PATH` 中。
    3. `ALTERNATE_EDITOR` 和 `EDITOR` 相应设置在 `.bashrc` 中。
    
### 桌面下运行 emacsclient

上面的步骤是针对命令行来的,对于 Gentoo 下的 XFCE 桌面环境,该如何办启动 emacsclient?

修改 */usr/share/applications/emacsclient.desktop* 文件. Gentoo 安装 Emacs 时,默认创建了 emacsclient.desktop 文件,但是在 XFCE 菜单里没有出现. 我们参考 emacs.desktop 文件，需要修改 `NoDisplay`，`Exec` 和 `Categories` 参数:

>NoDisplay=false
>
>Exec=/usr/bin/emacsclient -c -a "" %F
>
>Categories=Development;TextEditor;

如果在前面的 *99local* 里设置了 `ALTERNATE_EDITOR`，这里面的 `-a` 参数也可以省略掉。一个更好的编辑 emacsclient.desktop 文件办法是直接：

    # pushd /usr/share/applications
    # mv emacs.desktop emacsclient.desktop
然后依照上面修改里面的 `Exec` 那一行即可。这里，我们去掉了默认的 Emacs GUI 启动菜单。

其中有个 `%F` 参数，具体意义参考 [Desktop Entry Specification](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html)。注意这个参数不能放在 Terminal 启动脚本里，它只属于 GUI X 菜单。

现在可以直接在系统菜单找到 emacsclient 菜单, 而且右键可以正常使用 *Open With "emacsclient"*。由于 Emacs 的启动速度问题解决了，mousepad 就基本推出历史舞台了.
