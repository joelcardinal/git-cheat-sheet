# Git Cheat Sheet

The intention of this document is to show git terminal commands that are used to accomplish very specific tasks.  This document doesn't show git workflows used with a development branching strategy, if you are looking for that see https://github.com/joelcardinal/git-recipes. 

This is my personal curated list of commands I find my self frequently using and use it as a quick reference, it does not cover all commands.  For the sake of brevity, I describe what the command does in very simple terms which may be an over simplification of what git actually does.  I provide this as a start to introduce you to concepts but highly encourage you to take a deep dive into learning git, resource provided at bottom.

- [Getting Started](#user-content-getting-started)
- [Cheat Sheet](#user-content-cheat-sheet)
- [Resources](#user-content-resources)

## Getting Started

1. Install Git: Windows ([msysgit](http://msysgit.github.io/)) Mac ( [osx-installer](http://sourceforge.net/projects/git-osx-installer/) | [older dmg](https://code.google.com/p/git-osx-installer/downloads/list)) Or Source ([instructions](http://git-scm.com/book/en/Getting-Started-Installing-Git))

2. Create an account on the repository host (e.g. [bitbucket.org](bitbucket.org), [github.com](github.com)).

3. Create your public key ([instructions](https://help.github.com/articles/generating-ssh-keys)).

4. Add public key in your remote host environment, and if working with team, provide remote host admin with public key.

NOTE: There is an issue with capitalization with git on a Mac OS.  A Mac is typically setup with a drive format that is not case sensitive yet case preserving:

```
touch this.txt	# creates file this.txt
mv this.txt THIS.txt	# renames this.txt to THIS.txt —which is actually surprising
touch this.txt	# does NOT create a new file, THIS.txt is still present —so I assume it silently fails
```

The best thing to do on a Mac is create a partition that uses format case-sensitive.

http://stackoverflow.com/questions/17683458/how-do-i-commit-case-sensitive-only-filename-changes-in-git
http://stackoverflow.com/questions/8904327/case-sensitivity-in-git


## Cheat Sheet


### How to not screw up

Git can seem difficult and complicated when you are starting out (even after using it awhile), it's also easy to screw up the git history.

Don't use destructive commands like rebase and reset if you can help it.  If you want to use them, you need to take a deep dive into learning and reading through git documentation.

Here are a few things that will help you stay out of trouble:

Don't use rebase with commits that exist on remote
Don't use reset that affect commits that exist on remote
Know what branch you are on (```git status```)
Know if you have uncommitted work on current branch (```git status```)
When you want to merge:
 - Make sure your reference to remote is up-to-date (```git status && git checkout master && git fetch origin && git fetch --prune```)
 - Your active branch needs to be the branch that you intend to receive the commits from some other branch.
 - Make sure you use the correct branch names
 - If you are merging in a remote branch make sure you reference it with origin (```git merge origin/REMOTE-BRANCH-NAME```)
Double-check above
Triple-check above


### Start using git

Initialize git in a directory not currently using git.
This creates a .git directory and starts listening for changes.

```git init```

To remove git from a project you just need to remove the git folder.

```rm -rf .git/```

To be able to push up to a remote repository, first create the new repo in your remote host environment (e.g. github, bitbucket).
Once provided a repo URL, run the following command in your local git repo.

```git remote add origin GIT-LOCATION-PROVIDED-BY-REMOTE-HOST```

Then push the repo up to remote

```git push -u origin master```

You can check that the remote repo was tied to the local repo correctly by running the following command

```git remote -v```

If a repo already exists on a remote source and you want to copy it locally run

```git clone GIT-LOCATION-PROVIDED-BY-REMOTE-HOST```

If for some reason the remote location changes you can point your repo somewhere else

```git remote set-url origin GIT-LOCATION-PROVIDED-BY-REMOTE-HOST```


### Making a commit

Check which branch you are on and if there are any modified files.  This command will become your best friend, if you use it between issuing other git commands you will keep yourself out of trouble because you will see exactly what happens to files after each command.

```git status```

Stage a single file for commit

```git add FILE-PATH-HERE```

Stage ALL modified files for commit

```git add -A```

Commit staged changes and add a short message to the commit

```git commit -m "Quick message of what commit is about"```

Above we used the -m option which allows us to add a short commit description inline.  If we left off the -m option git would open your default terminal editor where it would expect you to write your commit message and save the message.  There is a global git config you can use to set your preferred editor.


### Create and Switch Branch

Create new branch based off current branch, but don't switch to it

```git branch BRANCH-NAME```

Create new branch based off current branch and switch to it

```git checkout -b BRANCH-NAME```

IMPORTANT:  When you want to make a local copy of a remote branch do the following:


```git checkout -b BRANCH-NAME origin/BRANCH-NAME```

Above we are telling git that we want to switch to a new branch called BRANCH-NAME and we want the commit history to be based off of the remote BRANCH-NAME branch.

If we had not included the ```origin/BRANCH-NAME```, then our local branch BRANCH-NAME would have been based off whatever the current active branch was.

Additionally, using ```origin/BRANCH-NAME``` tells git what remote branch to track, so you can use the command ```git pull``` with confidence that it will merge in the correct remote branch.  See the discussion on merging in this document for more information.

Switch to last used branch (good way to toggle between two branches)

```git checkout -```

Essentially "git checkout all" when you have modified files and want to discard all changes, after this command execute git status to make sure it had the affect you intended.

```git checkout -- .```


### List branches

Show list of local and remote branches

```git branch -a```


### Delete branch

To delete a local branch (first checkout master to avoid erroneous git warnings)

```git branch -d the_local_branch```

If you perform the above on a local branch who's commit history has not be merged into any other existing branch, git will throw a warning and not delete the branch.  If you are certain you don't need any of the commit history in the branch you are trying to delete used capital -D option to force the deletion.

```git branch -D the_local_branch```

To remove a remote branch (if you know what you are doing!)

```git push origin :the_remote_branch```

Prevent showing branches that have already been deleted

```git fetch --prune```


### Show local commit history

To show history use the following to open a Unix pager of info

```git log```

q = quits
j,k, Ctrl+F, Ctrl+B,arrow up/down = move
/search-term = search
While in search mode you can use...
n,N = jump back and forth through query results

Show history on one line

```git log --oneline```

To show history with change

```git log -p```

To show branching tree of history

```git log --graph```

Show history on one line with graph

```git log --oneline --graph```

Show last 3

```git log -3```

Show between now and yesterday

```git log --after="yesterday"```

Show between now and yesterday

```git log --after="2 weeks ago"```

Show from certain date

```git log --after="3/10/16"```

Show between last year and two weeks ago

```git log --after="last year" --before="two weeks ago"```

NOTE: Can also use "since/until" instead of "before/after"

Show commits by certain person

```git log -i --author="Joel"```

NOTE: Use -i to ignore case

Options are regex

```git log --author="Joel\|John"```

Filter by message words

```git log --grep="deleted"```

Filter by code change where Math was edited

```git log -p -S"Math"```

Same as above but regex

```git log -p -GMath\|Random```

Show commits that were not merges

```git log --no-merges```

Show commit diff betweenn branches

```git log master..BRANCH-NAME```

Show commits of specific files

```git log PATH-TO-FILE PATH-TO-OTHER-FILE```

Mash up filter

```git log -3 -i --author="Joel" README.md```


### Show diff

Shows changes, by default between last commit and working directory

```git diff```

Diff different references, last commit and staged

```git diff --cached```

Diff working and stage against last commit

```git diff HEAD```

Diff current branch against another branch

```git diff BRANCH-NAME```

Diff current branch against remote branch

```git diff --raw HEAD...origin/[BRANCH-NAME]```


### See history on remote

```git log origin/seewhy_test```

NOTE: before executing above you should ```git checkout master && git fetch origin && git fetch --prune```


### See who edited code

See who last changed each line of code

```git blame PATH-TO-FILE```

After running above you can get better context for the changes by copying the commit hash then run the following.

```git log HASH -p```


### Show the commits that are in your current branch, but not origin -- i.e., whether you're ahead of origin and by which commits.

```git rev-list origin..HEAD```


### Renaming a local branch.

```git branch -m old_branch_name new_branch_name```


### Stash your work and apply later

This is one of my favorite features of git, it allows you to store away changes that have not been committed. It also resets the head to the last commit, as if you never made those changes.

This feature is great for those times when you have multiple solutions to a problem and you want to try both without committing.  It's also a great way to save work when your daily priorities change and you have modified files but donn't want to make a commit yet as you are not done, but need to switch to another branch to work on something else.

Here we are saving an uncommitted but staged changes and adding a message:

```git stash save "BRANCH-NAME - Partial work for issue X"```

NOTE: I suggest including the branch name that the changes were done in as a reminder of the branch you will typically intend to apply the stash to at a later time.

Here we list our stashes:

```git stash list```

Here we bring back and apply our stash. I believe you can edit the stash ID so that you apply the stash you want:

```git stash apply stash@{0}```

Note: It's possible that if you've changed the branch since you stashed the work you may encounter a merge conflict when you apply it.

The stash will still be there after you apply it, to delete the stash:

```git stash drop stash@{0}```

In the above example ```git stash save "Partial work for issue X"``` you'll note it only stashed staged changes, below are some options allowing stash used on changed files in different states.

stash only unstaged changes to tracked files

```git stash --keep-index```

stash any changes to tracked files

```git stash```

stash untracked and tracked files

```git stash --include-untracked```

stash ignored, untracked, and tracked files

```git stash --all```

TIP: Applying your stash does not delete the stash.  If it's no longer needed after applying, delete it or you'll just be confused later and wonder if you applied it or not.


### File or Commit History:

Shows commits by line

```git blame filePath```

Shows history of file

```git log -p --follow filePath```

Shows diff of commit

```git show commitHash```


### Undo the last commit:

Use soft if you want to undo the change, but keep your changes in the active workspace

```git reset --soft HEAD~1```

Use hard if you want to undo the change and get rid of the changes in the active workspace

```git reset --hard HEAD~1```


### Copy a commit to another branch:

First you would checkout the branch you want to work with:

```git checkout master```

Then you would look at the commits and find the commit you wanted

```git log```

Then copy the commit serial number. Then checkout the branch you want to put it in:

```git checkout receiving_branch```

Then use the following statement where the serial number is pasted after cherry-pick:

```git cherry-pick 103298764wek23089476iwei28934```

Note: cherry-pick does not delete the commit from the original branch.


### Undo a specific commit somewhere in the history:

First look at the commits and find the commit you want to undo

```git log```

Then copy the commit serial hash and use the following command, where the hash of the commit is used after revert:

```git revert 103298764wek23089476iwei28934```

Note: this does not get rid of the commit you didn't want--it creates a new commit at the end of the branches history that performs the opposite action of that commit. THIS IS VERY IMPORTANT TO UNDERSTAND. If you ever revert your commit history to a period earlier than this "undo" commit, the "undo" won't be there. Also, if you perform this, commits that depended on the unwanted commit will also be undone.

In some cases it may be best to just create a branch that starts at the commit prior to the unwanted commit, then cherry-pick over commits that happened later that you do want. There are always a few different ways to skin a cat in git.


### Undo a merge:

Undoing a merge is similar to the above, but we need to provide git with more information.

```git revert 803298764wek23089476iwei28934 -m 1```

In the example above, the long sha number is the sha number provided in the log when you merged. The 1 stands for the first commit in the branch you merged in, a 2 would stand for the second commit made in the branch you merged in. In this example, if you had more than one commit, you would not be un-committing the entire merge, just that particular commit in the branch you merged in. Understandably, this method is only practical if you merged in a branch which had few commits.

Additionally, after doing this you will not be able to merge back in any ancestors of this merge branch point


### Add Tag

A tag just adds a name to the last commit, typically used to mark a point in history, like a release.

Add tag simple tag with no description

```git tag v1.0.0```

Create tag with message

```git tag -a v2.0.1 -m "This was a hotffix"```

Oops, I mis-spelt hotfix, so let's delete it using the name as reference:

```git tag -d v2.0.1```

If you want a tag with a long message, like release info

```git tag -a v2.0.1 -m```

Push tags to remote

```git push --tags```

Show tags

```git tag```


### You can even use git to find in what file and line a search query can be found.

```git grep -n somethingtosearchfor```


### Index / Staging

Other than commits, you have two other states of your changes, staged and un-staged. This wiki mostly discusses committing changes with:

```git commit -a```

The above actually commits BOTH staged and un-staged changes. So lets go through a scenario. You are in Studio or an editor of your choice. You make a change to a file and then you save the file in the editor.
Now you have an un-staged commit. Let's now stage the commit:

```git stage file_name```

You could also use the command below, however doing so also adds any untracked files into the git history:

```git add file_name```

If you stage something and then want to un-stage it on a certain file, run:

```git reset HEAD file_name```

To un-stage all files use:

```git reset HEAD```

The advantage of staging before committing is that it gives you more freedom to manage what gets committed and when. Note, staging is not like creating a commit, if you make several changes and stage each change, their will not be a record for each staging point. If you now run…:

```git commit```

…only the files you staged will be committed.


### Merge Conflicts

On occasion when you merge, you may encounter a conflict.

To abort the merge as soon as the conflict is introduced:

```git merge --abort```

To continue with a merge:

```git mergetool```

The git mergetool will open a gui where you can resolve your merge conflict. The gui will depend on what you have installed. For developers on Mac with Apple development tools installed it is likely to be opendiff/FileMerge. The following gui description assumes you are using FileMerge. The top two panes are your files: remote, and local. You can verify which is which by looking at the file name at top. The bottom pane allows you to manually make changes. You can use the gui by clicking the conflict error arrows and selecting which version to use. You can change the arrows direction by using the drop-down menu in the bottom right.
When you are done change the arrows or making manual edits, save the change (control + S). A window will appear as to what format to save the change as, it doesn't matter which you select. This creates a file of the merge which needs to be deleted.

We can use terminal commands to delete these files. They will have a unique extension, so we can use a find and remove by extension:

Find all files in current directory with the extension “this” and delete them.

```find . "*.this" | rm```

Now we can commit the merge changes:

```git commit -am "I fixed a merge conflict"```


### Find bug

Find bad code commit is a process of serveral commands

Get into bisect mode

```git bisect start```

Mark current commit bad (or checkout at bad commit)

```git bisect bad```

Find last known good hash

```git log --oneline```

Copy last known good hash, then run the following to mark as good

```git bisect good HASH```

Git will checkout at commit between these points, test and mark good or bad, keep doing so

```
git bisect bad
...
git bisect good
```

After marking good, git will give you the bad commit hash where issue was first introduced
But you need to now get out of bisect mode by running the following

```git bisect reset```


### Interactive rebase

IMPORTANT: Using rebase is destructive to git history, as you are changing history and commit hashes, don't use rebase with commits you've already pushed up to remote!

git rebase interactive is a pretty damn cool feature where you can manipulate your local commits before pushing up a branch.
For example, it is frequently used to squash all commits from an issue branch into one single commit; to tidy up the history.
You can also delete commits, reword messages and more.

Let's say we have a master branch and an issue branch.  And let's say that the issue branch is two commits ahead of master.

```git rebase -i```

git will open your shell editor with something like this:

```
pick 218bdd8 added 8
pick 9c63b7c added 9

# Rebase a2fb79b..9c63b7c onto a2fb79b (2 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

Fortunately, git is telling you everything you need to know.  Unfortunately, it doesn't explain what you need to do.
Notice above "pick" is written before the hash, you need to replace "pick" with whatever command you want to use.
I'll show a few examples.

```
pick 218bdd8 added 8
squash 9c63b7c added 9
```

Above will replace these commits with one commit that has the message of both commits.

```
r 218bdd8 added 8 and 9
f 9c63b7c added 9

```

Above will replace these commits with one commit that has the message of only the first commit which we've amended.

```
e 218bdd8 added 8
r 9c63b7c added 9 and it rocks
```

By replacing the command "pick" with the command "edit" (you can also use "e"), you can tell git rebase to stop after applying that commit,
so that you can edit the files and/or the commit message, amend the commit, and continue rebasing.
When you are done editing and/or resolving conflicts you can continue with git rebase --continue.

Before trying to use rebase for the first time, do yourself a favor and play around in a separate test repo to get the hang of it.


## Resources

### Basics and Commands

[https://github.com/joelcardinal/git-cheat-sheet](https://github.com/joelcardinal/git-cheat-sheet)
[https://services.github.com/kit/downloads/github-git-cheat-sheet.pdf](https://services.github.com/kit/downloads/github-git-cheat-sheet.pdf)
[http://ndpsoftware.com/git-cheatsheet.html](http://ndpsoftware.com/git-cheatsheet.html)
[try.github.com](try.github.com)
[BitBucket Docs](https://confluence.atlassian.com/display/BITBUCKET/Bitbucket+Documentation+Home)
[GitHub Docs](https://help.github.com/)
[http://progit.org/book/](http://progit.org/book/)
[http://hoth.entp.com/output/git_for_designers.html](http://hoth.entp.com/output/git_for_designers.html)
[http://schacon.github.com/git/git.html](http://schacon.github.com/git/git.html)
[http://schacon.github.com/git/user-manual.html](http://schacon.github.com/git/user-manual.html)
[http://git-scm.com/documentation](http://git-scm.com/documentation)
[http://gitref.org/](http://gitref.org/)
[http://www-cs-students.stanford.edu/~blynn/gitmagic/ch02.html](http://www-cs-students.stanford.edu/~blynn/gitmagic/ch02.html)
[http://cheat.errtheblog.com/s/git](http://cheat.errtheblog.com/s/git)
[http://progit.org/2011/07/11/reset.html](http://progit.org/2011/07/11/reset.html)
[http://caiustheory.com/adding-a-remote-to-existing-git-repo](http://caiustheory.com/adding-a-remote-to-existing-git-repo)
[http://ariejan.net/2009/10/26/how-to-create-and-apply-a-patch-with-git](http://ariejan.net/2009/10/26/how-to-create-and-apply-a-patch-with-git)
[http://ricardofilipe.com/projects/firstaidgit/#/](http://ricardofilipe.com/projects/firstaidgit/#/)

### Branch Management

[http://git-scm.com/book/en/Git-Branching](http://git-scm.com/book/en/Git-Branching)
[https://www.atlassian.com/git/workflows](https://www.atlassian.com/git/workflows)
[http://nvie.com/posts/a-successful-git-branching-model/](http://nvie.com/posts/a-successful-git-branching-model/)
[http://drewfradette.ca/a-simpler-successful-git-branching-model/](http://drewfradette.ca/a-simpler-successful-git-branching-model/)

### Connect Git Commits to Jira

[https://confluence.atlassian.com/display/BITBUCKET/Linking+Bitbucket+and+GitHub+accounts+to+JIRA](https://confluence.atlassian.com/display/BITBUCKET/Linking+Bitbucket+and+GitHub+accounts+to+JIRA)

### Self hosting repository

[https://www.atlassian.com/software/stash](https://www.atlassian.com/software/stash)
[http://www.gitorious.com/local_install/](http://www.gitorious.com/local_install/)
[www.gitlab.com](www.gitlab.com)

### Git Management Tools
SourceTree - If the terminal isn't your thing.</p>

[https://www.atlassian.com/software/sourcetree/overview](https://www.atlassian.com/software/sourcetree/overview)
