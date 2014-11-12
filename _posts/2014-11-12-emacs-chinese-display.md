---
layout: post
title: Emacs 24.3 Display Chinese under Windows
---

系统为英文版WIN8 RTM X64位。在使用Emacs 24.2打开文件时候，发现中文字体部分显示为方块。通过对编码的设置依然不能解决问题。在网上查找解决方案的时候，发现有人提到通过设置字体能够解决这个问题。于是仔细看了下emacs里的那些个方块，里面的内容其实是中文的编码，由于不能显示对应的文字，Emacs于是原样将字符编码给打印出来。能够显示的中文也很丑，歪歪扭扭。进入控制面板里查看了下字体，发现中文该有的字体都有，只是不同的是，英文版下字体自然也是英文名称。于是根据谷歌的搜索，在.emacs文件的最开始写入如下内容：

`(set-fontset-font "fontset-default" 'gb18030' ("Microsoft YaHei" . "unicode-bmp"))`

Reference: [emacs在win8乱码](http://blog.csdn.net/qianchenglenger/article/details/10950769)
