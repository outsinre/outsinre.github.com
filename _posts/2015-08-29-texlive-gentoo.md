---
layout: post
title: TeXLive Gentoo
---

1. toc
{:toc}

# ABCs

1. Before anything else, read [2 Overview of TEX Live](https://www.tug.org/texlive/doc/texlive-en/texlive-en.html#x1-80002)
2. Read [LaTeX](/2015/02/05/LaTeX/) on TeX *engine* and *format*.

# Installation

```bash
root@tux / # echo "app-text/texlive xetex cjk l10n_zh science publishers" > /etc/portage/package.use/texlive
root@tux / # emerge -avt app-text/texlive
root@tux / # emerge -avt font-adobe-100dpi font-adobe-75dpi
```

1. *xetex* is compatible with Fontconfig.

   XeTeX includes xeCJK macro package which invokes XeTeX engin to compile Chinese TeX files.
2. (opt) *cjk* draws in CJK (*dev-texlive/texlive-langcjk* and *dev-tex/cjk-latex*)

   The old CJK macro script requires more user involvement.
3. *l10n_zh* (*dev-texlive/texlive-langchinese*)

   CTeX extends and depends on CJK, thus making Chinese TeX compiling much easier.
4. USE flags like *extra* depend on a world of packages many of which are unnecessary. If you are sure, just install what we need on the fly.

   Specially, *publishers* and *science* USE is useful when you write conference papers like IEEE, ACM etc.
5. Many TeXLive packages depends on 100dpi and 75dpi bitmap fonts.
6. TeXLive layout */etc/texmf/web2c/texmf.cnf*, */usr/share/texmf-dist/web2c/texmf.cnf* and */etc/texmf/texmf.d/05searchpaths.cnf*.

## 2015 to 2016

```bash
root@tux / # emerge -avtC --deselect=n $(qlist -IC texlive)
root@tux / # emerge -av1 app-text/texlive
```

If you are upgrading TeXLive, follow [Tex Live Migration Guide](https://wiki.gentoo.org/wiki/Project:TeX/Tex_Live_Migration_Guide) and [Upgrading TeXLive](https://wiki.gentoo.org/wiki/Upgrading_TeXLive) first.

## TeXLive packages

*app-text/texlive* defines installtion configuration while *app-text/texlive-core*, *dev-texlive/texlive-basic*, *dev-libs/kpathsea* and *web2c* thereof consists of real sources. Separate TeXLive packages (i.e. *dev-tex/biblatex*, *dev-tex/cjk-latex* etc.) will go to TEXMFSITE (*/usr/share/texmf-site*) which is Gentoo-specific directory. This allows for more unitary upgrades or security fixes.

Use *app-portage/pfl* to search online repository locating specific package files.

We can also *manually* put TeXLive packages into TEXMFHOME or TEXMFLOCAL without *portage* management. By convention, we put personal macro files or packages into TEXMFHOME.

For example, for *biblatex* (superior to *natbib*) support:

```bash
root@tux / # emerge -av1 dev-tex/biblatex-3.7-r1
root@tux / # emerge -avt dev-tex/biber-2.7
```

Versions *<dev-tex/biblatex-3.7-r1* would block *dev-texlive/texlive-plaingeneric-2017*. Make sure sub versions (*\*.5*, *\*.7*, *\*.10* etc.) of *biblatex* and *biber* should match.

Since verion over 2.0, *biblatex* package use *biber* (superior to *bibtex*) as te default backend:

```bash
root@tux / # emerge -avt dev-tex/biber
```

We could change it to TeXLive default *bibtex* in TeX source file. Read [bibtex-vs-biber-and-biblatex-vs-natbib](http://tex.stackexchange.com/a/25702) on bibliography.

# Fontconfig

Though CTeX makes use of Fontconfig fonts directly, we need to make sure the fonts name are correctly resolved between Fontconfig and TeX. Check what you have:

```bash
$ fc-list :lang=zh-cn | sort
```

Edit */usr/share/texmf-dist/tex/latex/ctex/fontset/ctex-xecjk-winfonts.def*:

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

XeTeX can resolve Fontconfig fonts by *font file name* apart from *font (family) name*. We should set OSFONTDIR varaible then. Details refer to [TeXLive Ubuntu](/2015/02/03/TeXLive-2014-Ubuntu-Installation/).

## Enable TeXLive fonts

There are many free fonts shipped with TeXLive. For XeTeX, they must be enabled in Fontconfig first:

```bash
# eselect fontconfig list
# eselect fontconfig enable 09-texlive.conf
```

# Chinese

1. CTeX by default requires XeTeX (*xelatex* binary) egnine.
2. Current stable TeXLive-2014 still requires *ctex-xecjk-winfonts.def*. Update font names there as above.
3. Due to a [bug](http://bbs.ctex.org/forum.php?mod=viewthread&tid=78821) of *xdvipdfmx*, "Noto Sans S Chinese" (思源黑体) is garbled. Update to TeXLive-2015 or apply the patch.
