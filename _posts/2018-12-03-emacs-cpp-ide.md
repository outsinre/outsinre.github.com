---
layout: post
title: Emacs C/C++ IDE
---

1. toc
{:toc}

This tutorial demonstrates the process of setting up a C/C++
development environment within Emacs. It mainly involves [GNU
GLOBAL](http://www.gnu.org/software/global/) and
[ggtags](https://github.com/leoliu/ggtags).

# [Tag](https://stackoverflow.com/q/12922526)

Tag is just an index file storing language objects from project
sources, which allows these objects to be quickly and easily located
and queried by text editors or other utilities.

In regard to tag environment, we should differentiate concepts of
*command* and *format*. `etags` generates 'TAGS' files understood by
Emacs while `ctags` generate 'tags' files for Vi.

'tags' format contains far richer metadata than 'TAGS' format that
cannot be interpreted by Emacs. In spite of that, both formats are
more or less the same.

Emacs package has built-in `etags` and `ctags` commands. There exist
other standalone package providing these commands, such as
[universal-ctags](https://github.com/universal-ctags) and
[exuberant-ctags](http://ctags.sourceforge.net/)(*deprecated*).

Nowadays, `ctags` from *universal-ctags* can generate TAGS format with
`-e` option. Hence, we can concentrate on `ctags`. Besides, `ctags`
supports more languages.

Another package [cscope](http://cscope.sourceforge.net/) that is more
powerful than `ctags` is, in reality, dead. Originally, it encompasses
much more features, especially when C and C++ are concerned. Moreover,
it has its own tag database unlike plain-text format. You will find,
from this tutorial, that `cscope` is silimar to GNU GLOBAL but support
fewer languages.

# Semantics and Completion

Tag only cares about sources navigation. Upon writing code, we want
semantic parser to understand source objects and then achieve auto
completion.

There exists a host of auto-completion packages, like
[company-mode](https://github.com/company-mode/company-mode).

# [GNU BLOBAL](http://www.gnu.org/software/global/)

GNU GLOBAL is an universal source code tagging system across diverse
platforms and environments, such as Emacs, Vim, Bash Shell, various
web browsers.

Through the command `global`, we can easily locate various objects,
such as functions, macros, structs, classes etc. The `gtags` command
is similar to `etags` and `ctags`, but is different from them in the
following two points:

1. Independence of any editor;
2. Capability to treat both definition and reference;
3. Tag database support.

GLOBAL supports 6 langages as C, C++, Yacc, Java, PHP4 and assembly by
built-in semantic parser. Actually, GLOBAL brings along `gtags-cscope`
support which, though outdated, can replace the default parser when
parsing C/C++.

To support a wider range, we can combine GLOBAL, Pygments,
`etags`/`ctags` plug-in parser. As mentioned above, `ctags` is a
superset of `etags`. Ideally, we prefer `ctags` of *universal-ctags*
over the built-in one of Emacs.

# Installation

## Universal Ctags

Before *make* GLOBAL, we should build
[universal-ctags](https://github.com/universal-ctags) `ctags`. The
goal is to replace the default */usr/bin/ctags* from Emacs.

Then supply

```
./configure --prefix=<PREFIX> --with-exuberant-ctags=/usr/local/bin/ctags
# -or-
./configure --prefix=<PREFIX> --with-universal-ctags=/usr/local/bin/ctags
```

argument, telling GLOBAL where *universal-ctags* locates. 

I won't discuss building *universal-ctags* in this post and maybe get
back to topic later on. Instead, the default GLOBAL built-in parser is
enough for C/C++.

## Emerge GLOBAL

Since GLOBAL is independent of editors, it requires imtermediate
bridges. To interact with Emacs, it has *gtags.el*. We should enable
the `emacs` USE:

```bash
root@tux ~ # echo 'dev-util/global emacs' >> /etc/portage/package.use/global (opt)
root@tux ~ # emerge -avt dev-util/global
```

Actually, Emacs also has *etags.el* for default `etags` and *ctags.el*
for default `ctags`.

## [ggtags](https://github.com/leoliu/ggtags)

However, *gtags.el* is quite primitve, we would like the improved
version *ggtags*. So just install *ggtags.el* through ELPA
*list-packages*.

After installing *ggtags*, add to *init.el*:

```lisp
(require 'ggtags)
(add-hook 'c-mode-common-hook
          (lambda ()
            (when (derived-mode-p 'c-mode 'c++-mode 'java-mode 'asm-mode)
              (ggtags-mode 1))))
```

 Another replacement is
[helm-gtags](https://github.com/syohex/emacs-helm-gtags). I will leave
it to yourself.

## [pygments](http://pygments.org/)

In order to support syntax highlighting, we need *pygments*. It can be
installed through package manager or within Python virtual environment.

After installing *pygments*, we also require an intermediate plugin
for GLOBAL, namely [Pygments Plug-in Parser for GNU
GLOBAL](https://github.com/yoshizow/global-pygments-plugin).

Since 6.3.2, GLOBAL includes it by default. For older version, we
should install it manually.

# GLOBAL tag database

A tag is a name of an object/entity in source code. An object can be a
variable, a method definition, an include-operator ...

A tag contains information like name of the entity, location in the
source code, and which source file it belongs to.

GNU GLOBAL makes use of three tag databases:

GTAGS | definition database
--- | ---
GRTAGS | reference database
--- | ---
GPATH | path name database

A definition of a tag is where the tag is implemented (*not*
declaration). For example, a function definition is the block where
the function is actually implemented.

On the contrary, a reference of a tag is where it is used in a source
tree, but not where it is defined.

# Usage

Up to now, we have almost finished the installation and configuration
process. Next, we will examine how these components collaborate with
each other to achieve an intelligent development environment.

The most commonly used command is *ggtags-find-tag-dwim* bound to
`M-.`, which *find a tag by context*. It will jump to a reference if
the tag at point is a definition. Rather, it will jump to a definition
if the tag at point is a reference. The the tag at point is a
*include* header, it jumps to that header file.

Other useful functions are:


ggtags-find-definition | find definition tags
--- | ---
ggtags-find-reference | find reference tags
--- | ---
ggtags-find-other-symbol | find tags that have no definitions
--- | ---
ggtags-find-tag-regexp | find definition tags by *regexp*
--- | ---
ggtags-query-replace | do a query & replace in all files found in a search

# Multiple matches

When a search find multiple matches, a new buffer named
*ggtags-global* is created and *ggtags-navigation-mode* is enabled to
faciliate entry selection by following commands:

M-n | move to the next match
--- | ---
M-p | move to the previous match
--- | ---
M-} | move to the next file
--- | ---
M-{ | move to the previous file
--- | ---
M-= | move to the file where navigation session starts
--- | ---
M-< | move to the first match
--- | ---
M-> | move to the last match
--- | ---
C-M-s, M-s s | use `isearch` to find a match
--- | ---
M-, | abort and go back to the location where search was started


# References

1. [C/C++ Development Environment for Emacs](https://tuhdo.github.io/c-ide.html)
2. [Universal Ctags](https://github.com/universal-ctags/ctags)
3. [ggtags](https://github.com/leoliu/ggtags)
4. [GNU GLOBAL](https://www.gnu.org/software/global/global.html)