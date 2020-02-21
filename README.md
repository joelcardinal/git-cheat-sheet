# Git Cheat Sheet

The intention of this document is to show git terminal commands that are used to accomplish very specific tasks.  This document doesn't show git workflows used with a development branching strategy, if you are looking for that see https://github.com/joelcardinal/git-recipes. 

This is my personal curated list of commands I find my self frequently using and use it as a quick reference, it does not cover all commands.  For the sake of brevity, I describe what the command does in very simple terms which may be an over simplification of what git actually does.  I provide this as a start to introduce you to concepts but highly encourage you to take a deep dive into learning git, resource provided at bottom.

- [Getting Started](#user-content-getting-started)
- [Understanding git](#user-content-understanding-git)
- [Cheat Sheet](#user-content-cheat-sheet)
- [Resources](#user-content-resources)

## Getting Started

1. Install Git:

 - Windows: ([msysgit](http://msysgit.github.io/))

 - Mac:
	- Use HomeBrew (`brew install git`, `brew upgrade git`)
	- Use MacPorts (port install git)
	- Source ([instructions](http://git-scm.com/book/en/Getting-Started-Installing-Git))

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

## Understanding git



### Git Demystified

There were a few things that made it hard for me to conceptually understand git when I first started using git:

1. What is a commit really?
2. What is this stuff about staging and remote?

We'll cover commits in a separate section below, if you've never used version control, it might help read that section first, but it's imperative you understand how git orchestrates file changes -- even if you only have a vague idea of what a commit or branch means.

I'm going to use an oversimplification to describe the 4 "pillars" of git:

**Workspace**

Consider the workspace the actual files you edit, and when you modify them this change is said to be a change in your workspace.

**Index**

When you stage a commit you are telling git that you intend on committing the change and making it permanent, it's technically not a separate physical version of the file.

**Local Repository**

After you commit a staged change it is permanently recorded in your local repository.

**Remote Repository**

The remote repository is the git history in your remote git host, typically in a service like GitHub, Bitbucket.  Technically the remote repository is no different than your local repository except for the commit and branch history -- which are not in sync.   Because they don't stay in sync, that's why you have to push up a branch.  And when you push up a branch to remote, it is really pushing up the commit history from your local repository.

There are some good answers and visuals found here: https://goo.gl/rtXYA2

Okay, so the first two should sorta make sense, the workspace is considered your files and any un-staged changes.  The index is essentially the staged changes, stuff that is intended to be committed.

It's important to clarify the latter two because it's the whole crux behind git's ability to allow people to collaborate, but it can also be the most confusing part.

For this example let's imagine we clone a remote repo, which means we pull down all existing history typically from an external server hosted by a service (e.g. GitHub, BitBucket).  For the sake of simplicity let's assume that the repo only has one branch, the default, "master".  So now we have a copy of all the files and history.

The files you'll see on your hard drive, the history is tucked away in the bowels of the hidden .git directory within the main parent folder that was created when we cloned the remote repo.

When we execute a terminal command (```git branch -a```) to show the local and remote branches it will state:

```
*master
remotes/origin/master
```

What you are seeing in the first line is a reference to the master branch in your local repository and on the second line a reference to the master branch in the repo.

Since we just cloned this, for the moment our local master and remote master have the same commit history.

Hang onto your hat, this is where it gets weird...

If someone else pushes up a commit to the remote master branch on GitHub, will my reference to the remote master have that commit information?

If you answered "yes", you are wrong.

Our local master and our reference to remote master still have the same commit history.

Notice how I explicitly state that it's a "reference" to the remote master?  There's no background process keeping our remote reference in sync.

To bring our reference to remote up-to-date we actually have to perform a git command:

```git fetch origin```

Now if we were to run a command (```git log origin/master```) to see the history from our remote reference we would see that the new commit by another developer is there.

---

**TIP:** If you are on git version 2.5 or higher you can use the following chained commands to quickly see all branches of which you are behind compared to origin, as fetch origin only shows changes from the last time you ran it; but you may not have merged in the changes yet.

```git fetch origin && git for-each-ref --format="%(refname:short) %(push:track)" refs/heads | grep behind```

Add this alias to your .bash_profile file, command is simply "behind":

```alias behind='git fetch origin && git for-each-ref --format="%(refname:short) %(push:track)" refs/heads | grep behind'```

And remember it won't take affect until you reopen the terminal or run this command to reset it:

```source ~/.bash_profile```

---

Next question, after we fetched origin, does our local master branch have the new commit?

If you answered "yes", you are wrong.

If we performed a command (```git status```) to see our local master branches status it would say our master branch is 1 commit behind remote master branch.

We have to issue another command to merge in the new commit into our local master branch.

```git merge origin/master```

Above means merge remote master branch into my local master branch.  The "origin/master" part is where we state the remote master branch, since we only have one local branch, which is master, it must be our current active branch.  When you merge git assumes you want to merge the provided branch into your current active branch.

It's really important to keep this mental model of how git works in your mind because it will help you remember to keep your remote references up-to-date and merge in any changes from remote.

I think conceptually it would help folks if we added a 5th "pillar" of git, right before "remote repository", which would be "reference to remote repository".


### Commits

Every commit has a unique SHA-1 (sometimes called serial, hash...etc.), example: 7635353d1b86ef164d80501559a0189f9216a92c
Any change to a commit changes the SHA-1
Any new SHA-1 is something new despite same content

A branch is just a pointer to a commit that is the branch head.  In the following we'll look at how git does it's magic while skipping the actual commands, let's first get a feel for commits and branches.

Here is master branch with three commits:

```
0-0-0

```

Let's number them so we can better understand:

```
1A-2A-3A

```

Here is branch my-new-branch, which branched off master, and has two unique commits:

```
1A-2A-3A
       \
	    1B-2B
```

Now if we merge my-new-branch into master git will by default use the "fast-forward" where will just move the commit pointer:

```
1A-2A-3A-1B-2B

```

However, if you want to maintain a history of the branching you can pass argument --no-ff to force git to make a new merge commit, 4A:

```
1A-2A-3A------4A
       \     /
	    1B-2B
```
	  
Let's go back to our previous state where master has three commits:

```
1A-2A-3A

```

Okay and we'll branch off again, my-new-branch, and make two commits:

```
1A-2A-3A
       \
	    1B-2B
```		
		
And now someone has added two commits to master:

```
1A-2A-3A-4A-5A
       \
	    1B-2B
```

If we merge my-new-branch into master, git will create a new merge commit:

```
1A-2A-3A-4A-5A-6A
       \      /
	    1B-2B
```

Okay, let's again go back to a previous state where we branch off master, make two commits, and master has two additional commits:

```
1A-2A-3A-4A-5A
       \
	    1B-2B
```

I'm going to add commit SHA-1 info to help illustrate what we are about to do:

```
1A (sdf786a) - 2A (h98f7ui) - 3A (dfgd78s) - 4A (hj987dw) - 5A (6bvn6v6)
									       \
										    1B (76b5vn6) - 2B (8k5c0b5)
```
		
Now we'll use rebase to add commits from master to my-new-branch:
http://onlywei.github.io/explain-git-with-d3/#rebase

```
1A (sdf786a) - 2A (h98f7ui) - 3A (dfgd78s) - 4A (hj987dw) - 5A (6bvn6v6) - 6A (986dfgp) - 7A (123n2z1)

```

Above notice that rebase is a "destructive" method -- we are changing history.
Commits 1B and 2B are orphaned and new commits 6A and 7A are used on master.
Don't confuse this with "fast-forward", where the pointer simply moves, with rebase you are changing history and creating new commits.

Why WOULD you want to do this?  So that you keep a "clean" history, free of branching speghetti.

Why WOULDN'T you want to do this?  If you do this incorrectly YOU WILL F-UP YOUR HISTORY AND EVERYONE ELSE'S!!!

!!!NEVER PUSH UP COMMITS YOU INTEND TO REBASE!!!

So let's looks at what you should never do, so that you can avoid it.
We'll go back to the moment before we used rebase:

```
1A (sdf786a) - 2A (h98f7ui) - 3A (dfgd78s) - 4A (hj987dw) - 5A (6bvn6v6) <-- master
									       \
										    1B (76b5vn6) - 2B (8k5c0b5) <-- my-new-branch

```
											
You DO NOT want to push up my-new-branch...

```
1A (sdf786a) - 2A (h98f7ui) - 3A (dfgd78s) 
									       \
										    1B (76b5vn6) - 2B (8k5c0b5) <-- my-new-branch

```

Have someone else work on it...

```
1A (sdf786a) - 2A (h98f7ui) - 3A (dfgd78s) 
									       \
										    1B (76b5vn6) - 2B (8k5c0b5) - 3B (234k345) <-- my-new-branch with commit 3B by someone else
```

...then you rebase your my-new-branch...

```
(before rebase)
1A (sdf786a) - 2A (h98f7ui) - 3A (dfgd78s) - 4A (hj987dw) - 5A (6bvn6v6) <-- master
									       \
										    1B (76b5vn6) - 2B (8k5c0b5) <-- my-new-branch

(after rebase)										
1A (sdf786a) - 2A (h98f7ui) - 3A (dfgd78s) - 4A (hj987dw) - 5A (6bvn6v6) - 6A (986dfgp) - 7A (123n2z1)

```

...and you push up your rebased my-new-branch...

...and when your teammate trys to pull or merge in your change git will bork.


###HEAD###

You'll see "HEAD" in documentation and commands.

HEAD normally refers to the currently checked out branch (e.g. master), and each branch typically refers to the last commit on that branch.

When you checkout a tag or specific commit that is not the tip, this is known as a "detached HEAD".

https://git-scm.com/docs/git-checkout#_detached_head
http://stackoverflow.com/questions/2304087/what-is-head-in-git


### Make git ignore files ###

Sometimes you may not want git to track a certain file, for example it may have sensitive information like a configuration file that holds passwords or api keys that you don't want to exposed.  The best way to manage which files or directories git should not track is by listing these files and directories in a file named ".gitignore".  This specifically named file has special significance to git.  Note that this file name starts with a period, typically these files are hidden from view by operating systems, you can view these hidden files in the terminal by using the command ```ls -a```.

Add your list of files and directories line deliminated, and save it in the root directory of your repo.

It is best to create this file before your first git commit in the repo, but you can add it or modify it at any time.  However anything committed to git will be permanent in git history, so this means that if you add an exclusion to the .gitignore it won't track it from that time on, but if you committed the file before it was in .gitignore those existing commits don't go away.

For exmaple, let's say when you made your first commit you had a file called "config.json" and it was not listed in the .gitignore, the file config.json is now in permanent history.  If you then add "config.json" to your .gitignore file and commit the change, any further changes to the "config.json" will not be tracked, but it will still exist in your repo history, and remote and other devs will have the file in their repo too because of that first commit.

Given the above, it is sometimes useful to commit a files structure, especially for JSON files, but then add it to your .gitignore so that you can populate the JSON file's values with your sensitive information and it will not get committed.

At this point it should make sense that the .gitignore is handled like any other file, you create and commit the file, and it will get shared with others using your repo having the same affect on their files and directories.

Below is an example of what the contents of a .gitignore might contain.  Note that "*" wildcards can be used which stand for any number of characters.  Directories don't require a trailing slash, below "logs" is a directory.

```
.DS_Store
Thumbs.db
config.json
.idea
*.sublime-workspace
node_modules
npm-debug.log
.tern
*.tern
.tern *
.settings
*.settings
logs
```

Use an exclimation as a NOT, to "un-ignore" specific files/directories

```
*.json
!spec/*.json
```

https://stackoverflow.com/questions/4621072/git-ignore-all-files-of-a-certain-type-except-those-in-a-specific-subfolder

If you want "empty" directories to exist in a git repo but never files in it, cd to the directory, then create a .gitignore with this in it:

```
*
*/
!.gitignore
```

https://stackoverflow.com/questions/4250063/how-to-gitignore-all-files-folder-in-a-folder-but-not-the-folder-itself#answer-5581995

## Cheat Sheet


### How to not screw up

Git can seem difficult and complicated when you are starting out (even after using it awhile), it's also easy to screw up the git history.

Don't use destructive commands like rebase and reset if you can help it.  If you want to use them, you need to take a deep dive into learning and reading through git documentation.

Here are a few things that will help you stay out of trouble:

* Don't use rebase with commits that exist on remote
* Don't use reset that affect commits that exist on remote
* Know what branch you are on (```git status```)
* Know if you have uncommitted work on current branch (```git status```)
* When you want to merge:
 * Make sure your reference to remote is up-to-date (```git status && git checkout master && git fetch origin && git fetch --prune```)
 * Your active branch needs to be the branch that you intend to receive the commits from some other branch.
 * Make sure you use the correct branch names
 * If you are merging in a remote branch make sure you reference it with origin (```git merge origin/REMOTE-BRANCH-NAME```)
* Double-check above
* Triple-check above


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

NOTE: For any file change, even rename or delete, you need to "add" this change, which especially for a file or directory delete can seem counterintuitive.

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

If you try to delete a remote branch that is named the same as a tag, git will throw an error, you'll need to use the following method to delete the branch; supply a more complete path.

```git push origin :refs/heads/the_remote_branch```

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

Show commit diff between branches

```git log master..BRANCH-NAME```

Show commits of specific files

```git log PATH-TO-FILE PATH-TO-OTHER-FILE```

Mash up filter

```git log -3 -i --author="Joel" README.md```

Show files changed in given commit

```git diff-tree --no-commit-id --name-only -r COMMITHASH```

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

Below commit message diffs require messages formatted as "BUGNUMBER_Description-Here"

Show partial commit message of diff between two branches

```git log NEWERBRANCHNAME ^OLDERBRANCHNAME --oneline | grep _ | awk '{print $2}' | grep _ | sort -u```

Show partial commit messages of diff between old hash and current branch HEAD

```git log OLDCOMMITHASH..HEAD  --oneline | grep _ | awk '{print $2}' | grep _ | sort -u```


### Show Merge Diff

Inspect a merge commit using log or other command.  Note the two short commit hashes listed in the commit message.  Reverse the order of the two then run the following.  The two dots in between are required.

```git diff HASH#2..HASH#1```

https://stackoverflow.com/questions/5072693/how-to-git-show-a-merge-commit-with-combined-diff-output-even-when-every-chang


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

This feature is great for those times when you have multiple solutions to a problem and you want to try both without committing.  It's also a great way to save work when your daily priorities change and you have modified files but don't want to make a commit yet as you are not done, but need to switch to another branch to work on something else.

Here we are saving an uncommitted but staged changes and adding a message:

```git stash save "BRANCH-NAME - Partial work for issue X"```

NOTE: I suggest including the branch name that the changes were done in as a reminder of the branch you will typically intend to apply the stash to at a later time.

Here we list our stashes, notice that stash uses a numerical index it stores different stashes under:

```git stash list```

Here we bring back and apply our stash. You'll need to use the correct stash index number so that you apply the stash you want, in this example it is 0:

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


### See What Branch Commit is in

Only list (local) branches which contain the specified commit.

```git branch --contains commitHash```

Note: if the commit is on a remote tracking branch, add the -a option.

```git branch -a --contains commitHash```

Lists remote tracking branches as well, that is "local branches that have a direct relationship to a remote branch".

```git branch -r --contains commitHash```

Commands above only shows which branches contain that exact commit. If you want to know which branches contain an "equivalent" commit (i.e. which branches have cherry-picked that commit) use ```git cherry```.

```git cherry -v```

The commits marked with a plus sign are those not in the upstream branch, more: https://www.kernel.org/pub/software/scm/git/docs/git-cherry.html


### See if Branch is in another Branch

Lists branches merged into master
```git branch --merged master```

Lists branches merged into HEAD (i.e. tip of current branch)
```git branch --merged```

Lists branches that have not been merged
```git branch --no-merged``` 

By default this applies to only the local branches. The -a flag will show both local and remote branches, and the -r flag shows only the remote branches.


### File or Commit History:

Shows commits by line

```git blame filePath```

Shows history of file

```git log -p --follow filePath```

Shows diff of commit

```git show commitHash```

### Commits in one branch but not another

```git log --no-merges oldbranch ^newbranch```

https://stackoverflow.com/questions/1710894/using-git-show-all-commits-that-are-in-one-branch-but-not-the-others

### Unstage all files and remove modified/untracked files

```git reset --hard HEAD```

To remove untracked files and directories

```git clean -fd```

### Undo the last commit:

Use soft if you want to undo all staged changes, but want to un-stage and keep those changes in the active workspace

```git reset --soft HEAD~1```

Use hard if you want to un-stage all changes and get rid of the un-staged changes in the active workspace

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

If you merged a branch in but haven't yet pushed it up to remote, and don't have additional commits that you added after the merge:
	
```git reset --merge ORIG_HEAD```
	
Above, ```ORIG_HEAD``` is a literal reference that will point to the original commit from before the merge, it is not a stand-in for a branch name.

(the --merge option has nothing to do with the merge, it's just like git reset --hard ORIG_HEAD but safer since it doesn't touch uncommitted changes)

If you merged a branch in but haven't yet pushed it up to remote, and unfortunately created additional commits that you added after the merge inspect the history:

```git reflog```

Above will print something like this:

```
23gg032 HEAD@{0}: commit: Fixed js
fbb0c0f HEAD@{1}: commit (merge): Merge branch 'master' into my-branch
43b6032 HEAD@{2}: checkout: moving from master to my-branch
```

Given the above, let's assume that you don't mind losing the work in commit 23gg032, and you don't want the merge in your branch, simply tell git to go back in time:

```git reset --hard 43b6032```

If you didn't want to lose the work in 23gg032 and those changes were not dependant on anything in the merge, you could probably create a new branch based off this borked one, let's call this new one Borky2 and let's call the original borked branch Borky1.  Now that you have a copy, go back and checkout Borky1, execute ```git reflog``` as shown above and reset to a commit before the merge.  Now cherry-pick 23gg032 into this Borky1 branch.  Then you can delete Borky2 branch.  Theoretically that should work but I haven't tried it.

Things get more complicated if you pushed up a branch that had a merge you didn't want.

Generally you want to do this:

http://stackoverflow.com/questions/7099833/how-to-revert-a-merge-commit-thats-already-pushed-to-remote-branch#answer-31223198

But first read this article to understand it better and the consequences:

http://www.christianengvall.se/undo-pushed-merge-git/

Another option which may not be practical:

http://stackoverflow.com/questions/7099833/how-to-revert-a-merge-commit-thats-already-pushed-to-remote-branch#answer-37819457


### Add Tag

A tag just adds a name to the last commit, typically used to mark a point in history, like a release.

IMPORTANT: DO NOT name a tag the same as a branch, git gets confused when you use common commands as it can't readily distinguish between branches and tags without you providing a full reference path. If you accidently do this, see the deletion commands below.

NOTE: By default a tag is created at the current HEAD position, but you can pass a commit hash as the last argument on the commands below to put a tag on any commit in the history. 

Add tag simple tag with no description

```git tag v1.0.0```

Create tag with message

```git tag -a v2.0.1 -m "This was a hotffix"```

Delete local tag using the name as reference:

```git tag -d v2.0.1```

Delete remote tag using the name as reference:

```git push origin :refs/tags/list```

If you want a tag with a long message, like release info

```git tag -a v2.0.1 -m```

Push tags to remote

```git push --tags```

Show tags

```git tag```

Show tags with the commit it is attached to

```git show-ref --tags```

You can follow up after the command above with the following to inspect the commit

```git show COMMITHASH```


### You can even use git to find in what file and line a search query can be found.

```git grep -n somethingtosearchfor```


### Merge Conflicts

This section assumes you've just issued command git merge and git states a merge conflict has occured.  Part of the messaging will state the files where a conflict was found, but if you need to see again which files have the conflict once again using status is your friend.

```git status```

You have two choices, you can abort the merge which takes your branch back to the state before you issued the merge command, or you can resolve the conflicts.  Let's look at both options, here is the command to abort the merge.

```git merge --abort```

Now we'll explore resolving the conflict.  Just like other versioning methods, git will wrap the problem lines of code in each file it finds conflict.  It will look something like this in the code file when you open in and editor.

```
<<<<<<< HEAD
var test = 'Hello World!';
=======
var test = 'Hello Cruel World?';
>>>>>>> origin/dev
```

Let's assume you were trying to merge remote branch dev into your local issue branch 19872_Fix-Header-Nav.  The top "<<<<<<< HEAD" section represents the code lines from 19872_Fix-Header-Nav, and the latter ">>>>>>> origin/dev" section is what it found on the remote dev branch that conflicts.

IMPORTANT: Sometimes the wrapping conflict "<<<<<<<" identifiers are hard to spot, in the wrong section, and there may be mutiple sections that need fixing, review the entire document carefully.

You'll need to manually remove all unnecessary code and the conflict "<<<<<<<" identifiers.  If our intention is to keep the work we did in 19872_Fix-Header-Nav and discard the code from the dev branch the correction would look like this.

```
var test = 'Hello World!';
```

After fixing all conflicts in all files be sure to save the files.  Now that the files have been corrected let's double-check we corrected them all.

```git status```

All the files you corrected should now be in the modified status, there should be no files with conflict, if there is, resolve them.

You'll notice git says that the it found Unmerged paths, you haven't yet resolved the merge conflict.  All this means is that you need to stage and commit these changes before git will see this merge conflict as resolved.

```
git add -A
git status
git commit -m "19872 Resolved merging in dev into 19872_Fix-Header-Nav"
```

Above in the first line we added all modified files, on the second we verified only the files we wanted staged are there, and the last we commit the fixes.


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
pick 218bdd8 fixed js prob
pick 9c63b7c fixed css prob

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
pick 218bdd8 fixed js prob
squash 9c63b7c fixed css prob
```

Above will replace these commits with one commit that has the message of both commits.

```
r 218bdd8 fixed js and css prob
f 9c63b7c added 9

```

Above will replace these commits with one commit that has the message of only the first commit which we've amended.

```
e 218bdd8 fixed js prob
r 9c63b7c fixed css prob and it rocks
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

### Git Archiving
[Use git archive to zip files at current state](https://git-scm.com/docs/git-archive)

[Use git bundle to transport the entire history as a single file](https://git-scm.com/docs/git-bundle)
[Bundle examples](https://stackoverflow.com/questions/11879287/export-git-with-version-history)

### SourceTree (...but you should really use the terminal :) )
[https://www.atlassian.com/software/sourcetree/overview](https://www.atlassian.com/software/sourcetree/overview)
