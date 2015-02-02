---
layout: page
title: TeXLive 2014 Installation under Windows
---

This is the details on installing TeXLive 2014 under Windows 8.1.

1. Go to [official website](http://tug.org/texlive "TeXLive").
2. Download and unpack `install-tl.zip` which will show the installler like `install-tl-advanced.bat`, `install-tl.bat` and `install-tl`.

	You may found other installer like `install-tl-windows.exe` and ` install-tl-unx.tar.gz`. The former one is only for Windows while the later one is only for Unix system.
3. Run `install-tl-advanced.bat` as administrator to install TeXLive for all users.
4. Use the default configuration except the one `texmfhome`. I change it from `~\texmf` to `E:\texmf`.

	If several users in this machine. How to distinguish the different user data under `texmfhome` in `E:\texmf`?
5. Before the real installation, read the first three chapters of [official documentation](http://tug.org/texlive/doc/texlive-en/texlive-en.html).
6. The while installation process might need one and a half hour. JUST WAIT!
7. Once finished, follow chapter 3.4 [3.4 Post-install actions](http://tug.org/texlive/doc/texlive-en/texlive-en.html#x1-300003.4).