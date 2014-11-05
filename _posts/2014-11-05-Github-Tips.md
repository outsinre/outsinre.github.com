---
layout: post
comments: true
title: Github Gips
---

If you ever changed some files or folders and found them inappropriate, you can revert/discard the changed by command <strong>checkout</strong> command.

For example, if you changed the file *file.txt*, then you can use command:

    git checkout -- file.txt

If you want to discard all the changes:

    git checkout -- .
