## Debugging with Git ##

Git also provides a couple of tools to help you debug issues in your project.  Since Git is designed to work with nearly any type of project, these tools are pretty generic, but can often be really helpful in hunting down a bug or culprit when things go wrong.

### File Annotation ###

If you track down a bug in your code and you want to know when it was introduced and why, file annotation is often your best tool.  This will show you what commit was the last to modify each line of any file.  So, if you see that a method in your code is buggy, you'll want to annotate the file with `git blame` to see when it was done.  In this example, we're going to use the `-L` option to limit the output to just lines 12 thorugh 22.

	$ git blame -L 12,22 simplegit.rb 
	^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 12)   def show(treeish = 'master')
	^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 13)     command("git show #{treeish}")
	^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 14)   end
	^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 15) 
	9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 16)   def log(treeish = 'master')
	79eaf55d (Scott Chacon  2008-04-06 10:15:08 -0700 17)     command("git log -n 25 #{treeish}")
	9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 18)   end
	9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 19) 
	42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 20)   def blame(path)
	42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 21)     command("git blame #{path}")
	42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 22)   end


The first thing to notice is that the first field is the partial SHA-1 of the commit that last modified that line. The next two fields are values extracted from that commit, the author name and the authored date of that commit, so you can easily see who and when that line was modified.  After that is the line number and the actual content of the file.  Also to note is the `^4832fe2` commit lines, which designates that those lines were in the original commit of this file.  That commit is when this file was first added to this project and those lines have been unchanged since.  This is a tad confusing, since now we've seen at least three different ways that Git uses the '^' to modify a commit SHA, but that is what it means here.

One of the other cool things about Git is that it does not track file renames explicitly.  It simply records the snapshots and then tries to figure out what was renamed implicitly, after the fact.  One of the interesting features of this is that you can ask it to figure out all sorts of code movement as well.  If you pass a `-C` to `git blame`, Git will analyze the file you're annotating and try to figure out where snippets of code within it originally came from if they were copied from elsewhere.  Recently I was refactoring a file named GITServerHandler.m into multiple files, one of them was GITPackUpload.m.  If I blame GITPackUpload.m with the `-C` option, I can actually see where sections of the code originally came from.

	$ git blame -C -L 141,153 GITPackUpload.m 
	f344f58d GITServerHandler.m (Scott Chacon 2009-01-04 18:59:04 -0800 141) 
	f344f58d GITServerHandler.m (Scott Chacon 2009-01-04 18:59:04 -0800 142) - (void) gatherObjectShasFromC
	f344f58d GITServerHandler.m (Scott Chacon 2009-01-04 18:59:04 -0800 143) {
	70befddd GITServerHandler.m (Scott Chacon 2009-03-22 20:02:27 -0700 144)         //NSLog(@"GATHER COMMI
	ad11ac80 GITPackUpload.m    (Scott Chacon 2009-03-24 18:32:50 +0100 145)         
	ad11ac80 GITPackUpload.m    (Scott Chacon 2009-03-24 18:32:50 +0100 146)         NSString *parentSha;
	ad11ac80 GITPackUpload.m    (Scott Chacon 2009-03-24 18:32:50 +0100 147)         GITCommit *commit = [g
	ad11ac80 GITPackUpload.m    (Scott Chacon 2009-03-24 18:32:50 +0100 148)         
	ad11ac80 GITPackUpload.m    (Scott Chacon 2009-03-24 18:32:50 +0100 149)         //NSLog(@"GATHER COMMI
	ad11ac80 GITPackUpload.m    (Scott Chacon 2009-03-24 18:32:50 +0100 150)         
	56ef2caf GITServerHandler.m (Scott Chacon 2009-01-05 21:44:26 -0800 151)         if(commit) {
	56ef2caf GITServerHandler.m (Scott Chacon 2009-01-05 21:44:26 -0800 152)                 [refDict setOb
	56ef2caf GITServerHandler.m (Scott Chacon 2009-01-05 21:44:26 -0800 153)                 

This is really useful, because normally you would get as the original commit the commit where you copied the code over because that is the first time you touched those lines in this file.  Git will tell you the original commit where you wrote those lines, even if it was in another file.

### Binary Search ###

Annotating a file helps if you know where the issue is to begin with.  If you don't even know what is breaking and there have been dozens or hundreds of commits since the last state where you know it worked, you will likely turn to `git bisect` to help you out.  `bisect` does a binary search through your commit history to help you identify which commit introduced an issue as quickly as possible.

Let's say you just pushed out a release of your code to a production environment and you're getting bug reports about something that was not happening in your development environment and you can't imagine why it's doing that.  You go back to your code and it turns out you can reproduce it, but you just can't figure out what is going wrong.  We can bisect it to find out.  First we want to run `git bisect start` to get things going, then `git bisect bad` to tell the system that the current commit we are on is broken.  Then we have to tell bisect when the last known good state was, so we run `git bisect good [good_commit]`.

	$ git bisect start
	$ git bisect bad
	$ git bisect good v1.0
	Bisecting: 6 revisions left to test after this
	[ecb6e1bc347ccecc5f9350d878ce677feb13d3b2] error handling on repo

What Git did was figure out that there were about 12 commits in between where you are now and the broken commit, and it checked out the middle one for you.  At this point you can run your test to see if the issue exists as of this commit.  If it does, then it was introduced sometime before this, if it is not here then the issue was introduced sometime after it.  It turns out that there is no issue here, so we tell Git that by typing `git bisect good` and continue on our journey.

	$ git bisect good
	Bisecting: 3 revisions left to test after this
	[b047b02ea83310a70fd603dc8cd7a6cd13d15c04] secure this thing

Now we're on another commit, halfway between the one we just tested and our bad commit.  We run our test again and we find that this commit is broken, so we tell Git that with `git bisect bad`.

	$ git bisect bad
	Bisecting: 1 revisions left to test after this
	[f71ce38690acf49c1f3c9bea38e09d82a5ce6014] drop exceptions table

We find that this commit is fine, and now Git has all the information it needs to determine where the issue was introduced.  It will tell us the SHA-1 of the first bad commit and show some of the commit information and which files were modified in that commit so we can figure out what happened that might have introduced this bug.

	$ git bisect good
	b047b02ea83310a70fd603dc8cd7a6cd13d15c04 is first bad commit
	commit b047b02ea83310a70fd603dc8cd7a6cd13d15c04
	Author: PJ Hyett <pjhyett@example.com>
	Date:   Tue Jan 27 14:48:32 2009 -0800

	    secure this thing

	:040000 040000 40ee3e7821b895e52c1695092db9bdc4c61d1730 f24d3c6ebcfc639b1a3814550e62d60b8e68a8e4 M	config

When you are finished, you'll want to run `git bisect reset` to reset your HEAD to where you were before you started this or else you'll end up in a weird state.
	
	$ git bisect reset
	
This is a really powerful tool that can help you check hundreds of commits for an introduced bug in minutes.  In fact, if you have a script that will exit 0 if the project is good or non-0 if the project is bad, you can fully automate `git bisect`.  First you again tell it the scope of the bisect by saying the known bad and good commits.  You can do this by listing them with the `bisect start` command if you want, listing the known bad commit first and the known good commit second.

	$ git bisect start HEAD v1.0
	$ git bisect run test-error.sh

That will run `test-error.sh` on each checked out commit automatically until it finds the first broken commit.  You can also run something like `make` or `make tests` or whatever you have that runs automated tests for you.
