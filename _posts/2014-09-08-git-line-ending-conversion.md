---
layout: post
title: Git Line Ending Conversion
---

>End of Line style(EOL) differs on various system, especially between Windows and Unix. Team work across systems must pay due attention to EOL conversion. If not properly addressed, some commands like `diff` cannot work.

When adding new file or modified file to staging area, you might get errors:

`fatal: LF would be replaced by CRLF in newfile.txt`  
or  
`fatal: CRLF would be replaced by LF in newfile.txt`

# In and out of the `object database`

What is the object database? Two different operations: writing to the object database and writing out to the `working directory`. It helps to understand these concepts a bit before moving on.

You may already know that *Git has it's own database* in that `.git` folder. All you need to know is that when you do something like `git commit` you are *writing objects into the database*. This involves taking the files that you are committing, calculating their shas and writing them into the object database as blobs. This is what I mean when I say writing to the object database and this is when Git has a chance to run filters and do things like converting line endings.

The other place that Git has a chance to run filters is when it *reads out of the object database and writes files into your working directory*. This is what I mean when I say writing out into the working directory. Many commands in Git do this, but `git checkout` is the most obvious and easy to understand. This also happens when you do a `git clone` or run a command like `git reset` that changes your working directory.

>Up here, we have two writing destinations: to the repository database and to the working directory.

# Parameters that Influence

1. global config
  1. core.autocrlf
  2. core.eol
  3. core.safecrlf
2. .gitattributes file
  1. `text` attribute
  2. `eol` attribute
3. System file Editor configuration

The file editor configuration should be adjusted according to the first two items. Therefore, we first discuss the previous two parameters. The `global config` applies to all the files which are detected as *text* files, while `.gitattributes` attributes apply to line ending conversion for specific file types resulting in fine-grained conversion control. **One or more  `.gitattributes` files might exist in different directories of the repository**.

`global config` is mainly set by command like

> git config --global xxx yyy

where `xxx` and `yyy` are attributes and values respectively.

`.gitattributes` file is mainly organized as pattern lines. Each line is as:

> pattern    attr1 attr2 ...

`pattern` mainly determines the file types, i.e. `*.md`, `*.txt`, and `*.cpp`.



# global config

`global config` is set by command or mannaully edited. The command is as follows:

    git config --global core.autocrlf true/false/input
    git config --global core.eol native/eol/crlf
    git config --global core.safecrlf true/false/warn

Use `git config --global core.xxx` to show the corresponding value. If you have not yet defined the `xxx` value, Git have default values:

    core.autocrlf=false # don't do CRLF to LF conversion when writing to repository database
    core.eol=native     # do LF to system native EOL style when writing to working tree

Though `core.eol=native` is the default setting by Git, but if `.gitattributes` and `core.autocrlf` are not set, `core.eol=native` does nothing. It is a relatively passive parameter that come into effect only when the file line ending is normlized when writing to repository.

`global config` is for corse-grained control on line ending control, especially when no specific patterns for a file type can be found in `.gitattributes` file. The exact definition of the parameter is detailed in references.

`core.safecrlf` is very useful, and set to `true` on whatever OS platform. It detects CRLF to LF conversion error when adding file to staging area. Especially when Git faultly detect a binary file to be text file. Git will give `fatal` error. Then you can add a pattern line in .gitattributes file for that binary file.

# .gitattributes file

This file is the main reference for line ending conversion since Git 1.7.2 and above. `global config` acts as the *fall-back reference* if no pattern is defined in  .gitattributes file for specific file type (officially called `pathnames`).

A pattern line example is like:

    *.jpg binary
    *.png -text # same as above
    *.txt text
    *.md eol=lf

`.gitattributes` file is committed into the repository and overrides an individual's core.autocrlf setting, ensuring consistent behavior for all users, regardless of their Git settings. The advantage of a .gitattributes file is that your line configurations are associated with your repository. You don't need to worry about whether or not collaborators have the same line ending settings that you do.

Sometimes you would need to override an setting of an attribute for a path to **Unspecified state**. This can be done by listing the name of the attribute prefixed with an exclamation point **!** like `!text`.

# Parameter Priority

If more than one pattern matches the path, a later line overrides an earlier line. When deciding what attributes are assigned to a path, Git consults `$GIT_DIR/info/attributes` file (which has the highest precedence), `.gitattributes` file in the same directory as the path in question, and its parent directories up to the toplevel of the work tree (the further the directory that contains `.gitattributes` is from the path in question, the lower its precedence). **Finally global and system-wide files are considered** (they have the lowest precedence).

When the .gitattributes file is missing from the work tree, the path in the index is used as a fall-back. During checkout process, .gitattributes in the index is used and then the file in the working tree is used as a fall-back.

If you wish to affect only a single repository (i.e., to assign attributes to files that are particular to one user’s workflow for that repository), then attributes should be placed in the `$GIT_DIR/info/attributes` file. Attributes which should be version-controlled and distributed to other repositories (i.e., attributes of interest to all users) should go into .gitattributes files. Attributes that should affect all repositories for a single user should be placed in a file specified by the `core.attributesfile` configuration option (see git-config[1]). Its default value is `$XDG_CONFIG_HOME/git/attributes`. If `$XDG_CONFIG_HOME` is either not set or empty, `$HOME/.config/git/attributes` is used instead. Attributes for all users on a system should be placed in the `$(prefix)/etc/gitattributes` file.

If several patterns for a *file type* are found, a subsequent pattern line overrides a precedent pattern line. The last pattern line is used.

Parameters in .gitattributes overrides those in global config. For example, `*.txt -text` and `core.autocrlf=true` are both set, then `*.txt` file will not be considered for EOL conversion.

# Parameter Direction

`text` and `core.autocrlf` determines whether whether converting CRLF to LF when writing to repository database. If yes, this implicates line ending conversion **must** be carried out when writing to working tree for that file type. Normalized line ending (to repository database) files must also do line ending conversion when writing to working tree. How to do line ending when writing back?

`eol` in .gitattributes, and `core.eol` determines line ending conversion when writing to working tree. If no `eol` pattern is set for a file type in .gitattributes, then use the `core.eol` in global config. If `core.eol` is not set in global config, then use the default `core.eol=native`.

As mentioned in section *global config*, `core.eol` paly its full paly only when needed. It cannot work independently without other parameters' implication. If a repository bears no .gitattributes file or relevant global setting, the default `core.eol=native` do nothing.

In .gitattributes, `eol` will automatically set `text` attributes. For example, `*.txt eol=lf`. This means `*.txt` file line ending will be converted to LF when writing to working tree. Meanwhile, it assigns `text` to `*.txt` file as `*.txt text`. Therefore, `*.txt` will definitedly do CRLF to LF conversion when writing to repository database.

`eol` or `core.eol` only play their roles when the file is set for normalization when writing to repository database.

# Parameter Scheme

The Git decision procedure:

- writing to repository, whether or not doing CRLF to LF conversion for a file type
  - **If yes**, then depends on `eol` and `core.eol` value to which the new line ending will be converted when writing to working tree
  - **If not**, no line ending conversion when writing to working tree

When editing files in different operating system, use the system native line ending scheme. This is usually set automatically by your file editor.

Everything new created by editor in system working tree adopts native EOL style.

By setting `core.eol=native`. If line ending conversion is a must, then converted to native line ending style.

If edited or new files are prepared for staging or committing, Git refers to `core.safecrlf` for EOL safety check.

**The following settings guarantee line ending in repository is LF independent of operating system, while that in working tree is operating system native value**.

## Windows

```
git config --global core.autocrlf true
git config --global core.safecrlf true
git config --global core.eol native
```
## Linux
```
git config --global core.autocrlf input
git config --global core.safecrlf true
git config --global core.eol native
```
For `core.autocrlf=input`, if a file type is determined by this setting to do CRLF to LF conversion when writing to repository databse, then this file type won't do EOL conversion when writing to working tree. The `input` value to to guarantee overlooked file type by `.gitattributes` to do CRLF to LF conversion when writing to repository database.

## Repository

.gitattributes file contents, a example:

```
# Auto detect text files and perform LF normalization when checkin. This should be the first line for other lines below to override it.
* text=auto

# For .md file specially
 #*.md text eol=lf #eol will overwrite text, so there is no need of text attribute here
 #*.md eol=lf # eol depends on core.eol=native
*.md text

.gitattributes text

# Remove text attribute for binary files，-text and binary are the same purpose
*.jpg -text
*.jpeg binary
*.svg -text
*.png binary
*.pdf -text
```
In `.gitattributes`, there is a pattern line `* text=auto`, this will override the `core.autocrlf=true` or `core.autocrlf=input` in `global config`. But we still need those two global config. Because `* text=auto` is only for the current repository. The other repositories might not have this pattern line in `.gitattributes`

## Conversion Process

According to the decision precedence, .gitattributes file is consulted before global config for line ending conversion.

When a *file type* is written to working tree or repository database, Git run line ending filter for that *file type* to determine whether conversion is needed.

If several entries are found for the *file type* in `.gitattributes` or `global config`, choose one pattern according to patameter priority.

For writing to repository database, `core.safecrlf` will check EOL conversion safety before the writing happens.

1. `* text=auto`: **auto** firstly detects if the the file is `text` or `binary`.
  * If `text` file, then convert CRLF to LF when writing to repository database.
     - Before adding file to staging area, `core.safecrlf` has already checked the conversion safety.
     - When writing back to working tree, according to `core.eol=native`, LF to CRLF conversion will be carried out on Windows. While on Linux, LF to LF will carried out.
     - `text=auto` is a universal setting in `.gitattributes` for those file types that don't have specific pattern line in .gitattributes.
  * If `binary` file, then do nothing related to line ending issue.
2. `*.md` file has specific pattern line that oeverides the `* text=auto`. `*.md` file will convert CRLF to LF when writing to repository. When writing to working tree, depends on `core.eol=native` in global config.
3. Similarly `*.jpg -text` means `*.jpg` file is not `text` file but `binary` file. So no line ending conversion is associated with it. This also applies to other picture formats and PDF file.
4. When you modify a file or add a new file type, then adding to staging area, `core.safecrlf` will do safety verification. If error occurs, then you can:
  - adjust your the file line ending style to allow staging.
  - add a specific line in .gitattributes for fine-grained control on line ending conversion for that file.

# Refreshing a repository after changing line endings

According to the previous setting scheme, EOL in repository databse both in Windows and Linux should be LF. If the database or working tree contains CRLF previously, you may found git wants to commit files that you have not modified.

The best way to automatically configure your repository's line endings is to first backup your files with Git, delete every file in your repository (except the .git directory), and then restore the files all at once.

```
git add . -u
git commit -m "Saving files before refreshing line endings"
git rm --cached -r .
git reset --hard
git add .
# It is perfectly safe to see a lot of messages here that read
# "warning: CRLF will be replaced by LF in file." or "fatal: CRLF will be replaced by LF in file."
# How to solve the fatal error read the first reference.
git commit -m "Normalize all the line endings"
```

# Read the 1st and 2nd reference carefully!

# Reference
1. [fatal: LF would be replaced by CRLF](http://stackoverflow.com/questions/26991025/fatal-lf-would-be-replaced-by-crlf)
2. [Mind the End of Your Line](http://adaptivepatchwork.com/2012/03/01/mind-the-end-of-your-line/)
2. [dealing-with-line-endings](https://help.github.com/articles/dealing-with-line-endings/#platform-all)
2. [line-endings-in-git](https://github.com/ninehills/blog.ninehills.info/blob/master/2012-5-line-endings-in-git.md)
3. [GitHub 第一坑换行符自动转换](http://blog.csdn.net/leonzhouwei/article/details/8933605#t0)
5. [gitattributes - defining attributes per path](http://git-scm.com/docs/gitattributes)
6. [Line endings handling in SVN, Git and SubGit](http://blog.subgit.com/line-endings-handling-in-svn-git-and-subgit/)
7. [fatal: LF would be replaced by CRLF in <some file in the repo>](http://stackoverflow.com/questions/15467507/trying-to-commit-git-files-but-getting-fatal-lf-would-be-replaced-by-crlf-in)
8. [core.eol core.autocrlf](http://git-scm.com/docs/git-config)
