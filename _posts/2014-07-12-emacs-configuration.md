---
layout: post
title: Emacs configuration
---
# init.el
Most of the time, configuring *Emacs* will update *init.el* file ultimately no matter through editing it *manually* or through *command line* in mini-buffer.

1. If editting it manually, pay attention the *lisp* grammar.
2. If editing by command line *M-x*, use *C-x C-s* to keep the updates to *init.el*.
3. If *Emacs* is running at *daemon* server mode, updates to *init.el* does **Not** take effect until *daemon* server is re-launched!

    **ATTENTION**: I wasted a lot of time on configuring *AucTex* below since the updates does not take effect until I re-launched *Emacs daemon*.

# frame VS window
Emacs中两个概念容易混淆，即frame和window。

frame就是我们通常所说的“窗口”，譬如，在系统菜单点击Emacs，那么eamcs就会运行，那个就称为frame，其实就是an emacs process/an emacs instance。

window是frame里的小窗口。譬如，可以把frame分成多个window，每一个window编辑不同的文件（buffer）。

# WINDOWS
The Windows *init.el* has a backup at *GitHub*.
## Emacs AucTeX Configuration in Windows:
1. [Emacs AUCTeX and PDF Synchronization on Windows](http://www.barik.net/archive/2012/07/18/154432/)

2. [简单搞定 Emacs + \Latex (三)](http://blog.csdn.net/nangnang/article/details/19234853)

3. Pay attention to the last line `(setq TeX-source-correlate-start-server t)`. This line is for inverse search. Details refer to [31.16. Using Emacs as a Server](http://www.nongnu.org/emacsdoc-fr/manuel/emacs-server.html), [4.2.2 Forward and Inverse Search](https://www.gnu.org/software/auctex/manual/auctex/I_002fO-Correlation.html), and [Setup SyncTeX with Emacs](http://tex.stackexchange.com/questions/29813/setup-synctex-with-emacs). Actually, I disable this function for security reason. When necessary (i.e. C-c C-v for preview), emacs will remind you of this function.
4. Add line `(setq-default TeX-engine 'xetex)` to use `xetex` as the default engine.

## Emacs 24.3 Display Chinese under Windows
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

## Ubuntu下emacs如何输入中文问题

Ubuntu 14.04系统默是UTF-8编码，没有问题，但是一直无法用Ibus、Sougou、等外置输入法输入中文。今天发现解决方法是**terminal**下：
	`emacs -nw`
这表示terminal模式运行emacs. 但是这个方法有个问题，terminal模式下无法`C-y`粘贴clipboard里面的内容，要用terminal自己的粘贴快捷键；还有就是`M`开头的快捷键经常和terminal冲突。

最后发现问题了，要把locale中的`LC_CTYPE`设置成中文:

`sudo update-locale LC_CTYPE=zh_CN.UTF-8`,然后reboot即可。

# Gentoo下emacs

中文输入的问题，可以参考[Gentoo Installation](http://www.fangxiang.tk/2015/03/25/gentoo-installation/).

另外注意一定要安装font-adobe-75dpi和font-adobe-100dpi字体。

## Emacs启动太慢

>Emacs = Emacs Makes A Computer Slow.

emacs 由于要加载好多脚本，特别是.emacs 或 init.el里的内容很多时，太慢，是emacs一大诟病。不过我们可以利用emacs的C/S模式，先让emacs运行在server mode，脚本的加载让server来完成。然后再用客户端emacsclient连接server。

### emacs 23之前的版本：

可以在启动emacs的时候，顺便选择开启server-start。一种方法是在init.el里面设定加入：`(server-start)` or `(server-mode)`。另一种是直接在emacs里面输入命令：`M-x server-start/server-mode`。

### emacs 23开始的版本

emacs 23之前的那种方法有个缺点：所开启的server mode只属于当前的emacs frame，如果这个emacs关闭来，那么server就关闭来，再开启eamcs时，又要重新加载server mode。

不过eamcs 23引入`emacs --daemon`，那么server mode可以常驻系统中，与某个emacs frame无关。关闭当前的emacs frame，server服务并没有停止。

### eamcsclient

解决来server mode问题之后，运行客户端`emacsclient`，连接server，对文件进行处理。eamcsclient有几个关键参数：

1. -t

    -t表示在terminal中打开字符界面的frame
2. -c

    -c表示创建create一个X11界面的frame，也就是通常所说的GUI
3. -a

    -a, --alternate-editor,表示如果server mode没有开启，那么选择一个替代编辑器，譬如vim，gedit等。

    通常我们设置成空：""。如 `-a ""` 或 `--alternate-editor=""`。设置成空表示，如果server mode没有加载，那么就emacsclient会先加载它，这通常是开机后第一次运行emacs时的需要。

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
    
    默认情况下，`/usr/local/bin`已经加入到了`PATH`中，测试：
    >_$_ type ecx ect
    
    >_$_ ect/ecx [path/to/filename]
    
3. 为了省略脚本中`-a`参数,在`/etc/env.d/`下创建文件99local，用于存放system-wide environment variable，内容如下：

        export ALTERNATE_EDITOR=""
4. 修改默认编辑器为ect。在`/etc/env.d/99editor`添加：

        EDITOR=/usr/local/bin/ect

    这种直接编辑*99editor*的方法不方便！在*root*下运行:

        # eselect editor list
        # eselect editor set "/usr/local/bin/ect"

    如果是在普通用户下运行*eselect*则设置只对普通用户生效。
5. _#_ env-update && source /etc/profile

    更新系统环境变量，让上面两步生效！
6. 上面的步骤是为system-wide设置的，也可以单独为用户自己设置private owned scripts：
    1. 脚本放在`~/bin`下面
    2. `~/bin`加入到用户的`PATH`中
    3. `ALTERNATE_EDITOR`和`EDITOR`相应设置在*.bashrc*中。
    
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

如果在前面的99local里设置了`ALTERNATE_EDITOR`，这里面的`-a`参数也可以省略掉。

一个更好的办法是是直接

    # pushd /usr/share/applications
    # mv emacs.desktop emacsclient.desktop
然后依照上面修改里面的*Exec*那一行即可。这里，我们去掉了默认的emacs X启动菜单。

其中有个*%F*参数，具体意义参考[Desktop Entry Specification](http://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html).注意这个参数不能放在前面的启动脚本里，它只属于X菜单。

mousepad就显得有些多余了.由于emacs的启动速度问题解决了,mousepad就基本推出历史舞台了.

## 安装Auctex
在emacs启动后，M-x: package-list-packages，会启动emacs自带的插件管理器。找到*auctex*，按下*i*标记为安装，再按*x*，开始安装。

具体有哪些基本按键，参考[How to Install Packages Using ELPA, MELPA, Marmalade](http://ergoemacs.org/emacs/emacs_package_system.html)页面。

### 配置Auctex

> The lastest *init.el* can be accessed from GitHub.

Refer to the beginnig of this post, while updating *init.el*, make sure to re-launch *daemon*, otherwise the updates would not take effect and you would thought your updates error.

The contents of "*${HOME}/.emacs.d/init.el*":

```lisp
(setq TeX-auto-save t)
(setq TeX-parse-self t)
(setq-default TeX-master nil)

(setq-default TeX-PDF-mode t)

(setq-default TeX-engine 'xetex)
```
The last line is to set `xetex` as the default *engine*. The last but second set the default output format as PDF.

Up to know, we can compile Chinese *LaTeX* files automatically.

Refer to [emacs-as-the-ultimate-latex-editor](http://piotrkazmierczak.com/2010/emacs-as-the-ultimate-latex-editor/).

### Previewer
My Gentoo system use *MuPDF* PDF viewer. How to make it the default previewer for *AucTeX*? Basically, we need to update two variable of *AucTeX*, which could be achieved by editing *init.el* manually or by command line *M-x*.

1. TeX-view-program-list

    This variable is a list of viewers for different output formats. *Emacs* has default builtin viewers for *.dvi*, *.pdf*, *.ps* etc. For example, the default viewer for *.pdf* is *Evince*. Here I just want to add *MuPDF* to the list for *.pdf* output. *MuPDF* does *NOT* support *forward* and *backward* search while *Evince*, *okular* etc do.

    If by manual edit *init.el*:

    ```lisp
    (setq TeX-view-program-list '(("MuPDF" "mupdf %s.pdf")))
    ```
    Use *C-c C-v* to preview PDF output.

    If use command line in mini-buffer, we first need to turn on *M-x tex-mode*, or open a *.tex* file.
    
    ```
    M-x customize-variable:  will open the builtin editor;
    TeX-view-program-list: open variable for edit
    INS: click to add a viewer to the list except the builtin ones
    Name: MuPDF
    Command: mupdf %s.pdf
    C-x C-s
    ```
    Open the *init.el* file, you will find *MuPDF* is added to the list.
2. TeX-view-program-selection

    This variable defines the exact viewer to choose from *TeX-view-program-list* when viewing output format. Similarly, edit by manual or by command line.

    Manually:

    ```lisp
    (setq TeX-view-program-selection '((output-pdf "MuPDF")))
    ```

    Command line:

    ```
    M-x customize-variable:  will open the builtin editor;
    TeX-view-program-selection: open for edit
    INS DEL Choice: Value Menu Single predicate: Value Menu output-p
df
                   Viewer: Value Menu MuPDF
    C-x C-s
    ```
    Just locate *Viewer* for *output-pdf*. Click on **Value Menu**, you could get a drop list menu, choose *MuPDF*. The final *init.el* contents:

    ```lisp
    (custom-set-variables
     ;; custom-set-variables was added by Custom.
     ;; If you edit it by hand, you could mess it up, so be careful.
     ;; Your init file should contain only one such instance.
     ;; If there is more than one, they won't work right.
     '(TeX-view-program-list (quote (("MuPDF" ("mupdf %s.pdf") ""))))
     '(TeX-view-program-selection
       (quote
        (((output-dvi style-pstricks)
          "dvips and gv")
         (output-dvi "xdvi")
         (output-pdf "MuPDF")
         (output-html "xdg-open")))))
    (custom-set-faces
     ;; custom-set-faces was added by Custom.
     ;; If you edit it by hand, you could mess it up, so be careful.
     ;; Your init file should contain only one such instance.
     ;; If there is more than one, they won't work right.
     )
    ```
3. RefTeX

   ```lisp
   ; Turn on RefTeX for AUCTeX, http://www.gnu.org/s/auctex/manual/reftex/reftex_5.html
   (add-hook 'LaTeX-mode-hook 'turn-on-reftex)
   ; Make RefTeX interact with AUCTeX, http://www.gnu.org/s/auctex/manual/reftex/AUCTeX_002dRefTeX-Interface.html
   (setq reftex-plug-into-AUCTeX t)
   ```

### Writing LaTeX
For a bignner, refer to [LaTeX](http://www.fangxiang.tk/2015/02/05/LaTeX/).
