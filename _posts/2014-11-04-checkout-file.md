---
layout: post
title: Jekyll Poole Lanyon Blog Building
---

<div class="message">
 This is to record the process of build the blog process.
</div>

If you ever changed some files or folders and found them inappropriate, you can revert/discard the changed by command <strong>checkout</strong> command.

For example, if you changed the file *file.txt*, then you can use command:

    git checkout -- file.txt

If you want to discard all the changes:

    git checkout -- .

Originally the sidebar "Download" and "Github Project" items does not work. I found that it is due to *site.github.repo* is not defined in *_config.yml*.

Just add two lines in *_config.yml* as follows. Pay attention to the indent of the second line.

github:

  repo:	https://github.com/outsinre/outsinre.github.com

This is called **custom variable**. Solved! Futhermore, I removed the "Download" item since it is not necessary.