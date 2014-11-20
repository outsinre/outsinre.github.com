---
layout: post
title: Emacs Commands
---

## Redo Undo

>We can use `C-/` to undo editing. But how do we **redo**?

   Consecutive repetitions  of the  `C-_`, `C-x  u` or  `C-/` commands undo earlier and  earlier changes, back to the limit  of what has been recorded.  If all recorded changes  have already been undone, the undo command prints an error message and does nothing.

   Any command other than an undo  command breaks the sequence of undo commands.  Starting  at this  moment, the  previous undo  commands are considered ordinary changes that can  themselves be undone.  Thus, you can redo changes you have undone  by typing `C-f' or any other command that will have no important effect, and then using more undo commands.

   **Short explanation**: by undoing the undo. If you undo, and then do a non-editing command such as `C-f`, then the next undo will undo the undo, resulting in a redo.

   **loger explanation**:

You can think of undo as operating on a stack of operations. If you perform some command (even a navigation command such as `C-f`) after a sequence of undo operations, all the undos are pushed on to the operation stack. So the next undo undoes the last command. Suppose you do have an operation sequence that looks like this:

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

**Reference**

1. [(emacs)Undo](http://www.cs.cmu.edu/cgi-bin/info2www?%28emacs%29Undo)
2. [How do you 'redo' changes after 'undo' with Emacs?](http://stackoverflow.com/questions/3527142/how-do-you-redo-changes-after-undo-with-emacs)

## Killing and Deleting

**Killing**: put the killed text in a ring for recovery; usually earase a text blcok.

`C-k`  (kill-line) kills all the text from point up to the end of the line.

`C-Shift-<DEL>` kills tall text from point up to the beginning of the line.

`M-x kill-whole-line` kills the whole line.

`C-w` kill a selected region. `M-w` copies the region into the kill ring.

`M-d` kills the next word. `M-<DEL>` kills the previous word.

`C-x <DEL>` kills back to beginning of sentence. `M-x` kills to the end of the sentence.

**Delete**: no ring involved; erase a character or several whitespaces at a time.

`C-d` (the <delect> key) deletes the next character. `<DEL>` ( the <Backspace> key) deletes the the previous character.
