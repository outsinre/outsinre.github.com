---
layout: post
title: Git Discards Local Changes by <em>checkout</em>
---

<div class="message">
 If you ever changed some files or folders and found them inappropriate, you can revert/discard the changed by command <strong>checkout</strong> command.
</div>

For example, if you changed the file *file.txt*, then you can use command:

    git checkout -- file.txt

If you want to discard all the changes:

    git checkout -- .

That is it.