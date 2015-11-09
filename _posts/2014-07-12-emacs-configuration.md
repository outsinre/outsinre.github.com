---
layout: post
title: Emacs Configuration
---

# ABCs

## frame VS window

Emacs 中两个概念容易混淆，即 frame 和 window。

1. frame 就是我们通常所说的“窗口”，譬如，在系统菜单点击 GNU Emacs，那么 eamcs就会运行，那个就称为frame，其实就是an Emacs process/an Emacs instance，指Emacs的整個窗口。
2. window 是frame 里的小窗口。譬如，`C-x 2` 可以把 frame 分成2个 window，每一个 window 编辑不同的文件（对应一个当前 buffer）。

# Configuration

1. Install (elpa or manually) and configure packages.
2. The packages and init files thereof are synced on GitHub - *.emacs.d*
3. 在 Emacs 启动后，`M-x: list-packages`，会启动 Emacs 自带的插件管理器 elpa (Emacs lisp package archive)。例如找到 auctex，按下 `i` 标记为安装，再按 `x` 开始安装。除了 Emacs 官方的 elpa，还有添加 melpa，org 等 package archive。
4. If edit *init* files manually, pay attention the *lisp* grammar. They can also be modified by *M-x: customize variable*. Remember to *C-x C-s* saving the updates.
5. If Emacs is running at *daemon* server mode, updates to *init* files does **Not** take effect until *daemon* server is re-launched!

    So when configuring Emacs, use *emacs* instead of C/S mode *emacsclient*.

## init origanization

xxx

## Emacs UTF-8

{% highlight ruby linenos %}

;; Disable CJK encoding
(setq utf-translate-cjk-mode nil) 
(set-language-environment "UTF-8")
(set-default-coding-systems 'utf-8)
(set-terminal-coding-system 'utf-8)
(set-keyboard-coding-system 'utf-8)
(setq locale-coding-system 'utf-8)
(set-selection-coding-system 'utf-8)
(setq-default buffer-file-coding-system 'utf-8)
;; priority based on reverse order, so the last one is used first
(prefer-coding-system 'utf-8)

{% endhighlight %}

*reference*:

1. [2009-07-09](http://masutaka.net/chalow/2009-07-09-1.html)
2. [windows下Emacs中文乱码解决办法](http://blog.csdn.net/sanwu2010/article/details/23994977)

## LaTeX AucTeX

Please refer to [auctex emacs](http://jimgray.tk/2015/01/30/auctex/emacs).

## elpy

xxxx

## goto-chg

xxxx

## undo-tree

xxx

## WINDOWS specifics

### Emacs 24.3 Chinese characters on Windows

系统为英文版 Windows RTM X64。在使用 Emacs 24.3 打开文件时候，发现中文字体部分显示为方块。通过对编码的设置依然不能解决问题。在网上查找解决方案的时候，发现有人提到通过设置字体能够解决这个问题。于是仔细看了下 Emacs 里的那些个方块，里面的内容其实是中文的编码，由于不能显示对应的文字，Emacs 于是原样将字符编码给打印出来。能够显示的中文也很丑，歪歪扭扭。进入控制面板里查看了下字体，发现中文该有的字体都有，只是不同的是，英文版下字体自然也是英文名称。于是根据谷歌的搜索，在 init 文件的最开始写入如下内容：

`(set-fontset-font "fontset-default" 'gb18030' ("Microsoft YaHei" . "unicode-bmp"))`

*reference*:

1. [Emacs在win8乱码](http://blog.csdn.net/qianchenglenger/article/details/10950769)

### Default PWD on Windows

`PWD` denotes *print name of current/working directory*. When starting Emacs through Windows shortcut, and using `C-x C-f` to open a file, you find that the default directory is `C\windows\system32`.

To change the default directory by either of the two:

1. Either edit the Emacs shortcut, in the `start in` field, fill in your default working directory.
2. Or add:

    ```lisp
    (setq default-directory "E:/workspace")
    ```
