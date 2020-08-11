---
layout: post
title: Git Configuration
---

1. toc
{:toc}

# Installation

1. Install

   ```bash
   # emerge -av dev-vcs/git
   ```

# Configuration

1. `touch ~/.config/git/config`

   If you'd like a per-user config, create this file first.

   In Gentoo Xfce4, `$XDG_CONFIG_HOME` is not set by default. Without touching this file manually, it would defaults to `~/.gitconfig` instead.
2. `git config --system user.name "<id+username>@users.noreply.github.com"`

   Specify a name so your commits will be properly labeled. It can be anything that is meaningful to you.
3. `git config --system user.email "<username>"`

   Tell Git the email address that will be associated with your Git commits. The email you specify should be the same one found in your email settings. To keep your email address hidden, see [Keeping your email address private](https://help.github.com/articles/keeping-your-email-address-private/).
4. `git config --system core.editor emc`

   You can configure the default text editor that will be used when Git needs you to type in a message (without `-m` opton on push). If not configured, Git uses your system’s default editor, which is generally one of Vim, Emacs and Nano.
5. `git config --system credential.helper 'cache --timeout=3600'`

   If you're [cloning GitHub repositories using HTTPS](https://help.github.com/articles/which-remote-url-should-i-use), you can use a *credential helper* to tell Git to remember your GitHub username and password for a while every time it talks to GitHub. By default, Git will cache your password for 15 minutes (900 seconds).
6. `git config --system push.default simple`
7. For EOL conversion, refer to [git line ending conversion](/2014/09/08/git-line-ending-conversion/).
8. `git config --system -l`

   Check current config.
9. For a single user system, you'd best use *--system* instead of *--global*.

# Gitignore

For language-specific ignorance, refer to [A collection of useful .gitignore templates](https://github.com/github/gitignore).

1. Two consective stars followed by a slash (`**/`) means to match all directories. Without the trailing slash, to match anything regardless of files or directories.

   ```
   # ignore name 'foo'
   **/foo
   # equivalent to:
   foo

   # ignore name 'bar' under directory 'foo'
   **/foo/bar

   # ignore anything under directory 'foo'
   foo/**
   ```

   1. All patterns are relative to the location of the *.gitignore* file unless a leading slash is provided.
   2. Without trailing slash, a name can match a file or a directory.
2. Ignore a directory but a file.

   ```
   my-dir/**
   !my-dir/my-file
   ```

   The consecutive stars `**` is to improve performance. Another way to solve this issue is creating a new *.gitignore* under *my-dir*:

   ```
   # my-dir/.gitignore
   
   *
   !.gitignore
   
   !my-file
   ```

   The newly created *.gitignore* has higher priority over those in uppper directories.
3. `~/.config/ignore`

   This is the global *ignore* list that applys to all Git repositories for current user.

   For example, on my Gentoo:

   ```
   *~
   .#*
   \#*\#
   ```

   The first line ignores Linux file auto-backup. The next two lines are for Emacs eidtor's backup.
4. If a file or directory is accidently pushed to remote repository, you decide to remove that later on.

   Firstly, create a *.gitignore* file. Put patterns there to ignore unwanted names.

   ```bash
   ~ $ git rm -r --cached name; git commit -m "remove name"; git push
   ```
