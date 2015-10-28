---
layout: post
title: git diff to get cjktty.patch
---
In this post, we will show how to extract *cjktty.patch* from a patched kernel. Afterwards, we will apply the extracted patch to our own kernel. During the process, we might look into several additional relevant commands.

1. Suppose our current system kernel is 4.0.5. We will generate a *cjktty.patch* suitable for this kernel.
2. The patched kernel is [linux cjktty](https://github.com/Gentoo-zh/linux-cjktty) from which we will extract the desired patch.

    According to past experience, *cjktty.patch* for kernel 3.18 ~ 3.19 fits 4.0.5 as well. We will choose the branch 3.19-utf8.
3. Clone the chosen branch locally.

    ```bash
    cd ~/workspace
    git clone --branch 3.19-utf8 --single-branch --depth 4 https://github.com/Gentoo-zh/linux-cjktty.git
    ```
    1. *--branch*: we only need branch 3.19-utf8. If not specified, git will clone all the branches, spending a day maybe depending on your network performance.
    2. *--single-branch*: combined with *--branch*.
    2. *--depth*: from the GitHub repository, we find the the top 3 commits are enough to extract our *cjktty.patch*. We only need to clone the top 4 commits. By only cloning the latest 4 commits, we save more time and network load.
4. git log

    ```
    commit 9a5a7d3215307e28df3aea6ac09931a4d55e151e
    Author: microcai <microcai@fedoraproject.org>
    Date:   Thu Jan 13 05:18:45 2011 +0800

        [vt] add cjk font that has 65536 chars

    commit 3dbe651a011ac6b6bbd976d96b5d08fb2baf709c
    Author: microcai <microcai@fedoraproject.org>
    Date:   Mon Feb 21 12:58:18 2011 +0800

        [vt] diable setfont if we have cjk font in kernel

    commit a2553800e3e8ea1d13f3e3a4211599a3268ce9a2
    Author: microcai <microcaicai@gmail.com>
    Date:   Fri Jan 9 12:45:07 2015 +0800

        [vt] fix 255 glyph limit, prepare for CJK font support

    commit d8cf08a565defc2f9810b9ecd33b1b3211b029e1
    Author: Greg Kroah-Hartman <gregkh@linuxfoundation.org>
    Date:   Thu Mar 26 14:00:21 2015 +0100

        Linux 3.19.3
    ```
    From the output, we can see only 4 commits history are cloned. We will extract the patch from the top 3 commits authored by microcai.
5. Extract the patch:

    ```bash
    git diff d8cf08a565defc2f9810b9ecd33b1b3211b029e1 9a5a7d3215307e28df3aea6ac09931a4d55e151e -- > cjktty.patch
    or
    git diff HEAD~3 -- > cjktty.patch
    or
    git format-patch -3 HEAD --stdout > cjktty.patch
    or
    git format-patch -3 9a5a7d3215307e28df3aea6ac09931a4d55e151e --stdout > cjktty.patch
    ```
    1. Pay attention to the order of commit ID (SHA1 hash). Put the earliest commit ID (4th) before the latest one (HEAD).
    2. HEAD~3 means extract patch from top to 3rd commits (inclusive). In this command, the latest ID (HEAD) precedes the 3rd one.
    3. *format-patch* generates the same patch. *-n <commit-ID>* means patch for the *n* commits leading up to <commit-ID> (inclusive) of current branch. However, the patch file orgnization is a little different.

        If you use the *diff* command to compare the 3rd patch with the 1st or the 2nd one, you will find them different. But actually they are the same patch but with different patch file format. Specially, the *format-patch* patch file, contains mail information.
    4. We recommend to use *format-patch* to include commit messages, making it more appropriate for most scenarios involving ebbbxchanging patches with other people. Details refer to reference 1.
6. Test the patch

    ```bash
    cd /usr/src/linux/
    patch -p1 --dry-run < /path/to/cjktty.patch
    or
    git apply --whitespace=warn --stat < /path/to/cjktty.patch
    git apply --whitespace=warn --check < /path/to/cjktty.patch
    ```
    When testing patch file, it might remind errors like:
    
    ```
    Hunk #17 FAILED at 2708.
    or
    error: patch failed: drivers/video/console/fbcon.c:2689
    error: drivers/video/console/fbcon.c: patch does not apply
    ```
    The line number (2707 or 2689) remind is where error occurs in source files (portage *gentoo-sources* and/or microcai *linux-cjktty*). Search *2708* or *2689* in patch file and compare it with source files to see what causes the error. You might go to line *2708* or *2689* directly in source files. However it does not work sometimes.
    
    *git apply* can be replaced with another command *git am*.
7. Apply the patch

    ```bash
    cd /usr/src/linux/
    patch -p1 < /path/to/cjktty.patch
    or
    git apply --whitespace=warn < /path/to/cjktty.patch
    ```
7. If you want to check whether a/the patch is applied or not

    ```bash
    cd /usr/src/linux/
    patch -p1 --dry-run -R < /path/to/cjktty.patch
    or 
    git apply --whitespace=warn --check -R < /path/to/cjktty.patch
    ```
    If error occurs, you can reverse the command by adding *-R* option to remove the patched parts.
8. If /usr/src/linux source is polluted with modification or patch, you can re-install the kernel source.

    ```bash
    emerge -av --unmerge gentoo-sources-4.0.5
    rm -r /usr/src/linux-4.0.5-gentoo
    emerge -av gentoo-sources-4.0.5
    ```
    Pay attention to use *unmerge* instead of *depclean*. The former will keep dependency since we will re-install *gentoo-sources* later on.
9. Reference
    1. [How to get patch files from multiple commits?](http://stackoverflow.com/q/32643640/2336707).