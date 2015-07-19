---
layout: post
title: Git config
---

1. # emerge -av dev-vcs/git

2. $ git config --global user.name "Jim Gray"

    Not your Github account name. Tell Git your name so your commits will be properly labeled. It can be anything.
3. $ git config --global user.email "YOUR EMAIL"

    Tell Git the email address that will be associated with your Git commits. The email you specify should be the same one found in your email settings. To keep your email address hidden, see [Keeping your email address private](https://help.github.com/articles/keeping-your-email-address-private/).

4. $ git config --global core.editor ect

    you can configure the default text editor that will be used when Git needs you to type in a message. If not configured, Git uses your system’s default editor, which is generally Vim / Emacs /Nano.

5. $ git config --global credential.helper 'cache --timeout=3600'

    If you're [cloning GitHub repositories using HTTPS](https://help.github.com/articles/which-remote-url-should-i-use), you can use a *credential helper* to tell Git to remember your GitHub username and password every time for a while it talks to GitHub. By default, Git will cache your password for 15 minutes (900 seconds). Here, we set it to 1 hour (3600 seconds).

6. $ git config --global -l

    Check current config.
