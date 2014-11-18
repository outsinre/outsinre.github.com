---
layout: post
title: Github Tips
---

# Checkout
If you ever changed some files or folders and found them inappropriate, you can revert/discard the changed by command <strong>checkout</strong> command.

For example, if you changed the file *file.txt*, then you can use command:

    git checkout -- file.txt

If you want to discard all the changes:

    git checkout -- .

# Clean
If you have many untracked or ignored files in your repository, use command:

    git clean -ndx [path]

`n` is to try a dry-run, only shows what files will be cleaned. For real action, remove `n` argument. `d` means untracked folders as well. `x` means don’t use the standard ignore rules read from .gitignore. `X` means removing only files ignored by Git. Option `[path]` is to only clean a sub-directory of current repository. For example, many `~` trailing files in `_post` folders:

    git clean -idx _post/

`i` is used for interactive mode which ask you yes or not when cleaning files. To simplify things, replace `i` with `f` which will force removing without hints.

#### Reference
1. [cleaning up untracked files](http://gitready.com/beginner/2009/01/16/cleaning-up-untracked-files.html).

# Line endings
Windows adopts `CRLF` while Linux adopts `LF`, which usually incur conflicts if you or your team edit your repository on both platforms. Basically, we set a universal line ending scheme as `LF`, even for Windows. The key for universal line ending is per-repository `.gitattribute` file setting. Never count on `autocrlf` thing under Windows.

## Windows
```
git config --global core.autocrlf false
git config --global core.safecrlf true
```

## Linux
```
git config --global core.autocrlf input
git config --global core.safecrlf true
```

#### Reference
1. [dealing-with-line-endings](https://help.github.com/articles/dealing-with-line-endings/#platform-all)
2. [line-endings-in-git](https://github.com/ninehills/blog.ninehills.info/blob/master/2012-5-line-endings-in-git.md)
3. [GitHub 第一坑换行符自动转换](http://blog.csdn.net/leonzhouwei/article/details/8933605#t0)
4. [fatal: LF would be replaced by CRLF in](http://stackoverflow.com/questions/15467507/trying-to-commit-git-files-but-getting-fatal-lf-would-be-replaced-by-crlf-in)

# Upload local directory as a repository

1. Go to Github web, create a repository. You can init the repository **with** or **without** a `README.md` file, which makes a difference on the procedures below.
2. Go to the local directory for commands
1. `git init`. Initialize the local directory as a Git repository. This will creat a hiden floder called `.git` all the git-related stuff will be there.
2. `git add .` Add the files in your new local repository. This stages them for the first commit.
5. `git commit -m 'First commit'`. Commits the tracked changes and prepares them to be pushed to a remote repository.
6. `git remote add origin <remote repository URL>`. The remote repository URL is the `https` URL of the repository you created in the 1st step.
  - `git remote -v` Verifies the new remote URL.
7. [optional] If you create the remote repository **with** a `REAMDME.md`:
  - `git pull origin master` to pull the `README.md` first, otherwise you could not push local directory contents to remote. The git assumes your local copy lags behind of the remote origin.
8. `git push origin master`. Pushes the changes in your local repository up to the remote repository you specified as the origin.

#### Reference
1. [adding an existing project to github using the comand line](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)

# gitignore

`gitignore` files can ignore files that will not be pushed to remote repository.There are several kinds of `gitignore` files including `.gitignore`, `.gitignore_global`, and `.git/info/exclude`.

#### Reference
1. [ignoring files](https://help.github.com/articles/ignoring-files/)
2. [gitignore](http://git-scm.com/docs/gitignore)

# Reference
1. [A very good simple cheatsheet](http://rogerdudler.github.io/git-guide/). It is not a good tutorial. But you can refer to it as you forget something.