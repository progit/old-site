## Recording changes to the repository ##

So now you have a bona-fide Git repository and a checkout or ‘working copy’ of the files for that project.  Now we need to make some changes and commit snapshots of those changes into our repository each time the project reaches a state we want to record.

Remember that each file in your working directory can be in one of two states – tracked and untracked, and of the tracked files, they can be unmodified, modified or staged.  When you clone a repository, all of your files will be tracked and unmodified.  As you make changes, new files you add will be untracked, and existing files that you change will be tracked and modified.  Then you will stage them, then you will commit all your staged changes and the cycle will repeat.  At any point you can inspect or roll back to versions of your files or project at the any point that you committed.

### Checking the status of your files

The main tool that you will use to determine which files are in which state is the ‘git status’ command.  If you run this command directly after a clone, you should see something like this:

	$ git status
	# On branch master
	nothing to commit (working directory clean)

This means that you have a clean working directory, in other words, there are no tracked and modified files.  We can also see that there are no untracked files that Git sees, or they would also be listed here.  Finally, we can see that our command tells us which ‘branch’ we are on.  For now, that is always going to be ‘master’, which is the default and we won’t worry about it here.  The next chapter will go over branches and references in great detail.

Let’s say we add a new file to our project, a simple README file.  If the file did not exist before and you run ‘git status’, we will see our untracked file like so:

	$ vim README
	$ git status
	# On branch master
	# Untracked files:
	#   (use "git add <file>..." to include in what will be committed)
	#
	#	README
	nothing added to commit but untracked files present (use "git add" to track)

So we can see that our new README file is ‘untracked’, as it is under the ‘Untracked files’ heading in our status output.  "Untracked" basically means that Git sees a file that you didn't have in the previous snapshot (commit), and so Git won't start including it in your commit snapshots until you explicity tell it to do so.  It does this so that you don't accidentally start including generated binary files or something. We do want to start including it, so let’s start tracking the file.

### Tracking new files

In order to begin tracking a new file we use the command ‘git add’.  In order to begin tracking the README file, we can simply run this:

	$ git add README
	Now, if we run our ‘status’ command again, we can see that our README file is now tracked and staged.
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#


We can tell that it is staged because it is under the ‘Changes to be committed’ heading.  If we commit at this point, the version of the file at the time you ran ‘git add’ is what will be in the historical snapshot.  You may recall when we ran ‘git init’ earlier that we then ran a ‘git add (files)’ – that was to begin tracking files in our directory. The ‘git add’ command will take a path name, either a file or a directory, and if it is a directory it will add all the files in that directory recursively.

### Staging modified files

Now let us change a file that was already tracked.  If I make a change to a previously tracked file called ‘benchmarks.rb’ and then run our ‘status’ command again, I will get something that looks like this:

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   benchmarks.rb
	#

Now we can see the ‘benchmarks.rb’ file under a section named ‘Changed but not updated’ – which means that a file that is tracked has been modified in the working directory but not yet staged.  To stage it, we run the ‘git add’ command (it’s kind of a multi-purpose command – we use it to start tracking new files, stage files and even other things, like marking merge conflicted files as resolved).  Let’s run ‘git add’ now to stage the benchmarks.rb file, then ‘git status’ again.

	$ git add benchmarks.rb
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#	modified:   benchmarks.rb
	#

Now we can see that both files are staged and will go into our next commit. At this point we remember one little change that we wanted to make in benchmarks.rb before we commit it, so we open it back up again and make that one tiny change.  Now we’re ready to commit.  However, let’s run ‘git status’ one more time.

	$ vim benchmarks.rb 
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#	modified:   benchmarks.rb
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   benchmarks.rb
	#

What the heck?  Now we have benchmarks.rb listed both as staged and unstaged?  How is that possible?  It turns out that Git stages a file as exactly as it is when you run the ‘git add’ command.  If we were to commit now, the version of ‘benchmarks.rb’ as it was when you actually last ran the ‘git add’ command is how it will go into the commit, not what that file looks like when you run ‘git commit’.  If you modify a file after you run ‘git add’, you have to run ‘git add’ again to stage the latest version of it.

	$ git add benchmarks.rb
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#	modified:   benchmarks.rb
	#

### Ignoring files

Often times there are a class of files that you don’t want Git to automatically add or even show you as being untracked.  These are generally automatically generated files, such as log files or files that are produced by your build system.

	$ cat .gitignore
	*.[oa]
	*~

The first line tells Git to ignore any files ending in .o or .a – object and archive files that might be the product of building your code. The second line tells Git to ignore all files that end with a tilde which is used by many text editors such as Emacs to mark temporary files.  You might also have a log, tmp or pid directory in here, automatically generated documentation, etc.  Setting this up before you get going is generally a good idea so you don’t accidentally commit files that you really don’t want in your Git repository.

DWP: Is it worth pointing to somewhere where the reader can learn about glob? Or explaining what a couple of symbols mean? {: class=note}

### Removing files

DWP: This is for removing a file that has already been committed, isn't it? The example just above has been staged but not committed, I think. Presumably removal of the file from staging is different. Would it be better to commit before the next para? Or would it work fine with a file which hasn't been committed yet.{: class=note}

In order to remove a file from Git, you have to remove it from your tracked files (more accurately, remove it from your staging area) and then commit.  The ‘git rm’ command will do that and will also remove it from your working directory so you don’t see it as an untracked file next time around.

If you simply remove the file, it will show up under the ‘Changed but not updated’ (ie: unstaged) area of your ‘git status’ output:

	$ rm grit.gemspec
	$ git status
	# On branch master
	#
	# Changed but not updated:
	#   (use "git add/rm <file>..." to update what will be committed)
	#
	#       deleted:    grit.gemspec
	#

Then, if you run ‘git rm’, it will ‘stage’ the removal.

	$ git rm grit.gemspec
	rm 'grit.gemspec'
	$ git status
	# On branch master
	#
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       deleted:    grit.gemspec
	#

Now the next time you commit, the file will be gone and no longer tracked.  If you modified the file and added it to the index already, you will need to force the removal with the -f option. This is a safety feature to prevent accidental removal of data which hasn't yet been recorded in a snapshot, and thus cannot be recovered from Git.

Another useful thing you might want to do is to keep the file in your working tree files but remove it from your index. In other words, you might want to keep the file on your hard drive but you don’t want Git to track it anymore.  This is particularly useful if you forgot to add something to your ‘.gitignore’ file and accidentally added it, like a large log file or a bunch of '.a' compiled files.  To do this, use the --cached option:

	$ git rm --cached readme.txt

You can pass files, directories and file glob patterns to the ‘git rm’ command. That means you can do things such as:

	$ git rm log \*.log

Note the '\\' in front of the '\*'.  This is because Git does it's own filename expansion in addition to your shells filename expansion. That command will remove all files that have the “.log” extension inside the “log/” directory. Or, you can simply do:

	$ git rm \*~

That will remove all files that ends with ‘~’.

### Moving files

Unlike many other VCS systems, Git does not explicitly track file movement.  If you rename a file in Git, there is no metadata stored in Git that tells it that you renamed the file.  However, Git is pretty smart about figuring that out after the fact – we’ll deal with detecting file movement a bit later.  

What is a bit confusing then is that Git actually does have a ‘mv’ command.  If you want to rename a file in Git, you can simply run something like 

	$ git mv file_from file_to

And this will work fine.  In fact, if you run something like this and look at the status, you’ll see that Git sees it as a renamed file.

	$ git mv README.txt README
	$ git status
	# On branch master
	# Your branch is ahead of 'origin/master' by 1 commit.
	#
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       renamed:    README.txt -> README
	#

However, this is actually equivalent to running something like this:

	$ mv README.txt README
	$ git rm README.txt
	$ git add README

Git figures out that it’s a rename implicitly, so it doesn’t matter if you rename a file that way or with the ‘mv’ command.  The only real difference is that ’mv’ is one command instead of three – it is there as a convenience function. More importantly, you can use any tool you like to rename a file, and address the add/rm later, before you commit.

### Viewing your staged and unstaged changes

If the ‘git status’ command is too vague for you – you want to know exactly what you changed, not just which files were changed, you can use the ‘git diff’ command.  We will cover ‘git diff’ in more detail a bit later, but the two ways you will probably use it most often will be to answer the questions ‘What have I changed but not yet staged?’ and ‘What have I staged and am about to commit?’.  Though ‘git status’ will answer those very generally, ‘git diff’ will show you the exact lines added and removed – the ‘patch’, as it were.  So, let's say that we edit and stage the 'README' file again, and then just edit the 'benchmarks.rb' file without yet staging it.  If we run our `status` command, we'll once again see something like this:

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   benchmarks.rb
	#

To see what you have changed but not yet staged, type ‘git diff’ with no other arguments.

	$ git diff
	diff --git a/benchmarks.rb b/benchmarks.rb
	index 3cb747f..da65585 100644
	--- a/benchmarks.rb
	+++ b/benchmarks.rb
	@@ -36,6 +36,10 @@ def main
	           @commit.parents[0].parents[0].parents[0]
	         end
 
	+        run_code(x, 'commits 1') do
	+          git.commits.size
	+        end
	+
	         run_code(x, 'commits 2') do
	           log = git.commits('master', 15)
	           log.size

That will compare what is in your working directory with what is in your staging area.  This tells you the changes you have made that you have not staged yet.

Now, if we want to see what we have staged and will go into our next commit, we can use ‘git diff –-cached’ (In newer versions of Git, 1.6.1 and later, you can also use ‘git diff –-staged’, which may be easier to remember).  This will compare our staged changed to our last commit.

	$ git diff --cached
	diff --git a/README b/README
	new file mode 100644
	index 0000000..03902a1
	--- /dev/null
	+++ b/README2
	@@ -0,0 +1,5 @@
	+grit
	+ by Tom Preston-Werner, Chris Wanstrath
	+ http://github.com/mojombo/grit
	+
	+Grit is a Ruby library for extracting information from a git repository

It is important to note that ‘git diff’ by itself does not show all changes made since your last commit – only the changes that are still unstaged.  This can be confusing, since if you have staged all of your changes, ‘git diff’ will give you no output.  

For another example, if you now stage that 'benchmarks.rb' file, then edit it, we can use `git diff` to see the changes in that file that are staged and the changes that are unstaged.

	$ git add benchmarks.rb
	$ echo '# test line' >> benchmarks.rb
	$ git status
	# On branch master
	#
	# Changes to be committed:
	#
	#	modified:   benchmarks.rb
	#
	# Changed but not updated:
	#
	#	modified:   benchmarks.rb
	#
 
So now we can use `git diff` to see what is still unstaged:

	$ git diff 
	diff --git a/benchmarks.rb b/benchmarks.rb
	index e445e28..86b2f7c 100644
	--- a/benchmarks.rb
	+++ b/benchmarks.rb
	@@ -127,3 +127,4 @@ end
	 main()

	 ##pp Grit::GitRuby.cache_client.stats 
	+# test line

And `git diff --cached` to see what we've staged so far:

	$ git diff --cached
	diff --git a/benchmarks.rb b/benchmarks.rb
	index 3cb747f..e445e28 100644
	--- a/benchmarks.rb
	+++ b/benchmarks.rb
	@@ -36,6 +36,10 @@ def main
	          @commit.parents[0].parents[0].parents[0]
	        end

	+        run_code(x, 'commits 1') do
	+          git.commits.size
	+        end
	+              
	        run_code(x, 'commits 2') do
	          log = git.commits('master', 15)
	          log.size



### Committing your changes

Now that your staging area is all setup how you want it, we can commit your changes.  Remember that anything that is still unstaged – any files you have added or modified that you have not run ‘git add’ on since you edited them – will not go into this commit.  They will stay as modified files on your disk.  

However, the last time we ran ‘git status’ we saw that everything was staged, so we’re ready to commit our changes.  The simplest way to commit is to just type ‘git commit’, which will launch you into your editor of choice (this will be set by the $EDITOR environment variable of your shell – usually vim or emacs, though you can override it with whatever you want with the `git config --global core.editor` command as we saw in Chapter 1), and in your editor it will have you type in your commit message.

	$ git commit

Now your editor will launch with the following text (this example is a vim screen):

	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       new file:   README
	#       modified:   benchmarks.rb 
	~                                                                            
	~                                                                            
	~                                                                            
	".git/COMMIT_EDITMSG" 10L, 283C
	
So you can see that it simply contains the latest output of the git status command commented out and one empty line on top.  You can remove these comments and type your commit message, or you can just leave them there to help you remember what you are committing.  (For an even more explicit reminder of what you modified, you can pass the ‘-v’ option to ‘git commit’, which will also put the diff of your change in the editor so you can see exactly what you did).  When you exit out of the editor, Git will create your commit with that commit message (with the comments and diff stripped out).

Alternatively, you can just type your commit message inline with the `commit` command by specifying it after a `-m` flag, like this:

	$ git commit -m "Story 182: Fix benchmarks for speed"
	[master]: created 463dc4f: "Fix benchmarks for speed"
	 2 files changed, 3 insertions(+), 0 deletions(-)
	 create mode 100644 README

So, now you’ve created your first commit!  You can also see that the commit has given you some output about that commit – telling you which branch you committed to (master), what SHA-1 checksum the commit has (463dc4f), how many files were changed and statistics on lines added and removed in the commit.

Remember that the commit records the snapshot you setup in your staging area.  Anything you didn’t stage is still sitting there modified, and you can do another commit to add them to your history.  Every time you perform a commit, you are simply recording a snapshot of your project that you can revert to or compare to later.
