## Undoing things ##

At any stage here we may want to undo something that we did.  Here we will review a few basic tools for undoing changes that we made.  Be careful here, undoing some of these undos is not always possible.  This is one of the few areas in Git where you may lose some work if you do it wrong.

### Changing your last commit

One of the more common ‘undo’s is when you commit too early, possibly forgetting to add some files, or you mess up your commit message.  If you want to try that commit again, you can simply run commit with the `--amend` option.

	$ git commit --amend

This is going to take your staging area and use it for the commit.  If you have made no changes since your last commit (for instance, you run it immediately after your previous commit) then your snapshot will look exactly the same and all you’re changing is your commit message.

The same commit message editor will fire up, but it will already contain the message of your previous commit.  So you can edit it the same as normal, but it will overwrite the last commit you did.

As an example, if you committed, then realized you forgot to stage the changes in a file you wanted to add to this commit, you can do something like this:

	$ git commit -m ‘initial commit’
	$ git add forgotten_file
	$ git commit --amend 
	
All three of those commands will only end up with a single commit – the second command replaces the results of the first.

### Unstaging a staged file

The next two sections are demonstrating how to wrangle your staging area and working directory changes.  The nice part is that the command that you use to tell you what state those two areas are in will also remind you how to undo changes to them.  For example, let’s say that we’ve changed two files and want to commit them as two separate changes, but we accidentally typed `git add .` and staged them both.  How can we unstage one of the two? The `git status` command will remind us.

	$ git add .
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       modified:   README.txt
	#       modified:   benchmarks.rb
	#

You can see that right below the ‘Changes to be committed’ text it says ‘use "git reset HEAD <file>..." to unstage’.  So, let’s use that advice to unstage the benchmarks.rb file.

	$ git reset HEAD benchmarks.rb 
	benchmarks.rb: locally modified
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       modified:   README.txt
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#       modified:   benchmarks.rb
	#

The command is a bit strange, but we can see that it worked.  The benchmarks.rb file is modified but once again unstaged.

### Unmodifiying a modified file

What if we realize that we actually don’t want the changes to the benchmarks.rb file at all. How can we easily unmodify it – revert it back to what it looked like when we last committed (or initially cloned – however we got it into our working directory)?  Luckily, ‘git status’ tells us how to do that, too.  Looking at the last example output, we can see the ‘unstaged’ area looks like this:

	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#       modified:   benchmarks.rb
	#

So it tells us pretty explicitly how to discard the changes we’ve made (at least, the newer versions of Git, 1.6.1 and later, will do this – if you have an older version, I highly recommend upgrading it to get some of these nicer usability features).  So, lets do what it tells us.

	$ git checkout -- benchmarks.rb
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       modified:   README.txt
	#

Now we can see that the changes have been reverted.  You should also realize that this is a pretty dangerous command.  Any changes you made to that file are completely gone – you just copied another file over it.  Don’t ever use this unless you absolutely know that you don’t want this file.  If you need to get it out of the way, we will go over stashing and branching in the next chapter which are generally better ways to go.  Remember – anything that is committed in Git you can almost always recover.  Even commits that were only on branches that were deleted or commits that were overwritten with an `--amend` commit can be recovered (see Chapter 9 for data recovery). However, anything that you lose that was _never_ committed is likely never to be seen again.
