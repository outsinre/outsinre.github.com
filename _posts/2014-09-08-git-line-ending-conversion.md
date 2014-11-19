---
layout: post
title: Git Line Ending Conversion
---

>End of Line style differs on various system, especially on Windows and Unix. Team work across systems must pay due attention to EOL conversion. If not properly addressed, some commands like `diff` cannot work. Usually, you might get errors like:

> fatal: LF would be replaced by CRLF in newfile.txt

or

> fatal: CRLF would be replaced by LF in newfile.txt

# In and out of the object database

What is the object database? I'm going to talk a lot about two different operations: writing to the object database and writing out to the working directory. It helps to understand these concepts a bit before moving on.

You may already know that Git has it's own database in that .git folder. All you need to know is that when you do something like git commit you are writing objects into the database. This involves taking the files that you are committing, calculating their shas and writing them into the object database as blobs. This is what I mean when I say writing to the object database and this is when Git has a chance to run filters and do things like converting line endings.

The other place that Git has a chance to run filters is when it reads out of the object database and writes files into your working directory. This is what I mean when I say writing out into the working directory. Many commands in Git do this, but git checkout is the most obvious and easy to understand. This also happens when you do a git clone or run a command like git reset that changes your working directory.

# Parameters that Influence

1. global config
  1. core.autocrlf
  2. core.eol
  3. core.safecrlf
2. `.gitattributes` file
  1. `text` attribute
  2. `eol` attribute
3. System file Editor configuration

The file editor configuration should be adjusted according to the first two items. Therefore, we first discuss the previous two parameters. The `global config` applies to all the files which are detected as *text* files, while `.gitattributes` attributes apply to line ending conversion for specific file types resulting in fine-grained conversion control. **One or more  `.gitattributes` files might exist in different directories of the repository**.

`global config` is mainly set by command like

> git config --global xxx yyy

where `xxx` and `yyy` are attributes and values respectively.

Parameters in `.gitattributes` mainly is organized as pattern lines. Each line is as:

> pattern    attr1 attr2 ...

`pattern` mainly determines the file types, i.e. `*.md`, `*.txt`, and `*.cpp`.

# Decision Precedence in `.gitattributes` file

If more than one pattern matches the path, a later line overrides an earlier line. When deciding what attributes are assigned to a path, Git consults `$GIT_DIR/info/attributes` file (which has the highest precedence), `.gitattributes` file in the same directory as the path in question, and its parent directories up to the toplevel of the work tree (the further the directory that contains `.gitattributes` is from the path in question, the lower its precedence). Finally global and system-wide files are considered (they have the lowest precedence).

When the .gitattributes file is missing from the work tree, the path in the index is used as a fall-back. During checkout process, .gitattributes in the index is used and then the file in the working tree is used as a fall-back.

If you wish to affect only a single repository (i.e., to assign attributes to files that are particular to one user’s workflow for that repository), then attributes should be placed in the `$GIT_DIR/info/attributes` file. Attributes which should be version-controlled and distributed to other repositories (i.e., attributes of interest to all users) should go into .gitattributes files. Attributes that should affect all repositories for a single user should be placed in a file specified by the `core.attributesfile` configuration option (see git-config[1]). Its default value is `$XDG_CONFIG_HOME/git/attributes`. If `$XDG_CONFIG_HOME` is either not set or empty, `$HOME/.config/git/attributes` is used instead. Attributes for all users on a system should be placed in the `$(prefix)/etc/gitattributes` file.

Sometimes you would need to override an setting of an attribute for a path to **Unspecified state**. This can be done by listing the name of the attribute prefixed with an exclamation point **!** like `!text`.


# global config

`global config` is set by command or mannaully edited. The command is as follows:

   git config --global core.autocrlf true/false/input
   git config --global core.eol native/eol/crlf
   git config --global core.safecrlf true/false/warn

Use `git config --global core.xxx` to show the corresponding value. `global config` is for corse-grained control on line ending control, especially when no specific patterns for a file type can be found.

## core.autocrlf

### true

This means that Git will process all text files and make sure that CRLF is replaced with LF when writing that file to the object database and **turn all LF back into CRLF when writing out into the working directory**. This is the recommended setting on Windows because it ensures that your repository can be used on other platforms while retaining CRLF in your working directory.

### input

This means that Git will process all text files and make sure that CRLF is replaced with LF when writing that file to the object database. It will **not**, however, do the reverse. When you read files back out of the object database and write them into the working directory they will still have LFs to denote the end of line. This setting is generally used on Unix/Linux/OS X to prevent CRLFs from getting written into the repository. The idea being that if you pasted code from a web browser and accidentally got CRLFs into one of your files, Git would make sure they were replaced with LFs when you wrote to the object database.

### false

This is the default, but most people are encouraged to change this immediately. The result of using false is that Git doesn't ever mess with line endings on your file. You can check in files with LF or CRLF or CR or some random mix of those three and Git does not care. This can make diffs harder to read and merges more difficult. Most people working in a Unix/Linux world use this value because they don't have CRLF problems and they don't need Git to be doing extra work whenever files are written to the object database or written out into the working directory.

## core.eol



## core.safecrlf

How does Git know that a file is *text* attribute file? Good question. Git has an internal method for heuristically checking if a file is binary or not. **A file is deemed text if it is not binary**. Git can sometimes be wrong and this is the basis for our next setting.

The next setting that was introduced is `core.safecrlf` which is designed to protect against these cases where Git might change line endings on a file that really should just be left alone.

### true

When getting ready to run this operation of replacing CRLF with LF before writing to the object database, Git will make sure that it can actually successfully back out of the operation. It will verify that the reverse can happen (LF to CRLF) and if not the operation will be aborted.

### warn

Same as above, but instead of aborting the operation, Git will just warn you that something bad might happen.

### false

Don't check if converting CRLF is reversible when end-of-line conversion is active..

# `.gitattributes` file

This file is the main reference for line ending conversion since Git 1.7.2 and above.