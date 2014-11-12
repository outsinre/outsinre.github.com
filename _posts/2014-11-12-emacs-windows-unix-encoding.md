---
layout: post
title: Emacs Encoding between Unix and Windows
---

>有时候同一个文件在不同的系统下编码不一样，导致乱码的问题。这时，就要用到emacs的一些M-x命令

`M-x revert-buffer-with-coding-system`这是改变buffer的编码，并不是真正的改变文件的编码，可以起到临时的查阅作用。

`M-x set-buffer-file-coding-system`这是设置buffer所对应的文件的编码，表示彻底改变编码了。

Reference: [How to switch back text encoding to UTF-8 with emacs?](http://superuser.com/questions/549497/how-to-switch-back-text-encoding-to-utf-8-with-emacs)
