---
layout: post
title: Git Architecture
---

<span style="color:blue">The central is commits.</span>

# ABCs
![Git Architecure]({{site.baseurl}}assets/git2.png)

Basically three components:

1. working space, working directory, local directory
  * the real project files on hard disk
2. staging area, index
3. local repository, repository, history
  * the 2nd and 3rd ones are stored in `.git` folder

`git add` not only add files that are not yet known to Git, but also files that you have just modified. This is because Git takes content for next **commit not from your working copy, but from a special temporary area, called index**. This allows finer control over what is going to be committed. You can not only exclude some files from commit, you can exclude even certain pieces of files from commit (try `git add -i`). This helps developers stick to `atomic commits principle`.

How do we know what is the current state of things? What was the latest commit in the history? To answer that let’s look at Git refs (short for references). They are basically named references for Git commits. There are two major types of refs: `tags`, `heads`, and `remotes`.

1. `Tags` are fixed references that mark a specific point in history, for example v2.6.29.
2. On the contrary, `heads` are always moved to reflect the current position of project development.
3. Another special kind of refs are `remotes`. Whenever you run git fetch, it asks the remote repository, what heads and tags does it have, downloads missing objects (if any) and stores remote refs under refs/remotes prefix. The remote heads are displayed if you run `git branch -r`.


For the sake of simplicity, let’s forget about trees and blobs for now, and look at commits only.

![Git Architecure]({{site.baseurl}}assets/git3.png)

They are acutall three heads above called `old`, `master`, and `stable`. They represents the latest commits (heads) of the three **branches** respectively. `heads` here can be called `branches` as well. Open `\.git\refs\heads` directory, you will many heads there.

But to know what is happening right here, right now there is a special reference called `HEAD`. Normally HEAD points to one of the heads, so everything works out just fine. It serves two major purposes:

1. It tells Git which commit to take files from when you checkout
  * When you run `git checkout ref` it makes `HEAD` to the `ref` you’ve designated and extracts files from it. 
2. It tells Git where to put new commits when you commit.
  * When you run git commit it creates a new commit object, which becomes a child of current HEAD.

We can say **branch is pointer to the latest commits** while **HEAD is pointer to the currrent working branch pointer**. That is to say **HEAD is a 2nd level commit pointer**.
![Git Architecure]({{site.baseurl}}assets/git6.jpg)

![Git Architecure]({{site.baseurl}}assets/git4.png)

`HEAD` normally refers to a named branch (e.g. `master`). Meanwhile, each branch refers to a specific commit. Let’s look at a repo with three commits, one of them tagged, and with branch `master` checked out:

	       HEAD (refers to branch 'master')
		|
		v
    a---b---c  branch 'master' (refers to commit 'c')
	^
	|
      tag 'v2.0' (refers to commit 'b')


When a commit is created in this state, the branch is updated to refer to the new commit. Specifically, git commit creates a new commit `d`, whose parent is commit `c`, and then updates branch `master` to refer to new commit `d`. `HEAD` still refers to branch master and so indirectly now refers to commit `d`:

    $ edit; git add; git commit

		   HEAD (refers to branch 'master')
		    |
		    v
    a---b---c---d  branch 'master' (refers to commit 'd')
	^
	|
      tag 'v2.0' (refers to commit 'b')

It is sometimes useful to be able to checkout a commit that is not at the tip of any named branch, or even to create a new commit that is not referenced by a named branch. Let’s look at what happens when we checkout commit `b` (here we show two ways this may be done):

    $ git checkout v2.0  # or
    $ git checkout master^^

       HEAD (refers to commit 'b')
	|
	v
    a---b---c---d  branch 'master' (refers to commit 'd')
	^
	|
      tag 'v2.0' (refers to commit 'b')

Notice that regardless of which checkout command we use, HEAD now refers directly to commit `b`. This is known as being in **detached HEAD** state (which is also discussed below). It means simply that HEAD refers to a specific commit, as opposed to referring to a named branch. Let’s see what happens when we create a commit:

    $ edit; git add; git commit

	 HEAD (refers to commit 'e')
	  |
	  v
	  e
	 /
    a---b---c---d  branch 'master' (refers to commit 'd')
	^
	|
      tag 'v2.0' (refers to commit 'b')

There is now a new commit `e`, but it is referenced only by HEAD. We can of course add yet another commit in this state:

    $ edit; git add; git commit

	     HEAD (refers to commit 'f')
	      |
	      v
	  e---f
	 /
    a---b---c---d  branch 'master' (refers to commit 'd')
	^
	|
      tag 'v2.0' (refers to commit 'b')

In fact, we can perform all the normal Git operations. But, let’s look at what happens when we then checkout master:

    $ git checkout master

		   HEAD (refers to branch 'master')
	  e---f     |
	 /          v
    a---b---c---d  branch 'master' (refers to commit 'd')
	^
	|
      tag 'v2.0' (refers to commit 'b')

It is important to realize that at this point nothing refers to commit `f`. Eventually commit `f` (and by extension commit `e`) will be deleted by the routine Git garbage collection process, unless we create a reference before that happens. If we have not yet moved away from commit `f` (not `git checkout master`), any of these will create a reference to it:

    $ git checkout -b foo   <1>
    $ git branch foo        <2>
    $ git tag foo           <3>

1. creates a new branch `foo`, which refers to commit `f`, and then updates HEAD to refer to branch `foo`. In other words, we'll no longer be in detached HEAD state after this command.
2. similarly creates a new branch `foo`, which refers to commit f, but leaves HEAD detached.
3. creates a new tag foo, which refers to commit f, leaving HEAD detached.

If we have moved away from commit `f` (by `git checkout master` etc) without creating reference to it, then we must first recover its object name (typically by using `git reflog`), and then we can create a reference to it. For example, to see the last two commits to which HEAD referred, we can use either of these commands:

    $ git reflog -2 HEAD # or
    $ git log -g -2 HEAD

Here is another illustration:
![Git Architecure]({{site.baseurl}}assets/git5.png)

# Commands Comprehension
![Git Architecure]({{site.baseurl}}assets/git1.png)

The four commands above copy files between the working directory, the stage (also called the index), and the history (in the form of commits).

1. *git add files* copies files (at their current state) to the stage.
2. *git commit* saves a snapshot of the stage as a commit.
3. *git reset -- files* unstages files; that is, it copies files from the latest commit to the stage. Use this command to "undo" a git add files. You can also git reset to unstage everything.
4. *git checkout -- files* copies files from the stage to the working directory. Use this to throw away local changes.

You can use `git reset -p, git checkout -p, or git add -p` instead of (or in addition to) specifying particular files to interactively choose which hunks copy.

It is also possible to jump over the stage and check out files directly from the history or commit files without staging first.
![Git Architecure]({{site.baseurl}}assets/git7.svg)

1. `git commit -a` is equivalent to running git add on all filenames that existed in the latest commit, and then running git commit. But new files you have not told Git about are not affected.
2. `git commit files` creates a new commit containing the contents of the latest commit, plus a snapshot of `files` taken from the working directory. Additionally, `files` are copied to the stage.
3. `git checkout HEAD -- files` copies `files` from the latest commit to **both** the stage and the working directory.

A more detailed illustraion:
![Git Architecure]({{site.baseurl}}assets/git8.svg)

Commits are shown in green as 5-character IDs, and they point to their parents. Branches are shown in orange, and they point to particular commits. The current branch is identified by the special reference HEAD, which is "attached" to that branch. In this image, the five latest commits are shown, with ed489 being the most recent. `master` (the current branch) points to this commit, while `maint` (another branch) points to an ancestor of master's commit.

## Diff
There are various ways to look at differences between commits. Below are some common examples. Any of these commands can optionally take extra filename arguments that limit the differences to the named files.

![Git Architecure]({{site.baseurl}}assets/git9.svg)

## Commit
When you commit, git creates a new commit object using the files from the stage and sets the parent to the current commit. It then points the current branch to this new commit. In the image below, the current branch is master. Before the command was run, master pointed to ed489. Afterward, a new commit, f0cec, was created, with parent ed489, and then master was moved to the new commit.

![Git Architecure]({{site.baseurl}}assets/git10.svg)

This same process happens even when the current branch is an ancestor of another. Below, a commit occurs on branch maint, which was an ancestor of master, resulting in 1800b. Afterward, maint is no longer an ancestor of master. To join the two histories, a merge (or rebase) will be necessary.

![Git Architecure]({{site.baseurl}}assets/git11.svg)

Sometimes a mistake is made in a commit, but this is easy to correct with git commit --amend. When you use this command, git creates a new commit with the same parent as the current commit. (The old commit will be discarded if nothing else references it.)

![Git Architecure]({{site.baseurl}}assets/git12.svg)

A fourth case is committing with a detached HEAD, as explained later.

## Checkout

The checkout command is used to copy files from the history (or stage) to the working directory, and to optionally switch branches.

When a filename (and/or -p) is given, git copies those files from the given commit to the stage and the working directory. For example, git checkout HEAD~ foo.c copies the file foo.c from the commit called HEAD~ (the parent of the current commit) to the working directory, and also stages it. (If no commit name is given, files are copied from the stage.) Note that the current branch is not changed.

![Git Architecure]({{site.baseurl}}assets/git13.svg)

When a filename is not given but the reference is a (local) branch, HEAD is moved to that branch (that is, we "switch to" that branch), and then the stage and working directory are set to match the contents of that commit. Any file that exists in the new commit (a47c3 below) is copied; any file that exists in the old commit (ed489) but not in the new one is deleted; and any file that exists in neither is ignored.

![Git Architecure]({{site.baseurl}}assets/git14.svg)

When a filename is not given and the reference is not a (local) branch — say, it is a tag, a remote branch, a SHA-1 ID, or something like master~3 — we get an anonymous branch, called a detached HEAD. This is useful for jumping around the history. Say you want to compile version 1.6.6.1 of git. You can git checkout v1.6.6.1 (which is a tag, not a branch), compile, install, and then switch back to another branch, say git checkout master. However, committing works slightly differently with a detached HEAD; this is covered below.

![Git Architecure]({{site.baseurl}}assets/git15.svg)

## Committing with a Detached HEAD

When HEAD is detached, commits work like normal, except no named branch gets updated. (You can think of this as an anonymous branch.)

![Git Architecure]({{site.baseurl}}assets/git16.svg)

Once you check out something else, say master, the commit is (presumably) no longer referenced by anything else, and gets lost. Note that after the command, there is nothing referencing 2eecb.

![Git Architecure]({{site.baseurl}}assets/git17.svg)

If, on the other hand, you want to save this state, you can create a new named branch using `git checkout -b name`.

![Git Architecure]({{site.baseurl}}assets/git18.svg)

## Reset

The reset command moves the current branch to another position, and optionally updates the stage and the working directory. It also is used to copy files from the history to the stage without touching the working directory.

If a commit is given with no filenames, the current branch is moved to that commit, and then the stage is updated to match this commit. If --hard is given, the working directory is also updated. If --soft is given, neither is updated.

![Git Architecure]({{site.baseurl}}assets/git19.svg)

If a commit is not given, it defaults to HEAD. In this case, the branch is not moved, but the stage (and optionally the working directory, if --hard is given) are reset to the contents of the last commit.

![Git Architecure]({{site.baseurl}}assets/git20.svg)

If a filename (and/or -p) is given, then the command works similarly to checkout with a filename, except only the stage (and not the working directory) is updated. (You may also specify the commit from which to take files, rather than HEAD.)

![Git Architecure]({{site.baseurl}}assets/git21.svg)

## Merge

A merge creates a new commit that incorporates changes from other commits. Before merging, the stage must match the current commit. The trivial case is if the other commit is an ancestor of the current commit, in which case nothing is done. The next most simple is if the current commit is an ancestor of the other commit. This results in a fast-forward merge. The reference is simply moved, and then the new commit is checked out.

![Git Architecure]({{site.baseurl}}assets/git22.svg)

Otherwise, a "real" merge must occur. You can choose other strategies, but the default is to perform a "recursive" merge, which basically takes the current commit (ed489 below), the other commit (33104), and their common ancestor (b325c), and performs a three-way merge. The result is saved to the working directory and the stage, and then a commit occurs, with an extra parent (33104) for the new commit. 

![Git Architecure]({{site.baseurl}}assets/git23.svg)

## Cherry Pick

The cherry-pick command "copies" a commit, creating a new commit on the current branch with the same message and patch as another commit.

![Git Architecure]({{site.baseurl}}assets/git24.svg)

## Rebase

A rebase is an alternative to a merge for combining multiple branches. Whereas a merge creates a single commit with two parents, leaving a non-linear history, a rebase replays the commits from the current branch onto another, leaving a linear history. In essence, this is an automated way of performing several cherry-picks in a row.

![Git Architecure]({{site.baseurl}}assets/git25.svg)

The above command takes all the commits that exist in topic but not in master (namely 169a6 and 2c33a), replays them onto master, and then moves the branch head to the new tip. Note that the old commits will be garbage collected if they are no longer referenced.

To limit how far back to go, use the --onto option. The following command replays onto master the most recent commits on the current branch since 169a6 (exclusive), namely 2c33a.

![Git Architecure]({{site.baseurl}}assets/git26.svg)

There is also git rebase --interactive, which allows one to do more complicated things than simply replaying commits, namely dropping, reordering, modifying, and squashing commits. There is no obvious picture to draw for this; see git-rebase(1) for more details.

## Technical Notes

The contents of files are not actually stored in the index (.git/index) or in commit objects. Rather, each file is stored in the object database (.git/objects) as a blob, identified by its SHA-1 hash. The index file lists the filenames along with the identifier of the associated blob, as well as some other data. For commits, there is an additional data type, a tree, also identified by its hash. Trees correspond to directories in the working directory, and contain a list of trees and blobs corresponding to each filename within that directory. Each commit stores the identifier of its top-level tree, which in turn contains all of the blobs and other trees associated with that commit.

If you make a commit using a detached HEAD, the last commit really is referenced by something: the reflog for HEAD. However, this will expire after a while, so the commit will eventually be garbage collected, similar to commits discarded with git commit --amend or git rebase.

# A successful Git branching model

![Git Architecure]({{site.baseurl}}assets/git27.png)

# Reference

1. [A Visual Git Reference](http://marklodato.github.io/visual-git-guide/index-en.html)
2. [Git Is Your Friend not a Foe Vol. 3: Refs and Index](http://hades.name/blog/2010/01/28/git-your-friend-not-foe-vol-3-refs-and-index/)
3. [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)