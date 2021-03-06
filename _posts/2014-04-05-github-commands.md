---
layout: post
title: Github Commands
---

> This post focues on Git configuration and some commands. For complete Git principle and analysis refer to post *Git Architecture*

# Checkout
If you ever **modified** some files or folders. Then found them inappropriate, you can revert/discard the changes by <strong>checkout</strong> command.

For example, if you changed the file *file.txt*, then you can use command:

    git checkout -- file.txt

This command will copy the lasted committed "file.txt" from the git repository. The staging area and working tree are also updated.

If you want to discard all the changes:

    git checkout -- .

This command is fairly handy when you want to discard several changes across different files.

# Clean

> git clean [-d] [-f] [-i] [-n] [-q] [-e <pattern>] [-x | -X] [--] [path…]

Cleans the working tree by recursively removing files that are not under version control, starting from the current directory.

Normally, only files unknown to Git are removed, but if the <span style="color:blue">-x</span> option is specified, ignored files are also removed. This can, for example, be useful to remove all build products.

If any optional <span style="color:blue"> [path...] </span>arguments are given, only those paths are affected.

If you have many untracked or ignored files in your repository, use command:

    git clean -ndx [path]

`n` is to try a dry-run, only shows which files will be cleaned. For real action, remove `n` argument. `d` means untracked folders as well. `x` means don’t use the standard ignore rules read from `.gitignore`. **Therefore, gitignored files won't be ignored by `git clean` and will be removed**. `X` means removing only files ignored by Git. Option `[path]` is to only clean a sub-directory of current repository. For example, many `~` trailing files in `_post` folders:

    git clean -idx _post/

`i` is used for interactive mode which ask you yes or not when cleaning files. To simplify things, replace `i` with `f` which will force removing without hints. Most of the time, `*~` files are ignored by git, so we need to include `x` option.

#### Reference
1. [cleaning up untracked files](http://gitready.com/beginner/2009/01/16/cleaning-up-untracked-files.html).

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
