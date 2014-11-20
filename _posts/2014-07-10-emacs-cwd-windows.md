---
layout: post
title: Emacs Default CWD on Windows
---

`CWD` denotes *current working directory*.

When starting Emacs through Windows shortcut, and using `C-x C-f` to open a file, you find that the default directory is `C\windows\system32`.

To change the default directory is:

1. Either edit the Emacs shortcut, in the `start in` field, fill in your default working directory.
2. Or add a line in your `init.el` file like `(setq default-directory "/path/to/documents/directory/")`.

The firt method is better, since you should keep your `init.el` file consistent for future use.