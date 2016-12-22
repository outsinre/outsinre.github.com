---
layout: post
title: TeXLive in Gentoo
---

1. Installation

   1. If you are upgrading TeXLive, follow [Tex Live Migration Guide](https://wiki.gentoo.org/wiki/Project:TeX/Tex_Live_Migration_Guide) first.
   2. TeXLive 2015 ebuild bug. Read [cjk-latex-4.8.3-r1 econf failed](https://bugs.gentoo.org/show_bug.cgi?id=596938).   My current solution is bumping to *cjk-latex-4.8.4* (i.e. create a local ebuild with *>=cjk-latex-4.8.4*). *already fixed in upstream*
   3. *app-text/texlive* handles basic configuration while *app-text/texlive-core* consists of real sources. Some important basic TeXLive macro pckages are split from *app-text/texlive-core* and are built separately, like *dev-libs/ptexenc*, *app-text/ps2pkm*, *dev-libs/kpathsea*, etc. This allows for more unitary upgrades or security fixes.

   ```
   # echo "app-text/texlive xetex cjk  l10n_zh" > /etc/portage/package.use/texlive
   # emerge -av app-text/texlive
   ```

   1. *xetex* makes use of system Fontconfig fonts directly besides those shipped with TeXLive.

      By default, XeTeX pulls xeCJK macro package.
   2. *cjk* draws in CJK which supports Chinese TeX documents though it requires extra efforts.
   3. *l10n_zh* draws in CTeX macro package which is based on CJK/xeCJK, thus compiling Chinese docs easier.
   4. xeCJK extends CJK for XeTeX.

2. USE flags like *extra* and *science* draws in a bundle of packages many of which are not necessary.

   - Even disable *extra* USE, *dev-texlive/texlive-latexextra* is installed as dependency.

   - (obsolete) Package *dev-tex/texmfind* returns what *portage package* you need for specific *TeXLive package*.

     *texmfind* is almost dead since 2010.
   - Even without *dev-tex/texmfind*, we can examine ebuild or Google to find required package.

      TeXLive package *emerged* will go to TEXMFSITE (*/usr/share/texmf-site*) which is Gentoo-specific directory.
   - Manually install TeXLive package to TEXMFLOCAL or TEXMFHOME without *portage* management (not recommended).

   For example, if need *biblatex* (an advance version of *bibtex*) support:

   ```bash
   # emerge -avt dev-tex/biblatex
   ```

   *dev-tex/biblatex* is marked as *testing* in portage. It is *not* inlcuded in *TeXLive*. Since verion over 2.0, *biblatex* package use *biber* as te *default* backend. You could change it to *bibtex* in *TeX* source. Or if you insist on default  *biber* backend:

   ```bash
   # emerge -av dev-tex/biber
   ```

   For more on *bibliography* thing, read [bibtex-vs-biber-and-biblatex-vs-natbib](http://tex.stackexchange.com/a/25702).
3. Font name resolution (TeXLive 2014)

   Though CTeX makes use of Fontconfig fonts directly, we need to make sure the fonts name are correctly resolved between Fontconfig and TeX. Check what you have:

   ```bash
   $ fc-list :lang=zh-cn | sort
   ```

   Edit * /usr/share/texmf-dist/tex/latex/ctex/fontset/ctex-xecjk-winfonts.def*:

   {% highlight TeX linenos %}

   % ctex-xecjk-winfonts.def: Windows 的 xeCJK 字体设置，默认为六种中易字体
   % vim:ft=tex

   \setCJKmainfont[BoldFont={NotoSansHans-Bold},ItalicFont={AdobeKaitiStd-Regular}]
     {AdobeFangsongStd-Regular}
   \setCJKsansfont{NotoSansHans-DemiLight}
   \setCJKmonofont{AdobeSongStd-Light}

   \setCJKfamilyfont{zhsong}{AdobeSongStd-Light}
   \setCJKfamilyfont{zhhei}{NotoSansHans-Bold}
   \setCJKfamilyfont{zhkai}{AdobeKaitiStd-Regular}
   \setCJKfamilyfont{zhfs}{AdobeFangsongStd-Regular}
   % \setCJKfamilyfont{zhli}{LiSu}
   % \setCJKfamilyfont{zhyou}{YouYuan}

   \newcommand*{\songti}{\CJKfamily{zhsong}} % 宋体
   \newcommand*{\heiti}{\CJKfamily{zhhei}}   % 黑体
   \newcommand*{\kaishu}{\CJKfamily{zhkai}}  % 楷书
   \newcommand*{\fangsong}{\CJKfamily{zhfs}} % 仿宋
   % \newcommand*{\lishu}{\CJKfamily{zhli}}    % 隶书
   % \newcommand*{\youyuan}{\CJKfamily{zhyou}} % 幼圆

   \endinput

   {% endhighlight %}

   Remove extra *square braces*. For setCJKmonofont, just use a normal 宋体 since all Chinese characters are monospace (single and same width).

   XeTeX can resolve Fontconfig fonts by *font file name* instead of *font (family) name*. We should add an extra OSFONTDIR varaible. Details refer to [texlive ubuntu](/2015/02/03/TeXLive-2014-Ubuntu-Installation/).
4. (opt) Enable TeXLive fonts

   There are many free fonts shipped with *TeXLive*. For XeTeX, use those fonts by *font name*, they should be enabled in Fontconfig. XeTeX can find those fonts by *font file name* like *FandolKai-Regular.otf*.

   ```bash
   # eselect fontconfig list
   # eselect fontconfig enable 09-texlive.conf
   ```

5. Chinese
    1. CTeX by default requires XeTeX (*xelatex* binary) egnine.
    2. Current stable TeXLive-2014 still requires *ctex-xecjk-winfonts.def*. Update font names there as above.
    3. Due to a [bug](http://bbs.ctex.org/forum.php?mod=viewthread&tid=78821) of *xdvipdfmx*, "Noto Sans S Chinese" (思源黑体) is garbled. Update to TeXLive-2015 or apply the patch.
6. Ref:
   1. Before anything else, read [2 Overview of TEX Live](https://www.tug.org/texlive/doc/texlive-en/texlive-en.html#x1-80002)
   2. TeXLive layout */etc/texmf/web2c/texmf.cnf* or /*/usr/share/texmf-dist/web2c/texmf.cnf*.
   3. Writing [LaTeX](/2015/02/05/LaTeX/).
