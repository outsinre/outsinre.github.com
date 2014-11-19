---
layout: post
title: Github Tips
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

# Line endings
Windows adopts `CRLF` while Linux adopts `LF`, which usually incur conflicts if you or your team edit your repository on both platforms. Basically, we set a universal line ending scheme as `LF`, even for Windows. The key is to **depend on your file editor for line ending, not the Git auto conversion**.

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

## My post on StackOverflow
***
[fatal: LF would be replaced by CRLF](http://stackoverflow.com/questions/26991025/fatal-lf-would-be-replaced-by-crlf)
***
Platform: Windows 8.1 Emacs 24.3

The git config `--global -l` shows:

    user.name=username
    user.email=useremail
    core.autocrlf=false
    core.safecrlf=true

Git repository `.gitattributes` file:

    # Auto detect text files and perform LF normalization
    * text=auto
    
    # Custom for Visual Studio
    *.cs     diff=csharp
    *.sln    merge=union
    *.csproj merge=union
    *.vbproj merge=union
    *.fsproj merge=union
    *.dbproj merge=union
    
    # Standard to msysgit
    *.doc	 diff=astextplain
    *.DOC	 diff=astextplain
    *.docx diff=astextplain
    *.DOCX diff=astextplain
    *.dot  diff=astextplain
    *.DOT  diff=astextplain
    *.pdf  diff=astextplain
    *.PDF	 diff=astextplain
    *.rtf	 diff=astextplain
    *.RTF	 diff=astextplain

I do think my Git and repository settings are right.

But everytime I create a new text file with Emacs, I cannot run `git add <newfile>`. The file is encoded by `utf-8-unix`.

The error message is as:

    E:\workspace\repository [master +0 ~2 -0]> git add .
    fatal: LF would be replaced by CRLF in newfile.txt

I don't think is due to emacs editor problem. Because I opened the new file and pretty sure the line ending is `LF` not the windows default `CRLF`.

Which configuration part decides LF will be replaced by CRLF?

---
***EDIT 1***  
If `safecrlf` is set to `warn` the output is:

    warning: LF will be replaced by CRLF in _posts/2014-11-19-test.md.
    The file will have its original line endings in your working directory.

This means that the file was successfully added to the index. My file is encoded by `utf-8-unix`.

---
***EDIT 2***  

Interestingly, if I create a new file with Notepad not the Emacs 24.3, the file can be added without any problem. The difference is Notepad adopts `CRLF` line ending while Emacs 24.3 adopts `LF` line ending.

So the problem is somewhere somehow Git converts `CRLF` to `LF` then back to `CRLF` which generates error for original `LF` line ending file.

---
***EDIT 3***  

Previously, GitHub Windows GUI client warned me no `.gitattributes` file for my repository and recommend a default `.gitattributes` file as above.

I think the problem is from the line `* text=auto`. So I comment out this line.

Everything works good now!

---
***EDIT 4***  

The core is:

- ***DISABLE AUTO LINE ENDING CONVERSION*** by GitHub.

- ***DEPEND ON PLATFORM FILE EDITOR FOR LINE ENDING***.

---
***EDIT 5***

1. `text`
This attribute enables and controls end-of-line normalization. When a text file is normalized, its line endings are converted to `LF` **in the repository**. To control what line ending style is used **in the working directory**, use the `eol` attribute **for a single file** and the `core.eol` configuration variable **for all text files**.
  - Setting the `text` attribute on a path enables end-of-line normalization and marks the path as a text file. End-of-line conversion takes place **without guessing the content type**.
  - Unsetting the `text` attribute on a path tells Git **not to attempt any end-of-line conversion upon checkin or checkout**.
  - When `text` is set to "`auto`", the path is marked for automatic end-of-line normalization. If *Git decides that the content is text*, its line endings are normalized to **`LF` on checkin**.
2. `eol`
This attribute sets a specific line-ending style to be used in the working directory. It enables end-of-line normalization without any content checks, **effectively setting the `text` attribute**.
  - *That is to say `eol` automatically set `text` attribute*.
  - Set to string value "crlf"  
This setting forces Git to normalize line endings for this file on checkin and convert them to CRLF when the file is checked out.
  - Set to string value "lf"  
This setting forces Git to normalize line endings to LF on checkin and prevents conversion to CRLF when the file is checked out.

3. If `eol` is put in `.gitattributes` file, it should be applied to specific file type. At the same time, it automatically marks the specific file type as `text` at the same time for LF normalization when checkin. If `eol` is set as `git config --global core.eol xxx`, then `eol` is set for all text files.

Refer to [gitattributes - defining attributes per path](http://git-scm.com/docs/gitattributes)

---
***EDIT 6***

Git attributes are specified in `.gitattributes` files. Line endings are controlled by `text` and `eol` attributes.

`text` attribute tells Git whether the file is `binary` (i.e. no `EOL` conversion should be performed while checking out and in) or `text` (perform `EOL` conversion, always convert to `LF` while checking in). Possible values are set (EOLs conversion is turned on), unset(EOLs conversion is turned off, default value) and `auto`(if the file is detected as binary, no conversion, otherwise EOLs conversion is performed).

`eol` attribute: if set implicitly sets `text` attribute and defines EOL to which the file should be converted while checking out.

Refer to [Line endings handling in SVN, Git and SubGit](http://blog.subgit.com/line-endings-handling-in-svn-git-and-subgit/)
***


#### Reference
1. [dealing-with-line-endings](https://help.github.com/articles/dealing-with-line-endings/#platform-all)
2. [line-endings-in-git](https://github.com/ninehills/blog.ninehills.info/blob/master/2012-5-line-endings-in-git.md)
3. [GitHub 第一坑换行符自动转换](http://blog.csdn.net/leonzhouwei/article/details/8933605#t0)
4. [fatal: LF would be replaced by CRLF in](http://stackoverflow.com/questions/15467507/trying-to-commit-git-files-but-getting-fatal-lf-would-be-replaced-by-crlf-in)
5. [gitattributes - defining attributes per path](http://git-scm.com/docs/gitattributes)

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