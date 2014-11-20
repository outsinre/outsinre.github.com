---
layout: post
title: Emacs Killing and Deleting
---

**Killing**: put the killed text in a ring for recovery; usually earase a text blcok.

`C-k`  (kill-line) kills all the text from point up to the end of the line.

`C-Shift-<DEL>` kills tall text from point up to the beginning of the line.

`M-x kill-whole-line` kills the whole line.

`C-w` kill a selected region. `M-w` copies the region into the kill ring.

`M-d` kills the next word. `M-<DEL>` kills the previous word.

`C-x <DEL>` kills back to beginning of sentence. `M-x` kills to the end of the sentence.

**Delete**: no ring involved; erase a character or several whitespaces at a time.

`C-d` (the <delect> key) deletes the next character. `<DEL>` ( the <Backspace> key) deletes the the previous character.
