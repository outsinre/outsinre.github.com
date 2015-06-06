---
layout: post
title: Environment Variable
---
In Unix-like systems, we have to deal with environment variables such as PATH, MANPATH etc. Here are some tips for reference.

1. Order of directory

    ```shell
    PATH=$PATH:~/opt/bin
    PATH=~/opt/bin:$PATH
        ```
	    The diffeence of the two lines is the order when searching for commands. The 2nd line make directory `~/opt/bin` searched before those ones in original `PATH`.
	    2. `export` or not

    ```shell
    PATH=$PATH:~/opt/bin
    export PATH=$PATH:~/opt/bin
        ```
	    `export` is a shell builtin for `bash`, `ksh` and `zsh`.  The first line omit the `export` keyword which is called `assignment`. We don't need `export` if the variable is already in the environment: any change of the value of the variable is reflected in the environment for modern shells. `PATH` is pretty much always in the environment; all Unix-like system sets it in the very beginning (usually in the very first process, in fact).

    But if the variable is not in the environment, you must add `export`. For example, some environment variable is for specific program newly installed. You could test by command:

    >_$_ echo $VARIABLE_NAME

    **Use `export` if not sure**.

3. `$HOME` or `~`

    ```shell
    export  PATH="$PATH:$HOME/bin"
    PATH="$PATH:$HOME/bin"
    PATH=$PATH:$HOME/bin
    PATH=$PATH:~/bin
    PATH="$PATH:~/bin"
        ```
	    **The last line does not work**. The `~` character is only expanded to your home directory when it's the first character of a word and it's unquoted. For the last line, the `~` is between double quotes and therefore not expanded. Even if you wrote `export "PATH=$PATH:"~/Unix/homebrew/bin`, the `~` would not be expanded because it is not at the beginning of a shell word.

    **Use `$HOME` instead of `~`**.
    4. Double quotes or not

    If you have `~` in variable settings, omit double quotes. In all other cases, this does not matter.
    5. `~/.profile` is the place to put stuff that applies to your **whole session**, such as programs that you want to start when you log in (but not graphical programs, they go into a different file, i.e. `~/.xinitrc`), and environment variable definitions.
    6. `~/.bashrc` is the place to put stuff that applies only to **bash itself**, such as alias and function definitions, shell options, and prompt settings. (You could also put key bindings there, but for bash they normally go into `~/.inputrc`.)
    7. `~/.bash_profile` can be used instead of `~/.profile`, but it is a `bash`-version `~/.profile`, not used by any other shell. (This is mostly a concern if you want your initialization files to work on multiple machines and your login shell isn't bash on all of them.) This is a logical place to include `~/.bashrc` if the shell is interactive.
    8. You may see here and there recommendations to either put environment variable definitions in `~/.bashrc` or always launch login shells in terminals. Both are bad ideas. The most common problem with either of these ideas is that your environment variables will only be set in programs launched via the terminal, not in programs started directly with an icon or desktop menu or keyboard shortcut.

    But in Gentoo, the official Wiki recommends per-user setting in `~/.bashrc` which is sourced by `~/.bash_profile`.
    9. `~/.bash_profile` is executed for login shells, while `~/.bashrc` is executed for interactive non-login shells. When you login (eg: type username and password) via console, either physically sitting at the machine when booting, or remotely via ssh: `.bash_profile` is executed to configure things before the initial command prompt. But, if you've already logged into your machine and open a new terminal window (xterm) inside Gnome or KDE, then .bashrc is executed before the window command prompt. `~/.bashrc` is also run when you start a new bash instance by typing `/bin/bash` in a terminal.
    10. Execution order:

    >Login shell: /etc/profile -> /etc/env.d/* -> /etc/bash/bashrc -> /etc/profile.d/*.sh -> ~/.bash_profile -> ~/.bashrc

    >Interactive shell: /etc/bash/bashrc -> ~/.bashrc

References: [1], [2] and [3].


[1]:http://unix.stackexchange.com/a/26059
[2]:http://superuser.com/a/183980
[3]:http://unix.stackexchange.com/a/25704
