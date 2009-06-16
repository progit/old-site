## Interactive Staging ##

Git comes with a couple of scripts that make some command line tasks easier.  Here we will look at a few interactive commands that help you easily craft your commits to only include certain combinations and parts of files.  These tools are very helpful if you modified a bunch of files and then decided that you wanted those changes to be in several focused commits rather than one big messy commit.  This is very helpful for making sure that your commits are logically separate changesets and easily reviewable by the developers working with you.

### Interactive Add ###

If you run `git add` with the `-i` or `--interactive` options, Git will go into an interactive shell mode, displaying something like this:

	$ git add -i
	           staged     unstaged path
	  1:    unchanged        +0/-1 TODO
	  2:    unchanged        +1/-1 index.html
	  3:    unchanged        +5/-1 lib/simplegit.rb

	*** Commands ***
	  1: status	  2: update	  3: revert	  4: add untracked
	  5: patch	  6: diff	  7: quit	  8: help
	What now> 


You can see that it shows you a much different view of your staging area - basically the same information you'll get with `git status` but a bit more succinct and informative.  It lists the changes you have staged on the left and unstaged changes on the right hand column.  After this you have a commands section.  Here we can do a number of things, including staging files, unstaging files, staging parts of files, adding untracked files and seeing diffs of what has been staged.

### Staging and Unstaging Files ###

If you type '2' or 'u' on the 'What now>' prompt, the script will prompt you for which files you want to stage.

	What now> 2
	           staged     unstaged path
	  1:    unchanged        +0/-1 TODO
	  2:    unchanged        +1/-1 index.html
	  3:    unchanged        +5/-1 lib/simplegit.rb
	Update>>

If you wanted to stage the 'TODO' and 'index.html' files, you can simply type the numbers:

	Update>> 1,2
	           staged     unstaged path
	* 1:    unchanged        +0/-1 TODO
	* 2:    unchanged        +1/-1 index.html
	  3:    unchanged        +5/-1 lib/simplegit.rb
	Update>>
	
The '\*' next to each file means that the file is selected to be staged.  If you just hit 'enter' after typing nothing at the 'Update>>' prompt, it will take anything selected and stage it for you.

	Update>> 
	updated 2 paths

	*** Commands ***
	  1: status	  2: update	  3: revert	  4: add untracked
	  5: patch	  6: diff	  7: quit	  8: help
	What now> 1
	           staged     unstaged path
	  1:        +0/-1      nothing TODO
	  2:        +1/-1      nothing index.html
	  3:    unchanged        +5/-1 lib/simplegit.rb

Now we can see that the TODO and index.html files are staged and the simplegit.rb file is still unstaged.  If we wanted to unstage the TODO file at this point, we use the '3' or 'r' for 'revert' option.

	*** Commands ***
	  1: status	  2: update	  3: revert	  4: add untracked
	  5: patch	  6: diff	  7: quit	  8: help
	What now> 3
	           staged     unstaged path
	  1:        +0/-1      nothing TODO
	  2:        +1/-1      nothing index.html
	  3:    unchanged        +5/-1 lib/simplegit.rb
	Revert>> 1
	           staged     unstaged path
	* 1:        +0/-1      nothing TODO
	  2:        +1/-1      nothing index.html
	  3:    unchanged        +5/-1 lib/simplegit.rb
	Revert>> [enter]
	reverted one path

Now if we look at our Git status again, we can see that we've unstaged the TODO file:

	*** Commands ***
	  1: status	  2: update	  3: revert	  4: add untracked
	  5: patch	  6: diff	  7: quit	  8: help
	What now> 1
	           staged     unstaged path
	  1:    unchanged        +0/-1 TODO
	  2:        +1/-1      nothing index.html
	  3:    unchanged        +5/-1 lib/simplegit.rb

If we want to see the diff of what we've staged, we can use the '6' or 'd' for 'diff' command.  This will show us a list of our staged files and we can select which ones we would like to see the staged diff for.  This is much like specifying `git diff --cached` on the command line.

	*** Commands ***
	  1: status	  2: update	  3: revert	  4: add untracked
	  5: patch	  6: diff	  7: quit	  8: help
	What now> 6
	           staged     unstaged path
	  1:        +1/-1      nothing index.html
	Review diff>> 1
	diff --git a/index.html b/index.html
	index 4d07108..4335f49 100644
	--- a/index.html
	+++ b/index.html
	@@ -16,7 +16,7 @@ Date Finder

	 <p id="out">...</p>

	-<div id="footer">contact : support@github.com</div>
	+<div id="footer">contact : email.support@github.com</div>

	 <script type="text/javascript">

With these basic commands we can use the interactive add mode to deal with our staging area a little more easily.

### Staging Patches ###

It is also possible for Git to stage certain parts of files and not the rest of it.  For example, if we had made two changes to our simplegit.rb file here and wanted to stage just one of them and not the other one, it turns out this is very easy to do in Git.  From the interactive prompt, type '5' or 'p' for 'patch'.  It will ask you which files you would like to patch stage, then for each section of the selected files, it will display each hunk of the file diff and ask you if you would like to stage them, one by one.

	diff --git a/lib/simplegit.rb b/lib/simplegit.rb
	index dd5ecc4..57399e0 100644
	--- a/lib/simplegit.rb
	+++ b/lib/simplegit.rb
	@@ -22,7 +22,7 @@ class SimpleGit
	   end
 
	   def log(treeish = 'master')
	-    command("git log -n 25 #{treeish}")
	+    command("git log -n 30 #{treeish}")
	   end
 
	   def blame(path)
	Stage this hunk [y,n,a,d,/,j,J,g,e,?]? 

There are a lot of options you have at this point.  If you type '?' you can see a list of what all you can do.

	Stage this hunk [y,n,a,d,/,j,J,g,e,?]? ?
	y - stage this hunk
	n - do not stage this hunk
	a - stage this and all the remaining hunks in the file
	d - do not stage this hunk nor any of the remaining hunks in the file
	g - select a hunk to go to
	/ - search for a hunk matching the given regex
	j - leave this hunk undecided, see next undecided hunk
	J - leave this hunk undecided, see next hunk
	k - leave this hunk undecided, see previous undecided hunk
	K - leave this hunk undecided, see previous hunk
	s - split the current hunk into smaller hunks
	e - manually edit the current hunk
	? - print help

Generally, you will just type 'y' or 'n' if you want to stage each hunk, but staging all of them in certain files or skipping a hunk decision until later can be helpful options too.  So, if we stage one part of the file and leave unstaged another, our status output will look like this.

	What now> 1
	           staged     unstaged path
	  1:    unchanged        +0/-1 TODO
	  2:        +1/-1      nothing index.html
	  3:        +1/-1        +4/-0 lib/simplegit.rb

The status of the simplegit.rb file is pretty interesting.  It shows us that there are a couple of lines staged and a couple unstaged.  We have partially staged this file.  At this point, you can exit out of the interactive adding script and simply run `git commit` to commit the carefully staged files.

You actually don't need to be in interactive add mode to do the partial file staging - you can start this same script by simply typing `git add -p` on the command line. These tools help you craft your staging area with a little more precision and possibly a little more easily.

