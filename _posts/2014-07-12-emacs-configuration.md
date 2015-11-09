---
layout: post
title: Emacs configuration
---

# init.el

Most of the time, configuring *Emacs* will update *init.el* file ultimately no matter through editing it *manually* or through *customize variable* in mini-buffer.

1. If editting it manually, pay attention the *lisp* grammar.
2. If editing by command line *M-x*, use *C-x C-s* to keep the updates to *init.el*.
3. If *Emacs* is running at *daemon* server mode, updates to *init.el* does **Not** take effect until *daemon* server is re-launched!

    **ATTENTION**: I wasted a lot of time on configuring AucTex below since the updates did not take effect until I re-launched *Emacs daemon*.

# frame VS window

Emacs 中两个概念容易混淆，即 frame 和 window。

1. frame 就是我们通常所说的“窗口”，譬如，在系统菜单点击 GNU Emacs，那么 eamcs就会运行，那个就称为frame，其实就是an Emacs process/an Emacs instance，指Emacs的整個窗口。
2. window 是frame 里的小窗口。譬如，`C-x 2` 可以把 frame 分成2个 window，每一个 window 编辑不同的文件（对应一个当前 buffer）。

# WINDOWS

The Windows *init.el* has a backup at GitHub.

## Emacs AucTeX Configuration in Windows:

1. [Emacs AUCTeX and PDF Synchronization on Windows](http://www.barik.net/archive/2012/07/18/154432/)
2. [简单搞定 Emacs + \Latex (三)](http://blog.csdn.net/nangnang/article/details/19234853)
3. Pay attention to the last line `(setq TeX-source-correlate-start-server t)`. This line is for inverse search. Actually, I disable this function for security reason. When necessary (i.e. C-c C-v for preview), Emacs will remind you of this function.
4. Add line `(setq-default TeX-engine 'xetex)` to use `xetex` as the default engine.

*reference*:

1. [31.16. Using Emacs as a Server](http://www.nongnu.org/emacsdoc-fr/manuel/emacs-server.html)
2. [4.2.2 Forward and Inverse Search](https://www.gnu.org/software/auctex/manual/auctex/I_002fO-Correlation.html)
3. [Setup SyncTeX with Emacs](http://tex.stackexchange.com/questions/29813/setup-synctex-with-emacs).

## Emacs 24.3 Display Chinese under Windows

系统为英文版 Windows RTM X64。在使用Emacs 24.2打开文件时候，发现中文字体部分显示为方块。通过对编码的设置依然不能解决问题。在网上查找解决方案的时候，发现有人提到通过设置字体能够解决这个问题。于是仔细看了下Emacs里的那些个方块，里面的内容其实是中文的编码，由于不能显示对应的文字，Emacs于是原样将字符编码给打印出来。能够显示的中文也很丑，歪歪扭扭。进入控制面板里查看了下字体，发现中文该有的字体都有，只是不同的是，英文版下字体自然也是英文名称。于是根据谷歌的搜索，在.emacs文件的最开始写入如下内容：

`(set-fontset-font "fontset-default" 'gb18030' ("Microsoft YaHei" . "unicode-bmp"))`

*reference*:

1. [Emacs在win8乱码](http://blog.csdn.net/qianchenglenger/article/details/10950769)

## Emacs Windows Language Environment

Windows 8.1 英文系统，非Unicode语言设置为Simplified Chinese。在Emacs下编辑中文老遇到编码的问题，特别windows ubuntu之间切换各种乱码问题。所以在`init.el`文件中加入如下代码：

{% highlight ruby linenos %}

;; to set UTF-8 language environment
(set-language-environment "UTF-8")
(set-default-coding-systems 'UTF-8)
(set-terminal-coding-system 'UTF-8)
(set-keyboard-coding-system 'UTF-8)
(setq-default buffer-file-coding-system 'UTF-8)

;; priority based on reverse order, so the last one is used first
(prefer-coding-system 'gb18030)
(prefer-coding-system 'UTF-8)

{% endhighlight %}

*reference*:

1. [2009-07-09](http://masutaka.net/chalow/2009-07-09-1.html) and [windows下Emacs中文乱码解决办法](http://blog.csdn.net/sanwu2010/article/details/23994977)

## Default PWD on Windows

`PWD` denotes *print name of current/working directory*. When starting Emacs through Windows shortcut, and using `C-x C-f` to open a file, you find that the default directory is `C\windows\system32`.

To change the default directory by either of the two:

1. Either edit the Emacs shortcut, in the `start in` field, fill in your default working directory.
2. Add a line in your `init.el`:

    ```lisp
    (setq default-directory "/path/to/documents/directory/")
    ```

# Linux

下面关于 Linux 的部分主要以 Gentoo 为主。
## 输入法

1. 如果是 Gentoo，一定要安装 font-adobe-75dpi 和 font-adobe-100dpi 字体。
2. `LC_CTYPE` 要设置称 `zh_CN.UTF-8`.
3. 安装 Fcitx 输入法。

参考 [Gentoo Installation](http://www.fangxiang.tk/2015/03/25/gentoo-installation/).

## Emacs 启动太慢

> Emacs = Emacs Makes A Computer Slow.

Emacs 由于要加载好多脚本，特别是 .emacs 或 init.el 里的内容很多时，太慢，是 Emacs 一大诟病。不过我们可以利用 Emacs 的 C/S 模式，先让 Emacs 运行在 server mode，脚本的加载让 server 来完成。然后再用客户端 emacsclient 连接 Emacs server，方法如下：

### Emacs 23 之前的版本：

可以在启动 Emacs 的时候，顺便选择开启 server-start。

1. 一种方法是在 init.el 里面设定加入 `(server-start)` 或 `(server-mode)`。
2. 另一种是直接在 Emacs 的 mini-buffer 输入命令`M-x server-start/server-mode`。

### Emacs 23 开始的版本

Emacs 23 之前的那种方法有个缺点，即所开启的 server mode 只属于当前的 Emacs frame，如果这个 frame 关闭了，那么 server 就随之关闭。再运行 Eamcs 时，又要重新加载 server mode。

幸好 Emacs 23 引入 `emacs --daemon`，那么 server mode 可以常驻系统（内存）中，与某个 Emacs frame/instance 无关。关闭当前的 Emacs frame，server 服务依然存在，这时关闭的仅仅是 client 端。

解决了 server mode 问题后，下面看看 client 端如何链接 server mode。

### eamcsclient

解决来server mode问题之后，运行客户端`emacsclient`，连接server，对文件进行处理。eamcsclient有几个关键参数：

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

# Plugins

1. The plugins and init.el thereof are located on GitHub.
2. As pointed out in the begining of the post, once *init lisp* files (i.e. *init.el*) got updated, make sure to re-launch *daemon*, otherwise the updates would not take effect, misdirecting you to modify your *init lisp* files.
3. 在 Emacs 启动后，`M-x: package-list-packages`，会启动 Emacs 自带的插件管理器 elpa (Emacs local package)。找到 auctex，按下 `i` 标记为安装，再按 `x` 开始安装。参考 [How to Install Packages Using ELPA, MELPA, Marmalade](http://ergoemacs.org/emacs/emacs_package_system.html)。

## LaTeX AucTeX

Let Emacs support LaTeX writing and compiling. To *${HOME}/.emacs.d/init.el* file, append:

```lisp
(setq TeX-auto-save t)
(setq TeX-parse-self t)
(setq-default TeX-master nil)

(setq-default TeX-PDF-mode t)

(setq-default TeX-engine 'xetex)
```
The last line is to set `xetex` as the default *TeX engine*. The last but second set the default output format as PDF instead of DVI. Now Emacs is capable of compiling Chinese LaTeX sources by `C-c C-c` command.

Refer to [emacs-as-the-ultimate-latex-editor](http://piotrkazmierczak.com/2010/emacs-as-the-ultimate-latex-editor/).

### LaTeX Previewer

My Gentoo system use MuPDF PDF viewer. How to make it the default previewer of AucTeX? Basically, we need to update two variables, which can be achieved by editing *init.el* manually or by command *M-x customize-variable* in frame mini-buffer.

1. TeX-view-program-list

    This variable is a list of viewers for different output formats. Emacs has default builtin viewers for `.dvi`, `.pdf`, `.ps` etc. For example, the default viewer for `.pdf` is Evince. Here I just want to add MuPDF to the list for `.pdf` output. MuPDF does **NOT** support *forward* and *backward* search while Evince, Ookular etc. do.

    If manually edit *init.el*:

    ```lisp
    (setq TeX-view-program-list '(("MuPDF" "mupdf %s.pdf")))
    ```
    Use *C-c C-v* to preview PDF output.

    If command line in mini-buffer, first turn on `M-x tex-mode` or open a `.tex` file.
    
    ```
    M-x customize-variable:  will open the builtin variable editor;
    TeX-view-program-list: open variable for edit
    INS: click to add a viewer to the list except the builtin ones
    Name: MuPDF
    Command: mupdf %s.pdf
    C-x C-s
    ```
    Open the *init.el* file, you will find MuPDF is added to the list.
2. TeX-view-program-selection

    This variable defines the exact viewer to choose from *TeX-view-program-list* when viewing a specific TeX output format. Similarly, edit by manual or by command line.

    If manually:

    ```lisp
    (setq TeX-view-program-selection '((output-pdf "MuPDF")))
    ```

    If command line in mini-buffer:

    ```
    M-x customize-variable:  will open the builtin variable editor;
    TeX-view-program-selection: open for edit
    INS DEL Choice: Value Menu Single predicate: Value Menu output-pdf
                   Viewer: Value Menu MuPDF
    C-x C-s
    ```
    Just locate *Viewer* for *output-pdf*. Click on **Value Menu**, you could get a drop list menu, choose *MuPDF*. The updated *init.el*:

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
### RefTeX and AucTeX

 Since version 24.3 of Emacs, RefTeX is developed exclusively as part of Emacs. So if you want to get the latest version of RefTeX, you should get the latest version of Emacs. (Now outdated versions of the standalone distribution are kept for archeologic purposes on the GNU FTP server.) 

   ```lisp
   ; Turn on RefTeX for AUCTeX, http://www.gnu.org/s/auctex/manual/reftex/reftex_5.html
   (add-hook 'LaTeX-mode-hook 'turn-on-reftex)
   ; Make RefTeX interact with AUCTeX, http://www.gnu.org/s/auctex/manual/reftex/AUCTeX_002dRefTeX-Interface.html
   (setq reftex-plug-into-AUCTeX t)
   ```

### Writing LaTeX

For a bignner, refer to [LaTeX](http://www.fangxiang.tk/2015/02/05/LaTeX/).
