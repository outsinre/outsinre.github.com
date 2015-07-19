---
layout: post
title: Emacs configuration
---
# frame VS window
Emacs中两个概念容易混淆，即frame和window。

frame就是我们通常所说的“窗口”，譬如，在系统菜单点击Emacs，那么eamcs就会运行，那个就称为frame，其实就是an emacs process/an emacs instance。

window是frame里的小窗口。譬如，可以把frame分成多个window，每一个window编辑不同的文件（buffer）。

# WINDOWS

## Emacs AucTeX Configuration in Windows:
1. [Emacs AUCTeX and PDF Synchronization on Windows](http://www.barik.net/archive/2012/07/18/154432/)

2. [简单搞定 Emacs + \Latex (三)](http://blog.csdn.net/nangnang/article/details/19234853)

3. Pay attention to the last line `(setq TeX-source-correlate-start-server t)`. This line is for inverse search. Details refer to [31.16. Using Emacs as a Server](http://www.nongnu.org/emacsdoc-fr/manuel/emacs-server.html), [4.2.2 Forward and Inverse Search](https://www.gnu.org/software/auctex/manual/auctex/I_002fO-Correlation.html), and [Setup SyncTeX with Emacs](http://tex.stackexchange.com/questions/29813/setup-synctex-with-emacs). Actually, I disable this function for security reason. When necessary (i.e. C-c C-v for preview), emacs will remind you of this function.
## Emacs 24.3 Display Chinese under Windows

4. Add line `(setq-default TeX-engine 'xetex)` to use `xetex` as the default engine.

系统为英文版WIN8 RTM X64位。在使用Emacs 24.2打开文件时候，发现中文字体部分显示为方块。通过对编码的设置依然不能解决问题。在网上查找解决方案的时候，发现有人提到通过设置字体能够解决这个问题。于是仔细看了下emacs里的那些个方块，里面的内容其实是中文的编码，由于不能显示对应的文字，Emacs于是原样将字符编码给打印出来。能够显示的中文也很丑，歪歪扭扭。进入控制面板里查看了下字体，发现中文该有的字体都有，只是不同的是，英文版下字体自然也是英文名称。于是根据谷歌的搜索，在.emacs文件的最开始写入如下内容：

`(set-fontset-font "fontset-default" 'gb18030' ("Microsoft YaHei" . "unicode-bmp"))`

Reference: [emacs在win8乱码](http://blog.csdn.net/qianchenglenger/article/details/10950769)

## Emacs Windows Language Environment
Windows 8.1 英文系统，非Unicode语言设置为Simplified Chinese。在Emacs下编辑中文老遇到编码的问题，特别windows ubuntu之间切换各种乱码问题。所以在`init.el`文件中加入如下代码：
{% highlight ruby linenos %}

;; to set UTF-8 language environment
(set-language-environment "UTF-8")
(set-default-coding-systems 'utf-8)
(set-terminal-coding-system 'utf-8)
(set-keyboard-coding-system 'utf-8)
(setq-default buffer-file-coding-system 'utf-8)

;; priority based on reverse order, so the last one is used first
(prefer-coding-system 'gb18030)
(prefer-coding-system 'utf-8)

{% endhighlight %}

Refer to [2009-07-09](http://masutaka.net/chalow/2009-07-09-1.html) and [windows下Emacs中文乱码解决办法](http://blog.csdn.net/sanwu2010/article/details/23994977)

## Default CWD on Windows

`CWD` denotes *current working directory*.

When starting Emacs through Windows shortcut, and using `C-x C-f` to open a file, you find that the default directory is `C\windows\system32`.

To change the default directory is:

1. Either edit the Emacs shortcut, in the `start in` field, fill in your default working directory.
2. Or add a line in your `init.el` file like `(setq default-directory "/path/to/documents/directory/")`.

The firt method is better, since you should keep your `init.el` file consistent for future use.

# Linux

## Emacs file and buffer Encoding

有时候同一个文件在不同的系统下编码不一样，导致乱码的问题。这时，就要用到emacs的一些M-x命令来**临时**改变编码。

`M-x revert-buffer-with-coding-system`这是改变buffer的编码，并不是真正的改变文件的编码，可以起到临时的查阅作用。

`M-x set-buffer-file-coding-system`这是设置buffer所对应的文件的编码，表示彻底改变编码了。

Reference: [How to switch back text encoding to UTF-8 with emacs?](http://superuser.com/questions/549497/how-to-switch-back-text-encoding-to-utf-8-with-emacs)

## Ubuntu下emacs如何输入中文问题

Ubuntu 14.04系统默是UTF-8编码，没有问题，但是一直无法用Ibus、Sougou、等外置输入法输入中文。今天发现解决方法是**terminal**下：
	`emacs -nw`
这表示terminal模式运行emacs. 但是这个方法有个问题，terminal模式下无法`C-y`粘贴clipboard里面的内容，要用terminal自己的粘贴快捷键；还有就是`M`开头的快捷键经常和terminal冲突。

最后发现问题了，要把locale中的`LC_CTYPE`设置成中文:

`sudo update-locale LC_CTYPE=zh_CN.UTF-8`,然后reboot即可。

# Gentoo下emacs

中文输入的问题，可以参考Gentoo Installation.

另外注意一定要安装font-adobe-75dpi和font-adobe-100dpi字体。

在~/.bashrc中加入下面行，把emacs设置为默认编辑器。

>EDITOR=/usr/bin/emacs

## Emacs启动太慢

>Emacs = Emacs Makes A Computer Slow.

emacs 由于要加载好多脚本，特别是.emacs 或 init.el里的内容很多时，太慢，是emacs一大诟病。不过我们可以利用emacs的C/S模式，先让emacs运行在server mode，脚本的加载让server来完成。然后再用客户端emacsclient连接server。

### emacs 23之前的版本：

可以在启动emacs的时候，顺便选择开启server-start。一种方法是在init.el里面设定加入：`(server-start)` or `(server-mode)`。另一种是直接在emacs里面输入命令：`M-x server-start/server-mode`。

### emacs 23开始的版本

emacs 23之前的那种方法有个缺点：所开启的server mode只属于当期的emacs frame，如果这个emacs关闭来，那么server就关闭来，再开启eamcs时，又要重新加载server mode。

不过eamcs 23引入`emacs --daemon`，那么server mode可以常驻系统中，与某个emacs frame无关。关闭当前的emacs frame，server服务并没有停止。

### eamcsclient

解决来server mode问题之后，运行客户端`emacsclient`，连接server，对文件进行处理。eamcsclient有几个关键参数：

1. -t

    -t表示在terminal中打开字符界面的frame
2. -c

    -c表示创建create一个X11界面的frame，也就是通常所说的GUI
3. -a

    -a, --alternate-editor,表示如果server mode没有开启，那么选择一个替代编辑器，譬如vim，gedit等。

    通常我们设置成空：""。如 `-a ""` 或 `--alternate-editor=""`。设置成空表示，如果server mode没有加载，那么就emacsclient会先加载它。

    但是如果每次运行emacsclient命令时，都带上`-a`很麻烦。更好的办法是设置系统变量`ALTERNATE_EDITOR=""`,下面会涉及到。

### 终端下运行

```
_$_ emacs --daemon
_$_ emacsclient -t -a "" [file name]
_$_ emacsclient -c -a "" [file name]
```

如果每次都这样运行，输入的命令太长来，作如下改进：

1. emacs --daemon可以加入default runlevel，开机启动。不过我不认为这样是个好主意，毕竟不是每次开机都铁定运行emacs，而且还影响开机速度。我们要的是按需启动emacs server。
2. 给这几个命令**取别名**、或**创建脚本**，简化命令的长度。脚本文件放在`/usr/local/bin`下：

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

    `"$@"`，表示接受命令行的所有参数,主要就是要编辑的文件名。
    
    默认情况下，`/usr/local/bin`加入到了`PATH`中，测试：
    >_$_ which ecx/ect
    
    >_$_ ect/ecx [path/to/filename]
    
3. 为了省略脚本中`-a`参数,在`/etc/env.d/`下创建文件99local，用于存放system-wide environment variable，内容如下：

    >export ALTERNATE_EDITOR=""
4. 修改默认编辑器为ect。在`/etc/env.d/99local`添加：

    >EDITOR=/usr/local/bin/ect
5. _#_ env-update && source /etc/profile

    更新系统环境变量。
6. 上面的步骤是为system-wide设置的，也可以单独为用户自己设置private owned scripts：
    1. 脚本放在`~/bin`下面
    2. `~/bin`加入到用户的`PATH`中
    3. `ALTERNATE_EDITOR`和`EDITOR`也必须设置。
    
### 桌面下运行

上面的步骤是针对命令行来的,对于Gentoo下的XFCE桌面环境,该如何办启动emacsclient?

修改/usr/share/applications/emacsclient.desktop文件. Gentoo安装emacs时,默认创建了emacsclient.desktop文件,但是在XFCE菜单里没有出现.

我们参考emacs.desktop文件,需要修改: NoDisplay和Exec. 然后添加Categories参数:

>NoDisplay=false
>
>Exec=/usr/bin/emacsclient -c -a "" %F
>
>Categories=Development;TextEditor;

现在可以直接在系统菜单找到Emacsclient菜单, 而且右键可以正常使用`Open With "Emacsclient"`。

如果在前面的99local里设置了`ALTERNATE_EDITOR`，这里面的`-a`参数也可以省略掉，

### mousepad就显得有些多余了.

由于emacs的启动速度问题解决了,mousepad就基本推出历史舞台了.