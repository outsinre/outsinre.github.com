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


If you have many untracked or ignored files in your repository, use

    git clean -ndx [path]

command. `n` is to try a dry-run, only shows what files will be cleaned. If you want to really clean, remove `n` argument. `d` means removing unsed folders as well. `x` means remove git ignored files such as those with a `~` trailing. Option `[path]` is to only clean a sub-directory of current repository. For example, many `~` trailing files in `_post` folders:

    git clean -idx _post/

`i` is used for interactive mode which ask you yes or not when cleaning files. To simplify things, replace `i` with `f` which will force removing without hints.

Detail referring to [cleaning up untracked files](http://gitready.com/beginner/2009/01/16/cleaning-up-untracked-files.html).