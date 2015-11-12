---
layout: post
title: Emacs Configuration
---

# README

1. Be patient.
2. Build your own step by step.
3. Long-term maintenance.
4. Read at least the first two chapters of *Mastering Emacs* book.

# ABCs

For Emacs installation, refer to [Emacs installation](http://jimgray.tk/2014/09/14/emacs-installation/).

1. Emacs consists of 90% Elisp codes and 10% core C codes. The tiny C part interact with the underlying operating system ABI, while of the Elisp part is the most important - *Elisp interpreter*.

    Almost all of our development and configuration are transferred to the interpreter and finally to the C core.
2. All files are buffers, but NOT all buffers are files. Some buffers just a throw-away area to temporarily store snippets from a log file, or manipulate text, or whatever your reason — you just create and name a new buffer.
3. In Emacs, *the buffer is the data structure*. This is an extremely powerful concept because the very same commands you use to move around and edit in Emacs are almost always the same ones you use behind-the-scenes in Elisp.

    Commands/key bindings/function calls on the front end UI are nearly the same ones in the background Elisp. Hence, when you write Elisp codes to specialize/customize Emacs, you use nearly the same set of commands/functions as you edit a file/buffer.
4. It’s important to remember that each buffer can have just one major mode. Minor modes, by contrast, are typically optional add-ons that you enable for some (or all) of yourbuffers.

    The major mode is always displayed in the modeline. Some minor modes are also displayed in the modeline, but usually only the ones that alter the buffer or how you interact with it in some way.

## frame VS window

Emacs 中两个概念容易混淆，即 frame 和 window。

1. frame 就是我们通常所说的“窗口”，譬如，在系统菜单点击 GNU Emacs，那么 Emacs 就会运行，那个整个界面就称为 frame，其实就是 an Emacs process/instance UI，指 Emacs 的整個窗口。
2. window 是 frame 里的小窗口，通常对应一个 buffer。Each frame can have one or more windows, and each window can have exactly one buffer。
3. A *window* is just a tiled *portion* of the *frame*, which is what most modern window managers call a window.

## [Help Summary](https://www.gnu.org/software/emacs/manual/html_node/emacs/Help-Summary.html)

Emacs is a sophisticated self-documenting editor. Every facet of Emacsis searchable or describable.

### Three help utililies

1. The *info* Manual

    ```lisp
    M-x: info
    or
    C-h i
    ```
    Emacs’s own manuals (and indeed, all manuals in the  ecosystem) are written in TeXinfo. Emacs’s info manual contains more than just topics relating to Emacs. By default, the info browser will index all the other info manuals installed on your system. It even shows info manual of Nano editor which is absolutely not part of Emacs.

    We usually use *info* command in terminal shell. That's it. The only difference is Emacs has its own TeXinfo viewer. If you want to focus on Emacs manual only without distraction, press `m: Emacs` after `C-h i`.

    This help utility is to get you into a thorough explanation of Emacs things, which is nearly the same as you type `info` on the command line including the navagation key bindings.
2. Apropos

    Emacs has an extensive apropos system that works in much the same way as *apropos* does on the command line. The apropos system is especially useful if you’re not entirely sure what you’re looking for or you just know a few characters. There is a variety of niche commands that only search particular aspects of Emacs’s self-documenting internals. And all of apropos supports regular expressions.

    The most common usage is:

    ```lisp
    C-h a
    or
    M-x: apropos-command
    ```
    shows all commands (and just the commands, *not* functions) that match a given pattern. For example, you are hunting for commands with "coding", just type `C-h a` followed by `*-coding-*`. There are a wide variaty of *apropos* commands to find useful information. Use `C-h a apropos` to check the list.

    If you’re unsure of what you are looking for – maybe you only have part of a name, or you just remember a bit of the documentation – then apropos is a tool that can help you.
3. The Describe System

    If you know what you’re looking for, then describe will explain what it is. Every facet of Emacs – be it code written in elisp or the core layer written is C – is accessible and indexed through the describe system. From keys, to commands, character sets, coding systems, fonts, faces, modes, syntax tables and more — it’s all there, neatly categorized.
4. To use the three Help Utilities, you always use the prefix key `C-h` (which is called *the help character*) followed by a *help option*.

    1. `C-h f`: help on *fuction*.
    2. `C-h v`: help on *variable*.
    3. `C-h m`: help on loaded *modes*.
    4. `C-h P`: help on *packages*.
    5. `C-h k`: help on *key* bindings.
    6. `C-h t`: builtin basic *tutorial*.
    7. `C-h w`: check the key bindings of a function.
    8. `C-h c`: check the function of a key binding.
    9. `C-x 8 C-h`: display the list of key bindings starting with`C-x 8`.
    10. `C-h C`: to see detailed coding system of current buffer. Though the *modeline* does give us some hints on *coding*, but it is somewhat unclear and obsecure sometimes. `C-h h` to see what language environment your Emacs support.
5. `C-h C-h` to display the list of *help options*.

    **Use the help system to teach you how to get help**.

## Killing and Deleting

1. Kill VS. Delete

    *Killing*: put the killed text in a ring for recovery; usually earase a text blcok.

    *Delete*: no ring involved; erase a character or several whitespaces at a time.
2. `C-k`  (kill-line) kills all the text from point up to the end of the line.
3. `C-Shift-<DEL>` kills tall text from point up to the beginning of the line.
4. `M-x kill-whole-line` kills the whole line.
5. `C-x <DEL>` kills back to beginning of sentence. `M-k` kills to the end of the sentence.
5. `C-w` kill a selected region. `M-w` copies the region into the kill ring.
1. `M-d` kills the next word. `M-<DEL>` kills the previous word.
1. `C-d` (the <delect> key) deletes the next character. `<DEL>` ( the <Backspace> key) deletes the the previous character.

## Useful keys

1. `C-x h`: select all.
2. `C-g`: during execution of Lisp code, this character causes a quit directly. You can use ‘C-g’ to cancel any action you have started.

    For some actions, you may need to repeat this. (If even this doesn’t clear things up entirely, then try `C-]` or `M-x top-level`; that should do the trick.)

    `ESC-ESC-ESC`: three consective ESC key. This command can exit an interactive command such as `query-replace (M-%)`,can clear out a prefix argument or a region,can get out of the minibuffer or other recursive edit,cancel the use of the current buffer (for special-purpose buffers),or go back to just one window (by deleting all but the selected window).
3. `M-:`: evaluate lisp statement.

## Universay arguments

Negative arguments add directionality to commands; digits add repetition or change how a command works.

1. `C-u a`, `C-u 4 a`, `C-u 6 a`, `C-6 a`; `C-u C-u a`; `C-u C-u C-u ... a`.
2. `C-0 a` to `C-9 a`; `M-0 a` to `M-9 a`.
3. `C-- 6 a`; `M-- 6 a`; `C-M-- 6 a`.
4. `M-- M-d` and `C-- M-d` both delete the previous word.

    But it the former is more convenient as there is no need to switch our finger from Ctrl to Alt.

Refer to *Universal Arguments* part of *Mastering Emacs* book. 

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
2. [How do you 'redo' changes after 'undo' with Emacs?](http://stackoverflow.com/q/3527142)

## file and buffer Encoding

有时候同一个文件在不同的系统下编码不一样，导致乱码的问题。这时，就要用到 Emacs 的一些M-x命令来临时改变编码。

1. `M-x revert-buffer-with-coding-system` 这是改变 buffer 的编码，并不是真正的改变文件的编码，可以起到临时的查阅作用。
2. `M-x set-buffer-file-coding-system`这是设置 buffer 所对应的文件的编码，表示彻底改变编码了。

*reference*:

1. [How to switch back text encoding to UTF-8 with emacs?](http://superuser.com/q/549497)

# Configuration

> Everything is synced on GitHub repository - *.emacs.d*

1. Plugin/package/library are the same meaning for Emacs, all regarded as *.el* or *.elc* files. These words are used losely, and do NOT have TECHNICAL definitions in Elisp.

    Emacs 一般称“插件”为 "package" 或者 "library" 。本质上，它们都提供一堆定义好的函数，来实现一些操作，进而实现某个功能。这里多说几句。在 Emacs 中，连移动光标这种最底层的操作都有对应的函数。比如，你在 Emacs 中可以键入 `C-f` 来将光标向右移动一个字符，同时也可键入 `M-x forward-char` 来实现。任何复杂的功能，比如给文档生成一个目录，都可以被分解为一个个操作，或者说调用一个个函数，而这些函数顺序执行下来功能就得到了实现。
2. 当 Emacs 想要加载某个插件时，归根到底需要定位并运行一个（也许是一些）脚本文件，那个脚本里定义了实现插件功能所需的变量和函数。Emacs 将它们转变为可供自己使用的对象（Elisp object），放到运行环境中等待调用。而脚本自身还可以在内部进一步加载其他脚本。
4. If edit *init* files manually, pay attention the *lisp* grammar. They can also be modified by *M-x: customize variable*. Remember to *C-x C-s* saving the updates.
5. If Emacs is running at *daemon* server mode, updates to *init* files does **Not** take effect until *daemon* server is re-launched!

    So when configuring Emacs, use *emacs* instead of C/S mode *emacsclient*.
6. *Common Lisp* needs a slash ending of directory name while *Emacs Lisp* does not.

*reference*:

1. [purcell](https://github.com/purcell/emacs.d)
2. [master emacs in one year](https://github.com/redguardtoo/mastering-emacs-in-one-year-guide)
2. [Emacs配置文件——新手攻略](https://www.zybuluo.com/qqiseeu/note/17692)

## Loading

其实，连整个 Emacs 的启动都可以概括为一句话：加载一系列脚本。只不过这些脚本有的是内置的（built in），有的是你安装的插件包含的，有的是你自己写的。

**配置emacs归根结底是在配置各种各样的脚本。**

Emacs Lisp's Library System: What's `require`, `load`, `load-library`, `load-file`, `autoload`, `feature`?

1. Interactive lisp function.

    `C-h f interactive`: specify a way of parsing arguments for interactive use of a function. *interactive* can be invoked by `M-x:` and prompts user to input arguments on-the-fly. It declares that the function in which it appears is a *command*, and that it may therefore be called interactively (via `M-x` or by entering a key sequence bound to it). 

    Most of time, they are bound to shortcut keys, which is optional.  Without *interactive*, a function can only be called *programmatically* (in Elisp *.el* sources), not from `M-x` nor via key-binding.

    *interactive* functions are functions and therefore not constrained to mini-buffer. Instead, they can be called *programmatically* by supplying a default *interactive* argument (if really need) in Elisp sources.

2. *load-file* - interactive

    Load one specific file by *full file path*. “.el” or “.elc” file name extentions are not auto added, but “.gz” is. Use this when you don't want emacs to guess the file name extention of “.el”, “.elc” or none.

    So basically the file can be put anywhere as long as you remember the file path.

    ```lisp
    M-x: load-file "~/elisp/foo-bar.el"
    or
    (load-file "~/elisp/foo-bar.el")
    ```
    *load-file* is not a smart way to pull in libraries as you have to remember and offer each library full path on call.
3. *load*

    Usually it is better to install files in your *load-path*, though. Load a file by searching through directories of *load-path*. Argument should be just the file name without full path.

    If file extension is omitted, it will auto add *.elc* for compiled version if exist, or add *.el*, or with *.gz*. Preference is given to the *compiled* (*.elc*) version.
4. *load-libarry* - interactive

    It is the *interactive* version of *load*. Can be called by	`M-x` (or key bindings if exists) to read arguments.
5. *require*

    ```lisp
    (require 'foo-bar "the-library-file" <soft-flag>)
    ```
    A library declares that it provides a certain *foo-bar* feature as follows:

    ```lisp
    (provide 'foo-bar)
    ```
    Checks the variable *features*, if symbol *foo-bar* is not a member of that list, *foo-bar* is not loaded yet. If *foo-bar* is not loaded, load it from *the-library-file* (which defined the *foo-bar* feature).
    If *the-library-file* is omitted, the printname of *foo-bar* is used as the file name, and *load* will try to load this name appended with the suffix *.elc* or *.el*, in that order. *the-library-file* (file name) is guessed from the feature name - *foo-bar* in this case.

    Load a package if it has not already been loaded. It won't load a library twice. It is similar to other lang's “require” or “import”.
6. *autoload*

    Load a file only when a function is called - on-demand loading. Associate a function name with a file path. When the function is called, load the file, and execute the function. 

    The *autoload* facility lets you register the existence of a function or macro, but put off loading the file that defines it. The first call to the function automatically loads the proper library, in order to install the real definition and other associated code, then runs the real definition as if it had been loaded all along. Autoloading can also be triggered by looking up the documentation of the function or macro.

    这样做的一个好处是，避免在启动 Emacs 时因为执行过多代码而效率低下，比如启动慢，卡系统等。想象一下，如果你安装了大量的有关 Python 开发的插件，而某次打开 Emacs 只是希望写点日记，你肯定不希望这些插件在启动时就被加载，让你白白等上几秒，也不希望这些插件在你做文本编辑时抢占系统资源（内存，CPU 时间等）。所以，一个合理的配置应该是，当你打开某个 Python 脚本，或者手动进入 Python 的编辑模式时，才加载那些插件。

    Loading a file triggered by calling a function defined is in it.
7. All the loading facilities at the end call the *load* function to do their job. Refer to [how programs do loading ](http://www.gnu.org/software/emacs/manual/html_node/elisp/How-Programs-Do-Loading.html).

## Package/Library/Feature names are not Managed

There is no absolute relation between any concept of package/library/feature/autoload facilities and the file name. If there exists relation, it's just a programming convention.

By convention, if a elisp file name is *xyz-mode.el*, it OFTEN provides a lisp symbol *xyz-mode* as its feature name (if it does at all), and the command to invoke the mode is OFTEN named *xyz-mode*. Sometimes the *-mode* part is omitted in any of {file name, feature symbol name, command name}.

This is only a lose convention. There are a lot exceptions. For example:


- The file *lisp-mode.el* provides the symbol *lisp-mode* as feature, and is invoked by a command named *emacs-lisp-mode*.
- The *cua-base.el* file provides symbols *cua-base* and *cua* as features, and is invoked by a command named *cua-mode*.
- The *text-mode.el* file does not provide any symbol for feature. It is invoked by a command named *text-mode*.
- The file *desktop.el* provides the symbol *desktop* as feature, and the command name to invoke it is *desktop-save-mode*.

All the above means, you could have a file named *Joe-xyz-mode_v2.1.el*, which provides a feature named *abc*, while the command name to activate it may be *opq*, and it might be displayed in mode line as *OPQ helper*. And, this file can be considered as a *package* or *library*.

## elpa

1. Compared to install libraries manually, *elpa* automates the process.
2. `M-x: list-packages`，会启动 Emacs 自带的插件管理器 elpa (Emacs lisp package archive)。例如找到 auctex，按下 `i` 标记为安装，再按 `x` 开始安装。除了 Emacs 官方的 elpa，还有添加 melpa，org 等 package archives。
3. Library is only loaded after init files are parsed. So when modifying a *variable* defined in that library during init files parsing, require/load it first!

    Take *elpa* itself for example, if we would like to add a package archive i.e. *melpa*, we should first `(require 'package)`, then update the variable *package-archive* defined within *package.el*. In *init-elpa.el*:

    ```lisp
    (setq package-enable-at-startup nil)
    (package-initialize)
    ```
    are appended after that. The first line is to disable load of installed packages after all init files. The 2nd line is to load those packages. That is to load installed packages earlier as usual.

## Find init

Normally Emacs uses the environment variable HOME to find .emacs; that’s what ‘~’ means in a file name. If .emacs is not found inside ~/ (nor .emacs.el), Emacs looks for ~/.emacs.d/init.el (which, like ~/.emacs.el, can be byte-compiled). 

However, if you run Emacs from a shell started by `su`, Emacs tries to find your own .emacs, not that of the user you are currently pretending to be. The idea is that you should get your own editor customizations even if you are running/pretending as the super user. If use `su -`, then you are not pretending but indeed that user.

More precisely, Emacs first determines which user’s init file to use. It gets your user name from the environment variables LOGNAME and USER; if neither of those exists, it uses effective user-ID. If that user name matches the real user-ID, then Emacs uses HOME; otherwise, it looks up the home directory corresponding to that user name in the system’s data base of users. 

1. After startup, use `C-h v user-init-file` to see your init file location.
2. If during configuration, you modified and want to verify init files:

    ```lisp
    M-x: load-file
    ```
    To load *HOME/.emacs.d/init.el*. If current buffer is your init file config, use:

    ```lisp
    M-x: eval-buffer
    ```
    You can usually just re-evaluate the changed region. Mark the region of ~/.emacs that you've changed, and then use:

    ```lisp
    M-x: eval-region
    ```
    More read on [How can I reload .emacs after changing it?](http://stackoverflow.com/q/2580650).
    
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

    *package.el* itself is also a Emacs library/package. It specially manages other packages.
4. Each time a package is installed:
    1. Usually add an *init-pkg-name.el* config file under *lisp/*. This file is specific to that installed package only.

        If this package is manually installed, then the config file needs more lisp codes to adjust performance. However, if it installed by builtin package managers, most configs are done in *elpa/pkg-name-version/*. Just a few arguments is enough.
    2. Add `(require 'init-pkg-name)` in *init.el*.

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
3. *user-emacs-directory* refers to *~/.emacs.d/* where '~' is determined by user environment variable `HOME`.

    `C-h v user-emacs-directory` shows the exact path.
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

Emacs's builtin undo/redo (see above *Undo/Redo* section) is confusing and not intuitive to use. This package is to replace Emacs' undo system with a system that treats undo history as what it is: a branching tree of changes. This simple idea allows the more intuitive behaviour of the standard undo/redo system to be combined with the power of never losing any history.

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

# TD

1. require vs autoload vs load[-libarry]
2. elpa packages activate/load default? package-activated-list, so many packages activated, but need only some of them.
3. set default fonts?
2. [emacs 配置整理](http://www.jianshu.com/p/99c3f6fa6355)
3. [从零开始——Emacs 安装配置使用教程 2015](http://www.jianshu.com/p/b4cf683c25f3)
4. [elisp library system](http://ergoemacs.org/emacs/elisp_library_system.html)
2. https://github.com/purcell/emacs.d, https://github.com/bbatsov/prelude
3. eshell?
4. Caps Lock VS Ctrl?
