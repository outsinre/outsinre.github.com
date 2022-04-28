---
layout: post
title: cjktty
---

>An alternative is Univt patch.

In this post, we will show how to extract *cjktty.patch* from a patched kernel. Afterwards, we will apply the extracted patch to our own kernel. During the process, we might look into several relevant commands.

1. Suppose our current system kernel is 4.0.5. We will generate a *cjktty.patch* suitable for this kernel.
2. The patched kernel is [linux-cjktty](https://github.com/Gentoo-zh/linux-cjktty) from which we will extract the desired patch.

   According to past experience, *cjktty.patch* for kernel *3.18 ~ 3.19* fits *4.0* as well. We will choose the branch *3.19-utf8*.
3. Clone the chosen branch locally.

   ```bash
   $ cd ~/workspace
   $ git clone --branch 3.19-utf8 --depth 4 https://github.com/Gentoo-zh/linux-cjktty.git
   ```
   
   1. `--branch`: we only need branch 3.19-utf8. If not specified, git will clone all the branches, spending a day maybe depending on bandwidth.
   2. `--depth`: from the repository, we find the the top 3 commits are enough to extract our *cjktty.patch*. We MUST clone at least the top `3 + 1 = 4` commits.
4. *git log*

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
   
   From the output, we can see only 4 commits history as expected. We will extract the patch from the top 3 commits authored by *microcai*.
5. Extraction

   ```bash
   ~ $ git diff d8cf08a565defc2f9810b9ecd33b1b3211b029e1 9a5a7d3215307e28df3aea6ac09931a4d55e151e -- > cjktty.patch
   # or
   ~ $ git diff HEAD~3 -- > cjktty.patch
   # or
   ~ $ git format-patch -3 HEAD --stdout > cjktty.patch
   # or
   ~ $ git format-patch -3 9a5a7d3215307e28df3aea6ac09931a4d55e151e --stdout > cjktty.patch
   ```
   
   1. Pay attention to the order of commit ID (SHA1 hash). Put the earliest commit ID (4th) before the latest one (HEAD).
   2. HEAD~3 means extract patch from top to 3rd commits (inclusive). The latest ID (HEAD) precedes the 3rd one.
   3. *format-patch* is similar to *git diff*. *-n <commit-ID>* means patch *n* commits leading up to <commit-ID> (inclusive) of current branch. The patch file orgnization is a little different from that of *git diff*.

      If you use the *git diff* command to compare the 3rd patch with the 1st, you will find them the same patch but with different patch format. Specially, *format-patch* contains more information.
   4. We recommend to use *format-patch* to include commit messages, making it more appropriate for most scenarios involving exchanging patches over the Internet. Details refer to reference 1.
6. Test the patch

   Attention please; the `-p1` must be right, otherwise the wrong files would be patched even if no errors prompt up. Inpsect the patch file to see the pathname.

   ```bash
   ~ # cd /usr/src/linux/
   
   # determine the '-p' option value
   ~ # less /path/to/cjktty.patch

   ~ # patch --verbose -p1 --dry-run < /path/to/cjktty.patch

   ~ # git apply --verbose --whitespace=warn -p1 --stat /path/to/cjktty.patch
   ~ # git apply --verbose --whitespace=warn -p1 --numstat /path/to/cjktty.patch
   ~ # git apply --verbose --whitespace=warn -p1 --check /path/to/cjktty.patch
   ~ # git apply --verbose --whitespace=warn -p1 --summary /path/to/cjktty.patch
   ```
   
   When testing patch file, it might remind errors like:

   ```
   Hunk #17 FAILED at 2708.
   # or
   error: patch failed: drivers/video/console/fbcon.c:2689
   error: drivers/video/console/fbcon.c: patch does not apply
   ```
   
   1. Error number 2708 (reported by *patch*) refers to line number in Linux-cjktty sources, while 2689 (reported by *git apply*) in Gentoo-sources. Search 2708 or 2689 in patch file and check either/both kernel sources.
   2. Some error are caused by extra whitespaces, especially those on *blank* lines.
7. Apply the patch

   ```bash
   ~ # cd /usr/src/linux/
   ~ # patch --verbose -p1 < /path/to/cjktty.patch
   ~ # git apply --verbose --whitespace=warn -p1 /path/to/cjktty.patch
   ```
   
7. If you want to make sure the patch is applied correctly,

   ```bash
   ~ # cd /usr/src/linux/
   ~ # patch --verbose -p1 --dry-run -R < /path/to/cjktty.patch
   ~ # git apply --verbose --whitespace=warn -p1 --check -R /path/to/cjktty.patch
   ```
   
   You can reverse a patch by adding *-R* argument.
8. If /usr/src/linux source is polluted by patch error, you can re-install the kernel source.

   >We cannot *reverse* a patch error.

   ```bash
   # emerge -av --unmerge gentoo-sources-4.0.5
   # rm -r /usr/src/linux-4.0.5-gentoo
   # emerge -av gentoo-sources-4.0.5
   ```
   
   1. Pay attention to use *unmerge* instead of *depclean*. The former will keep package dependencies.
   2. Maybe we can just extract the source code to */usr/src/linux*.
9. Reference
   1. [How to get patch files from multiple commits?](http://stackoverflow.com/q/32643640/2336707).
