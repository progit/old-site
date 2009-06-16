## Rewriting History ##

There are many times when working with Git you may want to revise your commit history for some reason. One of the great things about Git is that it allows you to make decisions at the last possible moment.  You can decide what files go into which commits right before you commit with the staging area, you can decide that you didn't mean to be working on something yet with the stash command and you can rewrite commits that already happened so they look like they happened in a different way.  This could be changing the order of the commits, changing messages or modifying files in a commit, squashing together or splitting apart commits or removing commits entirely - all before we share our work with others.  

In this section we will cover how to accomplish these very useful tasks so that you can make your commit history look how you want it to look before you share it with others.

### Changing the Last Commit ###

Changing your last commit is probably the most common rewriting of history that you will do.  There are two basic things you will want to do to your last commit - you will often just want to change the commit message, or you may want to change the snapshot you just recorded by adding, changing and removing files.

If you only want to modify your last commit message, it is very simple. Just run

	$ git commit --amend

That will drop you into your text exitor which will have your last commit message in it that you can then edit.  When you save and close the editor, it will write a new commit with that message in it and make it your new last commit.  

If you have committed and then you want to change the snapshot you committed somehow by adding or changing files, it works basically the same way.  If you stage the changes you want by editing a file and running `git add` on it or `git rm` to a tracked file, the subsequent `git commit --amend` will take your current staging area and make it the snapshot for the new commit.

You need to be careful with this because amending does change the SHA-1 of that commit - it is like a very small rebase, do not amend your last commit if you've already pushed it.


### Changing Multiple Commit Messages ###

If you want to modify a commit that is farther back in your history, then we have to move to more complex tools.  Git does not have a 'modify history' tool, but we can sort of abuse the rebase tool to rebase a series of commits onto the head they were originally based on instead of moving them to another one.  With the 'interactive rebase' tool we can then stop after each commit we want to modify and change the message or add files or whatever we wish. You can run this with the -i option to git rebase. You will need to supply how far back you want to rewrite commits by telling it which commit to rebase onto.

For example, if you want to change the last 3 commit messages, or any of the commit messages in that group, you will have to supply as an argument to `git rebase -i` the parent of the last commit you want to edit, which would be `HEAD~2^` or `HEAD~3`.  It may be easier to remember the `~3` since you're trying to edit the last 3 commits, but keep in mind that you're actually designating 4 commits ago, the parent of the last commit you want to edit.

	$ git rebase -i HEAD~3

Remember again that this is a rebasing command - every commit that is included in this will be re-written, whether you change the message or not. Do not include any commit you have already pushed to a central server - it will mess other people up.

Running this command will give you a list of commits in your text editor that looks something like this:

	pick f7f3f6d changed my name a bit
	pick 310154e updated README formatting and added blame
	pick a5f4a0d added cat-file

	# Rebase 710f0f8..a5f4a0d onto 710f0f8
	#
	# Commands:
	#  p, pick = use commit
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit
	#
	# If you remove a line here THAT COMMIT WILL BE LOST.
	# However, if you remove everything, the rebase will be aborted.
	#

Now, it's important to note that these commits are listed in the opposite order then you normally see them using the `log` command.  If we run a log, we'll see something like this.

	$ git log --pretty=format:"%h %s HEAD~3..HEAD"
	a5f4a0d added cat-file
	310154e updated README formatting and added blame
	f7f3f6d changed my name a bit

Notice that is in the reverse order.  What the interactive rebase is doing is giving you a script that it's going to run.  It will start at the commit you specified on the command line (`HEAD~3`) and it will replay the changes introduced in each of these commits from top to bottom.  Since it does that, it lists the oldest at the top, rather than the newest, since it is the first it will replay.  

So, what we want to do is edit the script so that it stops at the commit we want to edit. To make the script stop after it applies a change, you need to edit the script.  Change the word 'pick' to the work 'edit' for each of the commits you want the script to stop after. For example, let's say we want to modify the third commit message only, we would change the file to look like this:

	edit f7f3f6d changed my name a bit
	pick 310154e updated README formatting and added blame
	pick a5f4a0d added cat-file

When you save and exit the editor, it will rewind you back to that last commit in that list and drop you on the command line with the following message:

	$ git rebase -i HEAD~3
	Stopped at 7482e0d... updated the gemspec to hopefully work better
	You can amend the commit now, with

		git commit --amend

	Once you are satisfied with your changes, run

		git rebase --continue

These instructions tell you exactly what to do. Type

	$ git commit --amend

Change the commit message and exit the editor. Then run

	$ git rebase --continue

This will try to apply the other two commits automatically and then you're done.  If you changed the 'pick' to 'edit' on more lines, you can repeat these steps for each commit you changed to 'edit'. Each time it will stop and let you amend the commit and continue when you're done.

#### Reordering Commits ####

You can also use interactive rebases to re-order or remove commits entirely.  If you wanted to remove the 'cat-file' commit and change the order that the other two commits were introduced, you would change the rebase script from this:

	pick f7f3f6d changed my name a bit
	pick 310154e updated README formatting and added blame
	pick a5f4a0d added cat-file

to this:

	pick 310154e updated README formatting and added blame
	pick f7f3f6d changed my name a bit

When you save and exit the editor, it will rewind your branch to the parent of these commits, then apply `310154e` and then `f7f3f6d` and then stop.  You have effectively changed the order of those commits and removed the 'cat-file' commit completely.

#### Squashing a Commit ####

It's also possible to take a series of commits and squash them down into a single commit with the interactive rebasing tool.  The script puts instructions in the rebase message that are very helpful.

	#
	# Commands:
	#  p, pick = use commit
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit
	#
	# If you remove a line here THAT COMMIT WILL BE LOST.
	# However, if you remove everything, the rebase will be aborted.
	#

So, if instead of 'pick' or 'edit' you specify 'squash', Git will apply both that change and the change directly before it and make you merge the commit messages together.  So, if we wanted to make a single commit from these three commits, we would make the script look like this:

	pick f7f3f6d changed my name a bit
	squash 310154e updated README formatting and added blame
	squash a5f4a0d added cat-file

When we save and exit the editor, Git will apply all three changes and then put us back into the editor to merge the three commit messages:

	# This is a combination of 3 commits.
	# The first commit's message is:
	changed my name a bit

	# This is the 2nd commit message:

	updated README formatting and added blame

	# This is the 3rd commit message:

	added cat-file

When we save that out, we'll have a single commit that introduces the changes of all three previous commits.

#### Splitting a Commit ####

Splitting a commit is just undoing a commit and then partially staging and committing as many times as commits you want to end up with.  For example, let's say we want to split the middle commit of our 3 commits.  Instead of 'updated README formatting and added blame', we want to split it into two commits - 'updated README formatting' for the first, and 'add blame' for the second.  We can do that by changing the instruction on the commit we want to split up to 'edit' in the `rebase -i` script.

	pick f7f3f6d changed my name a bit
	edit 310154e updated README formatting and added blame
	pick a5f4a0d added cat-file

Then when the script drops you to the command line, you reset that commit and then take the changes that have been reset and create multiple commits out of them.  When you save and exit the editor, it will rewind to the parent of the first commit in your list, apply the first commit (`f7f3f6d`), apply the second (`310154e`) and then drop you to the console.  There, you can do a mixed reset of that commit with `git reset HEAD^`, which will effectively undo that commit.  Now you can stage and commit files until you have several commits and run `git rebase --continue` when you are done.

	$ git reset HEAD^
	$ git add README
	$ git commit -m 'updated README formatting'
	$ git add lib/simplegit.rb
	$ git commit -m 'added blame'
	$ git rebase --continue

Now Git will then apply the last commit (`a5f4a0d`) in the script and now you have a history that looks like this instead:

	$ git log -4 --pretty=format:"%h %s"
	1c002dd added cat-file
	9b29157 added blame
	35cfb2b updated README formatting
	f3cc40e changed my name a bit

Once again, this changes the SHAs of all the commits in your list, so make sure that no commit shows up in that list that you have already pushed to a shared repository.

### The Nuclear Option : filter-branch ###

There is another history rewriting option which you would use if you need to rewrite a larger number of commits in some scriptable way.  For instance, change your email address globally, or remove a file from every commit.  The command is `filter-branch` and it can rewrite huge swaths of your history, so it probably shouldn't be used unless your project is not yet public or has otherwise not had other people base work off the commits you're about to rewrite.  However, it can be very useful and we'll just cover a few of the common uses so you can get an idea of some of the things it's capable of.

#### Removing a File from Every Commit ####

This occurs fairly commonly.  Someone accidentally commits a huge binary file with a thoughtless `git add .` and you want to remove it everywhere. Perhaps you accidentally committed a file that had a password in it and you want to make your project open source.  `filter-branch` is the tool you'll probably want to use in order to scrub your entire history.  To remove a file named passwords.txt from your entire history, you can use the `--tree-filter` option to filter-branch.

	$ git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
	Rewrite 6b9b3cf04e7c5686a9cb838c3f36a8cb6a0fc2bd (21/21)
	Ref 'refs/heads/master' was rewritten
	
The `--tree-filter` option will run the specified command on after each checkout of the project and then re-commit the results.  In this case, I'm removing a file called 'passwords.txt' from every snapshot, whether it exists or not.  If I wanted to remove all accidentally committed editor backup files I might run something like `git filter-branch --tree-filter 'rm -f *~' HEAD`.

You will be able to watch it rewriting trees and commits and then move the branch pointer at the end.  It is generally a good idea to do this in a testing branch and then hard reset your master branch after you've determined the outcome is what you really want.  If you want to run it on all of your branches, you can pass `--all` to the command.

#### Making a Subdirectory the new Root ####

If you've done an import from another source control system and have subdirectories that simply make no sense ('trunk', 'tags', etc) and you want to make the 'trunk' subdirectory be the new project root for every commit, `filter-branch` can help you do that too.

	$ git filter-branch --subdirectory-filter trunk HEAD
	Rewrite 856f0bf61e41a27326cdae8f09fe708d679f596f (12/12)
	Ref 'refs/heads/master' was rewritten

Now our new project root is what was in the 'trunk' subdirectory each time.  It will also remove commits that did not effect that subdirectory automatically.

#### Changing Email Addresses Globally ####

Another common case is that you forgot to run `git config` to set your name and email address before you started working, or perhaps you want to open source a project at work and want to change all your work email addresses to your personal one.  In any case, changing email addresses in multiple commits in a batch is done with `filter-branch` as well.  You want to be careful to only change the email addresses that are yours, so we'll use a `--commit-filter` for this.

	$ git filter-branch --commit-filter '
	        if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
	        then
	                GIT_AUTHOR_NAME="Scott Chacon";
	                GIT_AUTHOR_EMAIL="schacon@example.com";
	                git commit-tree "$@";
	        else
	                git commit-tree "$@";
	        fi' HEAD

This will go through and rewrite every commit to have your new address.  Since commits contain the SHA-1 values of their parents, this will basically change every commit SHA in your history, not just the ones that have the matching email address.