---
layout: post
comments: true
title: Github Tips
---

[A very good simple cheatsheet](http://rogerdudler.github.io/git-guide/). It is not a good tutorial. But you can refer to it as you forget something.

If you ever changed some files or folders and found them inappropriate, you can revert/discard the changed by command <strong>checkout</strong> command.

For example, if you changed the file *file.txt*, then you can use command:

    git checkout -- file.txt

If you want to discard all the changes:

    git checkout -- .
