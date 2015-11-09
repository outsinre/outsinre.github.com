---
layout: post
title: AucTeX Emacs
---

> Let Emacs + AucTeX support LaTeX writing and compiling.

# AucTeX

```lisp
;; In order to get support for many of the LaTeX packages you will
;; use in your documents, you should enable document parsing as
;; well, which can be achieved by:
(setq TeX-auto-save t)
(setq TeX-parse-self t)

;; If you want to make AUCTeX aware of style files and multi-file documents
;; right away, insert the following three lines in your 'init.el' file.
(setq-default TeX-master nil)

;; Default to PDF output
(setq-default TeX-PDF-mode t)

;; 'xetex' as the default engine
(setq-default TeX-engine 'xetex)
```

# Inverse/Reverse search

```lisp
;; Method to use for enabling forward and inverse search.
;; https://www.gnu.org/software/auctex/manual/auctex/I_002fO-Correlation.html
(setq TeX-source-correlate-method (quote synctex))

;; Toggles support for forward and inverse search. Forward search
;; refers to jumping to the place in the previewed document corresponding
;; to where point is located in the document source and inverse search to
;; the other way round.
(setq TeX-source-correlate-mode t)

;; If TeX-source-correlate-mode is active and a viewer is invoked,
;; the default behavior is to ask if a server process should be started.
;; Set this variable to t if the question should be inhibited and the
;; server should always be started. Set it to nil if the server should
;; never be started. Inverse search will not be available in the latter case.
;; YOU'D BEST NOT ENABLE IT BY DEFAULT FOR SECURITY REASON.
(setq TeX-source-correlate-start-server t)
```

# Previewer

```lisp
;; Set MuPDF as the default PDF previewer. Now use the default
;; Evince instead since it support forward/reverse search.
(if (eq system-type 'gnu/linux)
    (progn
      (setq TeX-view-program-list '(("MuPDF" "mupdf %s.pdf")))
      ;(setq TeX-view-program-selection '((output-pdf "Evince")))
      )
)

;; For Emacs + Sumatra on Windows, the inverse search:
;;   C:\emacs\bin\emacsclientw.exe +%l "%f"
;; Add the above line to Sumatra's settings:
;;   File -> Set inverse search command line
;; http://william.famille-blum.org/blog/static.php?page=static081010-000413
(if (eq system-type 'windows-nt)
    (progn
      (setq TeX-view-program-list (quote (("Sumatra PDF" ("\"E:/SumatraPDF/SumatraPDF.exe\" -reuse-instance" (mode-io-correlate " -forward-search %b %n") " %o")))))
      (setq TeX-view-program-selection (quote (((output-dvi style-pstricks) "dvips and start") (output-dvi "Yap") (output-pdf "Sumatra PDF") (output-html "start"))))
      )
)
```

My Gentoo system use MuPDF PDF viewer. How to make it the default previewer of AucTeX? Basically, we need to update two variables, which can be achieved by editing *init.el* manually or *M-x: customize-variable* in frame mini-buffer. The following steps only show you how to manully add viewer.

1. TeX-view-program-list

    This variable is a list of viewers for different output formats. Emacs has default builtin viewers for `.dvi`, `.pdf`, `.ps` etc. For example, the default viewer for `.pdf` is Evince. Here I just want to add MuPDF to the list for `.pdf` output. MuPDF does **NOT** support *forward* and *backward* search while Evince, Ookular etc. do.

    If manually edit *init.el*:

    ```lisp
    (setq TeX-view-program-list '(("MuPDF" "mupdf %s.pdf")))
    ```
    Use *C-c C-v* to preview PDF output.

    If command line in mini-buffer, first turn on `M-x tex-mode` or open a `.tex` file.
    
    ```
    M-x customize-variable:  will open the builtin variable editor;
    TeX-view-program-list: open variable for edit
    INS: click to add a viewer to the list except the builtin ones
    Name: MuPDF
    Command: mupdf %s.pdf
    C-x C-s
    ```
    Open the *init.el* file, you will find MuPDF is added to the list.
2. TeX-view-program-selection

    This variable defines the exact viewer to choose from *TeX-view-program-list* when viewing a specific TeX output format. Similarly, edit by manual or by command line.

    If manually:

    ```lisp
    (setq TeX-view-program-selection '((output-pdf "MuPDF")))
    ```

    If command line in mini-buffer:

    ```
    M-x customize-variable:  will open the builtin variable editor;
    TeX-view-program-selection: open for edit
    INS DEL Choice: Value Menu Single predicate: Value Menu output-pdf
                   Viewer: Value Menu MuPDF
    C-x C-s
    ```
    Just locate *Viewer* for *output-pdf*. Click on **Value Menu**, you could get a drop list menu, choose *MuPDF*. The updated *init.el*:

    ```lisp
    (custom-set-variables
     ;; custom-set-variables was added by Custom.
     ;; If you edit it by hand, you could mess it up, so be careful.
     ;; Your init file should contain only one such instance.
     ;; If there is more than one, they won't work right.
     '(TeX-view-program-list (quote (("MuPDF" ("mupdf %s.pdf") ""))))
     '(TeX-view-program-selection
       (quote
        (((output-dvi style-pstricks)
          "dvips and gv")
         (output-dvi "xdvi")
         (output-pdf "MuPDF")
         (output-html "xdg-open")))))
    (custom-set-faces
     ;; custom-set-faces was added by Custom.
     ;; If you edit it by hand, you could mess it up, so be careful.
     ;; Your init file should contain only one such instance.
     ;; If there is more than one, they won't work right.
     )
    ```


# RefTeX and AucTeX

 Since version 24.3 of Emacs, RefTeX is developed exclusively as part of Emacs. So if you want to get the latest version of RefTeX, you should get the latest version of Emacs. (Now outdated versions of the standalone distribution are kept for archeologic purposes on the GNU FTP server.)

```lisp
;;; The following RefTeX lines might be dispensible since AucTeX is installed
;;; through package manager and RefTeX is builtin.

;; Turn on RefTeX for AUCTeX. Read http://www.gnu.org/s/auctex/manual/reftex/reftex_5.html
; with AUCTeX LaTeX mode
(add-hook 'LaTeX-mode-hook 'turn-on-reftex)
; with Emacs latex mode
(add-hook 'latex-mode-hook 'turn-on-reftex)

;; Make RefTeX interact with AUCTeX. Read http://www.gnu.org/s/auctex/manual/reftex/AUCTeX_002dRefTeX-Interface.html
(setq reftex-plug-into-AUCTeX t)
```

# Writing LaTeX

For a bignner, refer to [LaTeX](http://jimgray.tk/2015/02/05/LaTeX/).

*reference*:

1. [4.2.2 Forward and Inverse Search](https://www.gnu.org/software/auctex/manual/auctex/I_002fO-Correlation.html)
2. [Configuring editors with SumatraPDF](http://william.famille-blum.org/blog/static081010-000413.html)
