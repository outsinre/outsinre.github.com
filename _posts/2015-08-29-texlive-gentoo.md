---
layout: post
title: TeXLive in Gentoo
---

1. Installation

   ```
   # echo "app-text/texlive cjk xetex l10n_zh" > /etc/portage/package.use/texlive
   # emerge -av app-text/texlive
   ```

   1. *cjk* draws in *xeCJK* which supports Chinese *TeX* documents though need extra efforts.
   2. *l10n_zh* draws in *ctex* macro package which is based on *xeCJK* macro package, thus compiling Chinese docs easier.
   3. *xetex* can make use of system Fontconfig fonts directly.
2. USE flags like *extra* usually draws in many packages many of which are not necessary.

   For instalce, I add *extra* to contain *texlive-bibtexextra*. However many other packages were installed as well, like *texlive-fontsextra*, *chktex* etc which might be never used. So another way, is to just emerge the specific package needed. Sometimes, to find out which package offers the wanted function, we need,
   - Look into the *.ebuild* file.
   - (deprecated) Package *dev-tex/texmfind*. *texmfind bbm.sty* will return *dev-texlive/texlive-fontsextra*.

     *texmfind* is almost dead since 2010, resulting in outdated information.
   - Google.

   For example, if need *biblatex* (an advance version of *bibtex*) support:

   ```bash
   # emerge -av dev-tex/biblatex
   ```

   *dev-tex/biblatex* is marked as *testing* in portage. It is *not* inlcuded in *TeXLive*. Since verion over 2.0, *biblatex* package use *biber* as te *default* backend. You could change it to *bibtex* in *TeX* source. Or if you insist on *biber* backend:

   ```bash
   # emerge -av dev-tex/biber
   ```

   For more on *bibliography* thing, read [bibtex-vs-biber-and-biblatex-vs-natbib](http://tex.stackexchange.com/a/25702).
2. Font name resolution

   Though *XeTeX* and *CTeX* make use of Fontconfig fonts directly, we need to make sure the fonts name are correctly resolved between Fontconfig and *TeX*.

   ```bash
   $ fc-list :lang=zh-cn | sort
   ```
   This will list the system Fontconfig fonts.

   ```bash
   # ect /usr/share/texmf-dist/tex/latex/ctex/fontset/ctex-xecjk-winfonts.def
   ```
   
   This will show how *TeX* understand the system fonts. *TeX* resolve *KaiTi* and *FangSong* by system *font file-name*, not by *font name* (mainly for Windows users).

   We need to edit *ctex-xecjk-winfonts.def*. Change *[SIMKAI.TTF]* to *KaiTi*, and *[SIMFANG.TTF]* to *FangSong*. Pay attention to *square braces* excluded as well.

   There is another method: let *TeX* resolves Fontconfig fonts by *font file-name* instead of *font name*. Details refer to [texlive ubuntu](/2015/02/03/TeXLive-2014-Ubuntu-Installation/).
3. (opt) Enable TeXLive fonts

   There are many free fonts shipped with *TeXLive*, we need to enable them for *XeTeX*.

   ```bash
   # eselect fontconfig list
   # eselect fontconfig enable 09-texlive.conf
   ```
   
   More about *eselect fontconfig*, refer to [fontconfig](/2015/04/13/fontconfig/).
4. In Gentoo, there is *NO* need to set *PATH* etc environment variables.
5. Editor

   Choose *Emacs + Auctex* as the editor. Refer to [emacs configuration](/2014/07/12/emacs-configuration/).
