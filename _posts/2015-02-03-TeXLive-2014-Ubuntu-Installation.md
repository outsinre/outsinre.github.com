---
layout: page
title: TeXLive 2014 Installation under Ubuntu 14.04
---

Unlike the Windows installer, Unix installer leaves many post-installation steps, especially the "fonts" and "Chinese" problem.

Basics
===
1. Go to [official website](http://tug.org/texlive "TeXLive").
2. Download and unpack `install-tl.zip` which will show the installler like `install-tl-advanced.bat`, `install-tl.bat` and `install-tl`.

	You may found other installer like `install-tl-windows.exe` and ` install-tl-unx.tar.gz`. The former one is only for Windows while the later one is only for Unix system.
3. Run `sudo ./install-tl -gui` to install TeXLive for all users. As the expert GUI mode needs "perl:tk" which is not installed. So the installer degrades into text mode.
4. Use the default configuration some of which can be modified through "/usr/share/texlive/2014/texmf.cnf" file after installation. Pay attention to the option "Create Symlinks in System directories". Leave it alone as "NO".
5. Before the real installation, read the first three chapters of [official documentation](http://tug.org/texlive/doc/texlive-en/texlive-en.html).
6. The while installation process might need one and a half hour. JUST WAIT!

Configuration
===

1. Environment variables for Unix. Add the following three lines to the end of ~/.profile.

	PATH=/usr/local/texlive/2014/bin/x86_64-linux:$PATH; export PATH
	MANPATH=/usr/local/texlive/2014/texmf-dist/doc/man:$MANPATH; export MANPATH
	INFOPATH=/usr/local/texlive/2014/texmf-dist/doc/info:$INFOPATH; export INFOPATH
2. Environment variables: Global configuration. Add the following line to file "/etc/manpath.config".

	MANPATH_MAP /usr/local/texlive/2014/bin/i386-linux /usr/local/texlive/2014/texmf-dist/doc/man
3. System font configuration for XeTEX and LuaTEX. Enable XeTEX and LuaTEX to use fonts shipped with TeXLive.
	1. Copy "/usr/local/texlive/2014/texmf-var/fonts/conf/texlive-fontconfig.conf" to "~/.config/fontconfig/font.conf".
	2. Run "fc-cache -fv".

	"texlive-fontconfig.conf" file specifies the TeXLive fonts locations. Copy this file to home foler as per-user fonts configuration. The official site assumes "to ~/.fonts.conf" as the destination folder, which is depricated now! Details please refer to post [Per user fonts.conf file](http://askubuntu.com/questions/202389/per-user-fonts-conf-file "user home fonts.conf file"). We can find in this post the new single user font configuration file location is `$XDG_CONFIG_HOME/fontconfig/fonts.conf`.

Test the Installation
---

Follow the official documentation for system test. Everything works fine except the "Chinese" issue. The current configuration cannot generate Chinese file since "xelatex" use windows fonts as default. We need to install windows fonts in Ubuntu though this is not legal.

Install Windows Chinese Fonts
===

1. Find the four fonts file from windows `simfang.ttf  simhei.ttf  simkai.ttf  simsun.ttc`.
2. Make a new folder "winfonts" under home directory and copy the four fonts into it.
3. I want to install this four fonts in home directory for the current user. Most of the tutorial online insall them as system-wide fonts. We can find a line `<dir prefix="xdg">fonts</dir>` in file "/etc/fonts/fonts.conf". This line denotes the location for single user fonts file location which is actually `$XDG_DATA_HOME/fonts` = `.local/share/fonts`. Details refer to [fonts.conf - Font configuration files](http://manpages.ubuntu.com/manpages/raring/man5/fonts-conf.5.html).
4. Make a folder "fonts" under ".local/share/" and move "winfonts" in step 2 into ".local/share/fonts/".
5. Run `fc-cache -fv` to update the fonts installation.
6. If the fonts were installed for system-wide users, step 5 will be different. Please refer to [Ubuntu 12.04 LTS 中安装 windows 字体](http://www.cnblogs.com/zhj5chengfeng/p/3251009.html) and [Ubuntu 下安装并配置 TeXLive](http://kayzhang.com/install-texlive-under-ubuntu/).
7. The last but most important step. We need to modify file "/usr/local/texlive/2014/texmf-dist/tex/latex/ctex/fontset/ctex-xecjk-winfonts.def" content. Because this file determines how engine `xelatex` finds the windows fonts installed. There are two ways to modify the file. Mainly focusing on the "KaiTi" and "FangSong" font name issue.
 1. Run `fc-list :lang=zh-cn | sort` to see the Chinese fonts file name and fonts name.
 2. Change "SIMKAI.TTF" to "KaiTi", and "SIMFANG.TTF" to "FangSong". This method helps find Chinese fonts through the "fontconfig' tool in Ubuntu.
 3. Change "SIMKAI.TTF" to "simkai.ttf" and "SIMFANG.TTF" to "simfang.ttf". Add line `OSFONTDIR = ~/.local/share/fonts//;/usr/share/fonts//;/usr/local/share/fonts//` to file "/usr/local/texlive/2014/texmf.cnf". This finds the fonts by fonts file name directly without the help of system "fontconfig" tool. So we need to add a line for `OSFONTDIR` in file "texmf.cnf". Since Ubuntu is case sensitive, so the file names "ctex-xecjk-winfonts.def" should be exactly the same as those in ".local/share/fonts/winfonts".
 4. For the second method, there is no need to install the fonts, we can put the fonts file anywhere as long as we define variable `OSFONTDIR` correctly. Details refer to [ubuntu 10.04 Tex Live 2010 + XeTex + ctex中文配置](http://blog.csdn.net/lostaway/article/details/6177486), [有关类 Unix 系统下 ctex 宏包的字体问题 ](https://code.google.com/p/ctex-kit/wiki/UnixFonts), and [Ubuntu 下安装并配置 TeXLive](http://kayzhang.com/install-texlive-under-ubuntu/).

References
===

1. [ubuntu fonts](https://wiki.ubuntu.com/Fonts)
2. [fonts.conf](http://manpages.ubuntu.com/manpages/raring/man5/fonts-conf.5.html)
3. [single user fonts.conf](http://askubuntu.com/questions/202389/per-user-fonts-conf-file)


