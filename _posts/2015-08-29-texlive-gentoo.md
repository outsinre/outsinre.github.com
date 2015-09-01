---
layout: post
title: TeXLive in Gentoo
---
1. Installation

    ```
    # echo "app-text/texlive cjk xetex linguas_zh" > /etc/portage/package.use/texlive
    # emerge -av app-text/texlive
    ```

    1. `cjk` USE draws in *xeCJK* support which is tedious to use while compiling Chinese *TeX* documents.
    2. `xetex` can make use of system fonts. We don't need to care too much about Chinese fonts setting in *TeX* documents as long as those fonts configured in system.
    3. `linguas_zh` draws in *ctex* macro package which is based on *xeCJK* macro package.
    4. <s>`science` offers packages related to academic writing like *algorithms*, *hepthesis* etc.</s>
    5. <s>`extra` offers packages like *bibtex* etc.</s>

    Actually, USE flags like *extra* usually draws in many packages many of which is not necessary. For instalce, I add *extra* to contain *texlive-bibtexextra*. However many other packages were installed as well, like *texlive-fontsextra*, *chktex* etc which might be never used.

    So another way, is to just emerge the specific package needed. Sometimes, to find out which package offers the wanted function, we need,

    1. Look into the *.ebuild* file.
    2. <s># `emerge -av dev-tex/texmfind`. Locate the ebuild providing a certain texmf file through regexp. `texmfind bbm.sty` will return *dev-texlive/texlive-fontsextra*.</s>

        *texmfind* is almost dead since 2010, which results in outdated information.
    3. Google.
	
    If I need *biblatex* (**NOT** *bibtex*) support:

    ```bash
    # emerge -av biblatex
    ```

    > There is a big difference between the two methods. The 1st won't add packages (pulled in by USE flag like `extra`) to *@world*, while the second do. The 2nd method implies that you *explicitly* installed that specific package.
2. Fonts name resolution

    Though *XeTeX* and *CTeX* make use of system fonts, we need to make sure the fonts name is correctly resolved between Gentoo and *TeX*.

    ```bash
    $ fc-list :lang=zh-cn | sort
    ```
    This will list the Gentoo system font names and font files thereof.

    ```bash
    # ect /usr/share/texmf-dist/tex/latex/ctex/fontset/ctex-xecjk-winfonts.def
    ```
    This will show how *TeX* understand the system fonts. *TeX* resolve `KaiTi` and `FangSong` by system *font file-name*, not by *font name*.

    We can see some Chinese font names are not consistent between Gentoo system and *TeX*. So we need to edit *ctex-xecjk-winfonts.def*. Change `[SIMKAI.TTF]` to `KaiTi`, and `[SIMFANG.TTF]` to `FangSong`. Pay attention to *square braces* excluded as well.

    There is another method: let *TeX* resolves systems fonts by *font file-name* instead of *font name*. Details refer to [texlive ubuntu](http://www.fangxiang.tk/2015/02/03/TeXLive-2014-Ubuntu-Installation/).
3. Enable TeXLive fonts

    There are many free fonts shipped with *TeXLive*, we need to enable them for *XeTeX*.

    ```bash
    # eselect fontconfig list
    # eselect fontconfig enable 09-texlive.conf
    ```
    More about *eselect fontconfig*, refer to [fontconfig](http://www.fangxiang.tk/2015/04/13/fontconfig/).
4. In Gentoo, there is *NO* need to set *PATH* etc environment variables.
5. Editor

    Choose *Emacs + Auctex* as the editor. Refer to [emacs configuration](http://www.fangxiang.tk/2014/07/12/emacs-configuration/).
