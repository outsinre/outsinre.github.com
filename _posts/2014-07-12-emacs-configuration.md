---
layout: post
title: Emacs configuration
---

# WINDOWS

## Emacs AucTeX Configuration in Windows:
1. [Emacs AUCTeX and PDF Synchronization on Windows](http://www.barik.net/archive/2012/07/18/154432/)

2. [简单搞定 Emacs + \Latex (三)](http://blog.csdn.net/nangnang/article/details/19234853)

## Emacs 24.3 Display Chinese under Windows

系统为英文版WIN8 RTM X64位。在使用Emacs 24.2打开文件时候，发现中文字体部分显示为方块。通过对编码的设置依然不能解决问题。在网上查找解决方案的时候，发现有人提到通过设置字体能够解决这个问题。于是仔细看了下emacs里的那些个方块，里面的内容其实是中文的编码，由于不能显示对应的文字，Emacs于是原样将字符编码给打印出来。能够显示的中文也很丑，歪歪扭扭。进入控制面板里查看了下字体，发现中文该有的字体都有，只是不同的是，英文版下字体自然也是英文名称。于是根据谷歌的搜索，在.emacs文件的最开始写入如下内容：

`(set-fontset-font "fontset-default" 'gb18030' ("Microsoft YaHei" . "unicode-bmp"))`

Reference: [emacs在win8乱码](http://blog.csdn.net/qianchenglenger/article/details/10950769)

## Emacs Windows Language Environment
Windows 8.1 英文系统，非Unicode语言设置为Simplified Chinese。在Emacs下编辑中文老遇到编码的问题，特别windows ubuntu之间切换各种乱码问题。所以在`init.el`文件中加入如下代码：
{% highlight ruby linenos %}

;; to set UTF-8 language environment
(set-language-environment "UTF-8")
(set-default-coding-systems 'utf-8-unix)
(set-terminal-coding-system 'utf-8-unix)
(set-keyboard-coding-system 'utf-8-unix)
(setq-default buffer-file-coding-system 'utf-8-unix)

;; priority based on reverse order, so the last one is used first
(prefer-coding-system 'chinese-gbk)
(prefer-coding-system 'gb2312)
(prefer-coding-system 'cp936)
(prefer-coding-system 'gb18030)
(prefer-coding-system 'utf-16)
(prefer-coding-system 'utf-8)
(prefer-coding-system 'utf-8-dos)
(prefer-coding-system 'utf-8-unix)

{% endhighlight %}

Refer to [2009-07-09](http://masutaka.net/chalow/2009-07-09-1.html) and [windows下Emacs中文乱码解决办法](http://blog.csdn.net/sanwu2010/article/details/23994977)

# Linux

## Emacs file and buffer Encoding

有时候同一个文件在不同的系统下编码不一样，导致乱码的问题。这时，就要用到emacs的一些M-x命令来**临时**改变编码。

`M-x revert-buffer-with-coding-system`这是改变buffer的编码，并不是真正的改变文件的编码，可以起到临时的查阅作用。

`M-x set-buffer-file-coding-system`这是设置buffer所对应的文件的编码，表示彻底改变编码了。

Reference: [How to switch back text encoding to UTF-8 with emacs?](http://superuser.com/questions/549497/how-to-switch-back-text-encoding-to-utf-8-with-emacs)

## Ubuntu下emacs如何输入中文问题

Ubuntu 14.04系统默是UTF-8编码，没有问题，但是一直无法用Ibus、Sougou、等外置输入法输入中文。今天发现解决方法是**terminal**下：
	`emacs -nw`
这表示terminal模式运行emacs.

但是这个方法有个问题，terminal模式下无法`C-y`粘贴clipboard里面的内容；还有就是`M-f`这个快捷键和terminal冲突。