---
layout: post
title: Emacs Configuration
---

# README

1. Be patient.
2. Build your own step by step.
3. Long-term maintenance.

# ABCs

## frame VS window

Emacs 中两个概念容易混淆，即 frame 和 window。

1. frame 就是我们通常所说的“窗口”，譬如，在系统菜单点击 GNU Emacs，那么 eamcs就会运行，那个就称为frame，其实就是an Emacs process/an Emacs instance，指Emacs的整個窗口。
2. window 是frame 里的小窗口。譬如，`C-x 2` 可以把 frame 分成2个 window，每一个 window 编辑不同的文件（对应一个当前 buffer）。

## [Help Summary](https://www.gnu.org/software/emacs/manual/html_node/emacs/Help-Summary.html)

1. `C-h f`: help on *fuction*.
2. `C-h v`: help on *variable*.
3. `C-h m`: help on loaded *modes*.
4. `C-h P`: help on *packages*.
5. `C-h k`: help on *key* bindings.
6. `C-h t`: builtin basic *tutorial*.

## Killing and Deleting

1. Kill VS. Delete

    *Killing*: put the killed text in a ring for recovery; usually earase a text blcok.

    *Delete*: no ring involved; erase a character or several whitespaces at a time.
2. `C-k`  (kill-line) kills all the text from point up to the end of the line.
3. `C-Shift-<DEL>` kills tall text from point up to the beginning of the line.
4. `M-x kill-whole-line` kills the whole line.
5. `C-w` kill a selected region. `M-w` copies the region into the kill ring.
1. `M-d` kills the next word. `M-<DEL>` kills the previous word.
1. `C-x <DEL>` kills back to beginning of sentence. `M-x` kills to the end of the sentence.
1. `C-d` (the <delect> key) deletes the next character. `<DEL>` ( the <Backspace> key) deletes the the previous character.

## Useful keys

1. `C-x h`: select all.
2. `C-g`: during execution of Lisp code, this character causes a quit directly. You can use ‘C-g’ to cancel any action you have started.

    For some actions, you may need to repeat this. (If even this doesn’t clear things up entirely, then try `C-]` or `M-x top-level`; that should do the trick.)

    `ESC-ESC-ESC`: three consective ESC key. This command can exit an interactive command such as `query-replace (M-%)`,can clear out a prefix argument or a region,can get out of the minibuffer or other recursive edit,cancel the use of the current buffer (for special-purpose buffers),or go back to just one window (by deleting all but the selected window).

## Undo/Redo

This section describe the builtin Undo/Redo, NOT the *undo-tree* (see configuration below) package.

>We can use `C-/` to undo editing. But how do we **redo**?

Consecutive repetitions of the `C-_`, `C-x  u` or  `C-/` commands undo earlier and earlier changes, back to the limit of what has been recorded. If all recorded changes have already been undone, the undo command prints an error message and does nothing.

Any command other than an undo command breaks the sequence of undo commands. Starting  at this moment, the previous undo commands are considered ordinary changes that can  themselves be undone. Thus, you can redo changes you have undone by typing `C-f' or any other command that will have no important effect, and then using more undo commands.

*Short explanation*: by undoing the undo. If you undo, and then do a non-editing command such as `C-f`, then the next undo will undo the undo, resulting in a redo.

*loger explanation*: you can think of undo as operating on a stack of operations. If you perform some command (even a navigation command such as `C-f`) after a sequence of undo operations, all the undos are pushed on to the operation stack. So the next undo undoes the last command. Suppose you do have an operation sequence that looks like this:

    Insert "foo"
    Insert "bar"
    Insert "I love spam"

Now, you undo. It undoes the last action, resulting in the following list:

    Insert "foo"
    Insert "bar"

If you do something other than undo at this point - say, `C-f`, the operation stack looks like this:

    Insert "foo"
    Insert "bar"
    Insert "I love spam"
    Undo insert "I love spam"

Now, when you undo, the first thing that is undone is the undo. Resulting in your original stack (and document state):

    Insert "foo"
    Insert "bar"
    Insert "I love spam"

If you do a modifying command to break the undo sequence, that command is added after the undo and is thus the first thing to be undone afterwards. Suppose you backspaced over "bar" instead of hitting `C-f`. Then you would have had

    Insert "foo"
    Insert "bar"
    Insert "I love spam"
    Undo insert "I love spam"
    Delete "bar"

This adding/re-adding happens ad infinitum. It takes a little getting used to, but it really does give Emacs a highly flexible and powerful undo/redo mechanism.

*reference*:

1. [(emacs)Undo](http://www.cs.cmu.edu/cgi-bin/info2www?%28emacs%29Undo)
2. [How do you 'redo' changes after 'undo' with Emacs?](http://stackoverflow.com/questions/3527142/how-do-you-redo-changes-after-undo-with-emacs)

## file and buffer Encoding

有时候同一个文件在不同的系统下编码不一样，导致乱码的问题。这时，就要用到 Emacs 的一些M-x命令来临时改变编码。

1. `M-x revert-buffer-with-coding-system` 这是改变 buffer 的编码，并不是真正的改变文件的编码，可以起到临时的查阅作用。
2. `M-x set-buffer-file-coding-system`这是设置 buffer 所对应的文件的编码，表示彻底改变编码了。

*reference*:

1. [How to switch back text encoding to UTF-8 with emacs?](http://superuser.com/q/549497)

# Configuration

> Everything is synced on GitHub repository - *.emacs.d*

1. Install (elpa or manually) and configure packages.
3. `M-x: list-packages`，会启动 Emacs 自带的插件管理器 elpa (Emacs lisp package archive)。例如找到 auctex，按下 `i` 标记为安装，再按 `x` 开始安装。除了 Emacs 官方的 elpa，还有添加 melpa，org 等 package archive。
4. If edit *init* files manually, pay attention the *lisp* grammar. They can also be modified by *M-x: customize variable*. Remember to *C-x C-s* saving the updates.
5. If Emacs is running at *daemon* server mode, updates to *init* files does **Not** take effect until *daemon* server is re-launched!

    So when configuring Emacs, use *emacs* instead of C/S mode *emacsclient*.
6. *Common Lisp* needs a slash ending of directory name while *Emacs Lisp* does not.

*reference*:

1. [purcell](https://github.com/purcell/emacs.d)
2. [master emacs in one year](https://github.com/redguardtoo/mastering-emacs-in-one-year-guide)
2. [Emacs配置文件——新手攻略](https://www.zybuluo.com/qqiseeu/note/17692)

## Find init

Normally Emacs uses the environment variable HOME to find .emacs; that’s what ‘~’ means in a file name. If .emacs is not found inside ~/ (nor .emacs.el), Emacs looks for ~/.emacs.d/init.el (which, like ~/.emacs.el, can be byte-compiled). 

However, if you run Emacs from a shell started by `su`, Emacs tries to find your own .emacs, not that of the user you are currently pretending to be. The idea is that you should get your own editor customizations even if you are running/pretending as the super user. If use `su -`, then you are not pretending but indeed that user.

More precisely, Emacs first determines which user’s init file to use. It gets your user name from the environment variables LOGNAME and USER; if neither of those exists, it uses effective user-ID. If that user name matches the real user-ID, then Emacs uses HOME; otherwise, it looks up the home directory corresponding to that user name in the system’s data base of users. 

## init directory

Run `$ tree -a -C -F -L 2 -I *~ .emacs.d/`:

```
.emacs.d/
├── .git/                   # git directory
│   ├── COMMIT_EDITMSG
│   ├── HEAD
│   ├── config
│   ├── description
│   ├── hooks/
│   ├── index
│   ├── info/
│   ├── logs/
│   ├── objects/
│   ├── packed-refs
│   └── refs/
├── .gitignore
├── LICENSE.md
├── README.md
├── auto-save-list/
├── elpa/                   # pkg installed through builtin package manager
│   ├── archives/
│   ├── auctex-11.88.9/
│   ├── auctex-11.88.9.signed
│   ├── company-0.8.12/
│   ├── elpy-1.10.0/
│   ├── evil-readme.txt
│   ├── find-file-in-project-3.8/
│   ├── flx-0.6.1/
│   ├── flx-ido-0.6.1/
│   ├── flx-ido-readme.txt
│   ├── fullframe-0.1.1/
│   ├── gnupg/
│   ├── goto-last-change-readme.txt
│   ├── highlight-indentation-0.7.0/
│   ├── ido-completing-read+-3.7/
│   ├── ido-ubiquitous-3.7/
│   ├── pyvenv-1.9/
│   ├── smex-3.0/
│   ├── swiper-0.6.0/
│   ├── undo-tree-0.6.5/
│   ├── undo-tree-0.6.5.signed
│   ├── undo-tree-readme.txt
│   ├── yasnippet-0.9.0.1/
│   └── yasnippet-readme.txt
├── init.el                 # first init file loaded
├── lisp/                   # specific pkg's init file
│   ├── init-auctex.el
│   ├── init-custom.el
│   ├── init-elpa.el
│   ├── init-elpy.el
│   ├── init-evil.el
│   ├── init-goto-chg.el
│   ├── init-ido.el
│   ├── init-site-lisp.el
│   ├── init-undo-tree.el
│   ├── init-utils.el
│   └── init-win-nt.el
└── site-lisp/              # manually installed pkgs located here.
    ├── evil/
    └── goto-chg/
```

## Load init

1. When Emacs starts, it first load *init.el* which in turn load package-specific *init-file-name.el* configs under *lisp/*.

    All *init-file-name.el* files MUST end up with `(provide `init-file-name)` (file name without *.el* extension).
2. *site-lisp/* is where manually installed packages located, i.e. *goto-chg* and *evil*.
3. *elpa/* stores packages installed through Emacs builtin package manager *package.el*.
4. Each time a package is installed:
    1. Usually add an *init-pkg-name.el* config file under *lisp/*. This file is specific to that installed package only.

        If this package is manually installed, then the config file needs more lisp codes to adjust performance. However, if it installed by builtin package managers, most configs are done in *elpa/pkg-name-version/*. Just a few arguments is enough.
    2. Add `(require 'init-pkg-name)` in *init.el*.
5. In Emacs Lisp code, you can require the loading of a Lisp library that provides a feature, as follows:

    ```lisp
    (require 'the-feature "the-library-file" <soft-flag>)
    ```
    A library declares that it provides a certain *feature* as follows:

    ```lisp
    (provide 'the-feature)
    ```
    Refer to [Required feature](http://www.emacswiki.org/emacs/RequiredFeature).

## init

The very first feature loaded on startup.

```lisp
;; Check Emacs version.
(let ((minver "23.3"))
  (when (version<= emacs-version "23.1")
    (error "Your Emacs is too old -- this config requires v%s or higher" minver)))
(when (version<= emacs-version "24")
  (message "Your Emacs is old, and some functionality in this config will be disabled. Please upgrade if possible."))

;; Init lisp el load path. Common lisp needs the end slash.
;; People with a CommonLisp background like to make sure that
;; the entry ends with a trailing slash, but this is not required by EmacsLisp.
(add-to-list 'load-path (expand-file-name "lisp" user-emacs-directory))

;; Temporarily reduce garbage collection during startup
(defconst sanityinc/initial-gc-cons-threshold gc-cons-threshold
  "Initial value of `gc-cons-threshold' at start-up time.")
(setq gc-cons-threshold (* 128 1024 1024))
(add-hook 'after-init-hook
          (lambda () (setq gc-cons-threshold sanityinc/initial-gc-cons-threshold)))

;; Load init lisp el. Each required feature corresponds to
;; a feature file - feature.el under ~/.emacs.d/lisp/ directory.
(require 'init-utils)
(require 'init-site-lisp) ;;Must come before init-elpa, as it may provide package.el
(require 'init-elpa)
(require 'init-custom)
(require 'init-auctex)
(require 'init-elpy)
(require 'init-ido)
(require 'init-goto-chg)
(require 'init-undo-tree)
(require 'init-evil)
(require 'init-win-nt)


(provide 'init)
```

1. *init* itself is also a feature. End with

    ```lisp
    (provide 'init)
    ```
    to provide feature. A feature must be *provide*d before *require*d.
2. Check Emacs version and give warning/error if too old.
3. *user-emacs-directory* refers to *~/.emacs.d/*.
4. Reduce garbage collection on startup from [purcell](https://github.com/purcell/emacs.d).
5. Pay attention to *require* orders, i.e. *init-site-lisp* comes before *init-elpa*.

## init-util

This feature is copied from [purcell](https://github.com/purcell/emacs.d). There are many functions and a special macro - *after-load* (alias as *with-eval-after-load*). Self-defined functions/macros are only useful when you are familiar with and really nee them.

Feature *site-lisp* need *after-load* macro support.

```lisp
(if (fboundp 'with-eval-after-load)
    (defalias 'after-load 'with-eval-after-load)
  (defmacro after-load (feature &rest body)
    "After FEATURE is loaded, evaluate BODY."
    (declare (indent defun))
    `(eval-after-load ,feature
       '(progn ,@body))))
```

## init-site-lisp

*site-lisp* mainly achieve two items:

1. Add each subdirectory of *site-lisp* to *load-path*.

    ```lisp
    (setq load-path (remove (expand-file-name "site-lisp/test/" user-emacs-directory) load-path)))
    or
    (add-to-list 'load-path (expand-file-name "lisp" user-emacs-directory))
    ```
    These two statements both operates on a single directory.
2. If Emacs version <= 23, download a package manager *package.el*.

```lisp
;; Add each subdirectory at different levels of site-lisp to the beginning
;; of load-path,including site-lisp itself. To remove site-lisp itself,
;; remove:
;;  (append
;;   (copy-sequence (normal-top-level-add-to-load-path '(".")))
;; To remove a subdir of site-lisp from load-path:
;;  (setq load-path (remove (expand-file-name "site-lisp/test/" user-emacs-directory) load-path)))
;; For example, site-lisp/package is removed if Emacs version > 23.
;; site-lisp/evil/lib is removed from load-path as well.
;; Adding to the end of load-path is much easier, pleae refer to
;;   http://www.emacswiki.org/emacs/LoadPath.

;; lisp and site-lisp are both in load-path but serve different roles.
;; lisp is to configure package behaviour while manually installed packages
;; are located in site-lips.

;; It will add subdirs at different levels to load-path.
(let ((default-directory (expand-file-name "site-lisp" user-emacs-directory)))
  (setq load-path
        (append
         (let ((load-path (copy-sequence load-path))) ;; Shadow
           (append 
            (copy-sequence (normal-top-level-add-to-load-path '(".")))
            (normal-top-level-add-subdirs-to-load-path)))
         load-path)))
```

## init-elpa

1. Add several package repositories to *package manager* - package.el

    *org* and *melpa stable*.
2. On demand installation of packages.

    ```lisp
    (require-package 'package-name)
    (require 'package-name)
    ```
    With this code in your config, self-defined function *require-package* will download and load *package-name* automatically!

## init-custom

Mainly personal like to adjust the basics of Emacs.

```lisp
;;; Configs that change the basics of Emacs are put here.

;; Enable line number mode on startup.
(global-linum-mode 1)

;; Display colum number at mode line.
(setq column-number-mode t)

;;; Language - UTF-8

;; Disable CJK encoding
(setq utf-translate-cjk-mode nil) 
;; Set UTF-8 language environment. Don't use 'UTF-8 or
;; 'utf-8! LANGUAGE-NAME should be a string - the name
;; of a language environment.  For example, "Latin-1".
(set-language-environment "UTF-8")
(set-default-coding-systems 'utf-8)
(set-terminal-coding-system 'utf-8)
(set-keyboard-coding-system 'utf-8)
(setq locale-coding-system 'utf-8)
(set-selection-coding-system 'utf-8)
(setq-default buffer-file-coding-system 'utf-8)
;; priority based on reverse order, so the last one is used first
(prefer-coding-system 'utf-8)


(provide 'init-custom)
```

*reference*:

1. [2009-07-09](http://masutaka.net/chalow/2009-07-09-1.html)
2. [windows下Emacs中文乱码解决办法](http://blog.csdn.net/sanwu2010/article/details/23994977)

## init-auctex

Please refer to [auctex emacs](http://jimgray.tk/2015/01/30/auctex-emacs/).

## init-elpy

Emacs Python Development Environment.

```lisp
;; lisp/init-elpy.el
(elpy-enable)

(provide 'init-elpy)
```

1. *elpy* requires *pyeven* - python virtualenv support which is a global minor mode.

    So when programming, Emacs is aware of and can activate/deactivate virutalenv! By default, *pyvenv* mode is turned globally on Emacs startup.

    Mainly through command `M-x: pyvenv-activate` and `M-x: pyvenv-deactivate`.
2. In order to support *elpy* better, we can `pip install` several backends like *rope*, *jedi*, *flake8*, *importmagic*, *autopep8*, *yapf* etc.

    Install these Python packages in *virtualenv* instead of in global environment. Do NOT mess true system up.

*reference*:

1. [elpy github](https://github.com/jorgenschaefer/elpy)
2. [readdoc elpy](https://elpy.readthedocs.org/en/latest/index.html)

## init-ido-smex

[IDO](http://www.emacswiki.org/emacs/InteractivelyDoThings) lets you interactively do things with buffers and files.

Emacs now has a builtin *ido.el* which just focuses on files and buffers. However *ido-ubiquitous.el* enable ido-style completion everywhere for almost every function that uses the standard completion function.

1. First enable builtin *ido*:

    ```lisp
    ;;; Enable ido in as many places as possible

    (ido-mode 1)
    (ido-everywhere 1)
    ```
2. Enable *ido-ubiquitous*:

    ```lisp
    ; (require 'ido-ubiquitous)
    (ido-ubiquitous-mode 1)
    ```

## init-goto-chg

*evil* depends on *goto-chg* which is not in stable *melpa* repository. So install both *evil* and *goto-chg* manually. Download and put *goto-chg.el* to *site-lisp/goto-chg/*.

```lisp
;;; Goto the point of the most recent edit in the buffer.
;;; When repeated, goto the second most recent edit, etc.
;;; Negative argument, C-u -, for reverse direction.
;;; Works by looking into buffer-undo-list to find points of edit.

(require 'goto-chg)

;; Define key bindings. Otherwise M-x: goto-last-change[-reverse]
;; each time.
(global-set-key [(control ?.)] 'goto-last-change)
(global-set-key [(control ?,)] 'goto-last-change-reverse)


(provide 'init-goto-chg)
```

Two *global* key bindings (`C-.` and `C-,`) are defined for *goto-chg*. These two key bindings work everywhere no matter what modes are Emacs in. *evil* itself also define two key bindings (`g-;` and `g-,`) only work within *evil* mode.

## init-undo-tree

Emacs's builtin undo/redo (see above *Undo/Redo*) is confusing and not intuitive to use. This package is to replace Emacs' undo system with a system that treats undo history as what it is: a branching tree of changes. This simple idea allows the more intuitive behaviour of the standard undo/redo system to be combined with the power of never losing any history.

```lisp
;;; Replace Emacs built-in undo/redo with undo-tree.

;; Turn on undo-tree globally everywhere.
(global-undo-tree-mode 1)


(provide 'init-undo-tree)
```

## init-evil

*evil* is not in stable *melpa* repository. So I install it manually.

1. [Emacs Evil](http://www.emacswiki.org/emacs/Evil)
2. Download and extract evil source to *site-lisp/evil/*.
3. Add

    ```lisp
    ;;; Load evil mode

    ;;; evil requires undo-tree and goto-chg which must be
    ;;; loaded at first.

    ;; evil was installed manually, so require it first before enabling it.
    (require 'evil)
    (evil-mode 1)
    ```
    By setting `(evil-mode 1)`, Emacs enters evil mode by default. `C-z` to switch between evil and normal Emacs.

## init-win-nt

Mainly special options for Windows system.

```lisp
;;; Windows system specific configs.

(if (eq system-type 'windows-nt)
    (progn
      ;; Set default directory.
      (setq default-directory "E:/workspace")
      ;; Set Chinese fonts, otherwise mojibak in a mess.
      (set-fontset-font "fontset-default" 'gb18030'("Microsoft YaHei"."unicode-bmp"))
      )
)


(provide 'init-win-nt)
```
There are many other Windows configs, but integrated into other features, like AucTeX PDF viewer in *init-auctex*.

### Emacs 24.3 Chinese characters on Windows

系统为英文版 Windows RTM X64。在使用 Emacs 24.3 打开文件时候，发现中文字体部分显示为方块。通过对编码的设置依然不能解决问题。在网上查找解决方案的时候，发现有人提到通过设置字体能够解决这个问题。于是仔细看了下 Emacs 里的那些个方块，里面的内容其实是中文的编码，由于不能显示对应的文字，Emacs 于是原样将字符编码给打印出来。能够显示的中文也很丑，歪歪扭扭。进入控制面板里查看了下字体，发现中文该有的字体都有，只是不同的是，英文版下字体自然也是英文名称。于是根据谷歌的搜索，在 init 文件的最开始写入如下内容：

`(set-fontset-font "fontset-default" 'gb18030' ("Microsoft YaHei" . "unicode-bmp"))`

*reference*:

1. [Emacs在win8乱码](http://blog.csdn.net/qianchenglenger/article/details/10950769)

### Default PWD on Windows

`PWD` denotes *print name of current/working directory*. When starting Emacs through Windows shortcut, and using `C-x C-f` to open a file, you find that the default directory is `C\windows\system32`.

To change the default directory by either of the two:

1. Either edit the Emacs shortcut, in the `start in` field, fill in your default working directory.
2. Or add:

    ```lisp
    (setq default-directory "E:/workspace")
    ```
# Issues

1. require vs autoload vs load[-libarry]
