## Stashing ##

The situation often occurs when you've been working on some part of your project, things are in a messy state and you want to switch branches for a bit to work on something else.  The problem is that you don't want to do a commit of half-done work just so you can get back to this point later.  The answer to this issue is the `git stash` command.

What `stash` does is to take the dirty state of your working directory - that is, your modified tracked files and staged changes - and saves it on a stack of unfinished changes which you can re-apply at any time.

### Stashing your work ###

So, let's go into our project and start working on a couple of files and possibly stage one of the changes.  If we run `git status` we can see our dirty state:

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	modified:   index.html
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   lib/simplegit.rb
	#

Now, we want to switch branches but we don't want to commit what we've been working on yet, so we're going to stash these changes.  To push a new stash onto your stack, simply type `git stash`.

	$ git stash
	Saved working directory and index state "WIP on master: 049d078 added the index file"
	HEAD is now at 049d078 added the index file
	(To restore them type "git stash apply")

Now we can see that our working directory is clean.

	$ git status
	# On branch master
	nothing to commit (working directory clean)

At this point we can easily switch branches and do work elsewhere and our changes are stored on our stack.  If we want to see which stashes we have stored, we can use `git stash list`.

	$ git stash list
	stash@{0}: WIP on master: 049d078 added the index file
	stash@{1}: WIP on master: c264051... Revert "added file_size" - not implemented correctly
	stash@{2}: WIP on master: 21d80a5... added number to log

In this case, I had already done two stashes previously, so we can see that we have access to 3 different stashed works.  So, let's reapply the one we just stashed with the command you can see in the help output of the original stash command, `git stash apply`.  If you want to apply one of the older stashes, you can specify it by naming it like `git stash apply stash@{2}`, but if you do not specify one Git will assume the most recent stash and try to apply that.

	$ git stash apply
	# On branch master
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   index.html
	#	modified:   lib/simplegit.rb
	#

You can see that it re-modifies the files that we had uncommitted when we saved the stash.  In this case we had a clean working directory when we tried to apply the stash and we tried to apply it on the same branch that we saved it from, however those are not neccesary to successfully apply a stash.  You can save a stash on one branch, switch to another branch later and try to reapply the changes.  You can also have modified and uncommitted files in your working directory when you apply a stash - Git will give you merge conflicts if anything no longer applies cleanly.

Now, you can see that we got the changes to our files re-applied, but the file we had staged before this was not re-staged.  If you want to do that, you have to run the `git stash apply` command with a `--index` option to tell it to try to re-apply the staged changes.  If we had run that instead, we would have gotten back to our original position.

	$ git stash apply --index
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	modified:   index.html
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   lib/simplegit.rb
	#
	
The `apply` option will only try to apply the stashed work - you will continue to have it on your stack.  To remove it, you can run `git stash drop` with the name of the stash to remove.

	$ git stash list
	stash@{0}: WIP on master: 049d078 added the index file
	stash@{1}: WIP on master: c264051... Revert "added file_size" - not implemented correctly
	stash@{2}: WIP on master: 21d80a5... added number to log
	$ git stash drop stash@{0}
	Dropped stash@{0} (364e91f3f268f0900bc3ee613f9f733e82aaed43)

You can also run `git stash pop` to apply the stash and then immediately drop it from your stack.

### Creating a Branch from a Stash ###

If you have stashed some work and then left it there for a while and continued on the branch you stashed your work from, you may have a problem re-applying the work.  If the apply tries to modify a file that you've since modified, you will get a merge conflict and will have to try to resolve it.  If you want an easier way to test out the stashed changes again, you can run `git stash branch`, which will create a new branch for you, checkout the commit you were on when you stashed your work, re-apply your work there and then drop the stash if it applies successfully.

	$ git stash branch testchanges
	Switched to a new branch "testchanges"
	# On branch testchanges
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	modified:   index.html
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   lib/simplegit.rb
	#
	Dropped refs/stash@{0} (f0dfc4d5dc332d1cee34a634182e168c4efc3359)

This is a nice shortcut to recovering stashed work easily and working on it in a new branch.
