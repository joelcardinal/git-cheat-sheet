# Git Cheat Sheet

- [Getting Started](#user-content-getting-started)
- [Cheat Sheet](#user-content-cheat-sheet)
- [Resources](#user-content-resources)

## Getting Started

Install Git: Windows ([msysgit](http://msysgit.github.io/)) Mac ( [osx-installer](http://sourceforge.net/projects/git-osx-installer/) | [older dmg](https://code.google.com/p/git-osx-installer/downloads/list)) Or Source ([instructions](http://git-scm.com/book/en/Getting-Started-Installing-Git))
Create an account on the repository host (e.g. [bitbucket.org](bitbucket.org), [github.com](github.com)).
Create your public key ([instructions](https://help.github.com/articles/generating-ssh-keys)).
Provide remote host admin with public key (if working with team).

## Cheat Sheet

**Creating new local branch (when one exists on server)**

bring down everything from server, including all branches

```git fetch origin```

create a local branch called "seewhy_test" and use history from origin/seewhy_test

```git checkout -b seewhy_test origin/seewhy_test```

now that you are in local branch "seewhy_test" merge that history with "other_branch"

```git merge other_branch```

push the changed local branch "seewhy_test" back up to the server

```git push origin seewhy_test```

To delete a local branch (first checkout master to avoid erroneous git warnings)

```git branch -d the_local_branch```

To remove a remote branch (if you know what you are doing!)

```git push origin :the_remote_branch```

Prevent showing branches that have already been deleted

```git remote prune origin```


**Committing changes and pushing back to server**

commit all changes in working space

```git commit -am "SeeWhy login page"```

bring down everything from server, including all branches

```git fetch origin```

merge any other updates into your build

```git merge origin/seewhy_test```

push back changed history to server

```git push origin seewhy_test```

To push to a different remote repository first add the new repository as an option

```git remote add demandware_pod  git@github.com:cpocommerce/demandware_pod.git```

Above, 'demandware_pod' is the local name you are giving the remote repository demandware_pod. The 'git@...' is the ssh path on GitHub. Now we'll push to this other remote repository.

```git push demandware_pod branchName```

Notice above instead of 'origin' we use the new local repository name of 'demandware_pod', which is just a convention for this example it could be anything.

**See history on server**

```git fetch origin/seewhy_test```

```git log origin/seewhy_test```

**Show the commits that are in your current branch, but not origin -- i.e., whether you're ahead of origin and by which commits.**

```git rev-list origin..HEAD```

**Renaming a local branch.**

```git branch -m old_branch_name new_branch_name```

**Stash: This allows you to store away changes that have not been committed. It also resets the head to the last commit, as if you never made those changes.**

Here we are saving an uncommitted change and adding a message:

```git stash save "Partial work for issue X"```

Here we list our stashes:

```git stash list```

Here we bring back and apply our stash. I believe you can edit the stash ID so that you apply the stash you want:

```git stash apply stash@{0}```

The stash will still be there after you apply it, to delete the stash:

```git stash drop stash@{0}```

**File or Commit History:**

Shows commits by line

```git blame filePath```

Shows history of file

```git log -p --follow filePath```

Shows diff of commit

```git show commitHash```

**Undo the last commit:**

Use soft if you want to undo the change, but keep your changes in the active workspace

```git reset --soft HEAD~1```

Use hard if you want to undo the change and get rid of the changes in the active workspace

```git reset --hard HEAD~1```

**Move a commit to another branch:**

First you would checkout the branch you want to work with:

```git checkout master```

Then you would look at the commits and find the commit you wanted

```git log```

Then copy the commit serial number. Then checkout the branch you want to put it in:

```git checkout receiving_branch```

Then use the following statement where the serial number is pasted after cherry-pick:

```git cherry-pick 103298764wek23089476iwei28934```

Note: cherry-pick does not delete the commit from the original branch.

**Undo a specific commit somewhere in the history:**

First look at the commits and find the commit you want to undo

```git log```

Then copy the commit serial hash and use the following command, where the hash of the commit is used after revert:

```git revert 103298764wek23089476iwei28934```

**Note: this does not get rid of the commit you didn't want--it creates a new commit at the end of the branches history that performs the opposite action of that commit. THIS IS VERY IMPORTANT TO UNDERSTAND. If you ever revert your commit history to a period earlier than this "undo" commit, the "undo" won't be there. Also, if you perform this, commits that depended on the unwanted commit will also be undone.**

In some cases it may be best to just create a branch that starts at the commit prior to the unwanted commit, then cherry-pick over commits that happened later that you do want. There are always a few different ways to skin a cat in git.

**Undo a merge:**

Undoing a merge is similar to the above, but we need to provide git with more information.

```git revert 803298764wek23089476iwei28934 -m 1```

In the example above, the long sha number is the sha number provided in the log when you merged. The 1 stands for the first commit in the branch you merged in, a 2 would stand for the second commit made in the branch you merged in. In this example, if you had more than one commit, you would not be un-committing the entire merge, just that particular commit in the branch you merged in. Understandably, this method is only practical if you merged in a branch which had few commits.

Additionally, after doing this you will not be able to merge back in any ancestors of this merge branch point

**Tags: These allow you to define a certain period in the history, spanning all branches.**

Lets create a tag with a simple name and message:

```git tag -a v1.4 -m 'my vrsion 1.4'```

Oops, I mis-spelt version so let's delete it using the name as reference:

```git tag -d v1.4```

After you create it correctly... Now we need to push it:

```git push --tags```

**You can even use git to find in what file and line a search query can be found.**

```git grep -n somethingtosearchfor```

**Index / Staging**

Other than commits, you have two other states of your changes, staged and un-staged. This wiki mostly discusses committing changes with:

```git commit -a```

The above actually commits BOTH staged and un-staged changes. So lets go through a scenario. You are in Studio or an editor of your choice. You make a change to a file and then you save the file in the editor.
Now you have an un-staged commit. Let's now stage the commit:

```git stage file_name```

You could also use the command below, however doing so also adds any untracked files into the git history:

```git add file_name```

If you stage something and then want to un-stage it on a certain file, run:

```git reset HEAD file_name
To un-stage all files use:

```git reset HEAD```

The advantage of staging before committing is that it gives you more freedom to manage what gets committed and when. Note, staging is not like creating a commit, if you make several changes and stage each change, their will not be a record for each staging point. If you now run…:

```git commit```

…only the files you staged will be committed.

**Merge Conflicts**

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

**Change Repo Name/Location**

On occasion the remote repo will change name or location.

To update your local pointer to the new remote location:

```git remote set-url origin [updated link url git@........git]```

## Resources

**Basics and Commands**

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

**Branch Management**

[http://git-scm.com/book/en/Git-Branching](http://git-scm.com/book/en/Git-Branching)
[https://www.atlassian.com/git/workflows](https://www.atlassian.com/git/workflows)
[http://nvie.com/posts/a-successful-git-branching-model/](http://nvie.com/posts/a-successful-git-branching-model/)
[http://drewfradette.ca/a-simpler-successful-git-branching-model/](http://drewfradette.ca/a-simpler-successful-git-branching-model/)

**Connect Git Commits to Jira**

[https://confluence.atlassian.com/display/BITBUCKET/Linking+Bitbucket+and+GitHub+accounts+to+JIRA](https://confluence.atlassian.com/display/BITBUCKET/Linking+Bitbucket+and+GitHub+accounts+to+JIRA)

**Self hosting repository**

[https://www.atlassian.com/software/stash](https://www.atlassian.com/software/stash)
[http://www.gitorious.com/local_install/](http://www.gitorious.com/local_install/)
[www.gitlab.com](www.gitlab.com)

**Git Management Tools**
SourceTree - If the terminal isn't your thing.</p>

[https://www.atlassian.com/software/sourcetree/overview](https://www.atlassian.com/software/sourcetree/overview)

