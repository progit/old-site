## Revision Selection ##

Git allows you to specify specific commits or a range of commits in several different ways, which are not neccesarily obvious but are very helpful to know.

### Single Revisions ###

You can obviously refer to a commit by the SHA-1 hash that it is given, but there are more human friendly ways to refer to commits as well.  The following section will outline the various ways that you can refer to a single commit.

#### Short SHA ####

Git is smart enough to figure out what commit you meant to type if you just provide the first few characters, so long as the partial SHA-1 you provided is at least 4 characters long and unambiguous - that is, there is only one object in the current repository that begins with the partial SHA-1 provided.

For example, if you want to see a specific commit, let's say we run a `git log` command and identify the commit where we added certain functionality.

	$ git log
	commit 734713bc047d87bf7eac9674765ae793478c50d3
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Fri Jan 2 18:32:33 2009 -0800

	    fixed refs handling, added gc auto, updated tests
	
	commit d921970aadf03b3cf0e71becdaab3147ba71cdef
	Merge: 1c002dd... 35cfb2b...
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Thu Dec 11 15:08:43 2008 -0800

	    Merge commit 'phedders/rdocs'

	commit 1c002dd4b536e7479fe34593e72e6c6c1819e53b
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Thu Dec 11 14:58:32 2008 -0800

	    added some blame and merge stuff

In this case, let's choose `1c002dd...`.  If we wanted to `git show` that commit, the following commands are equivalent (assuming the shorter versions are actually unambiguous):

	$ git show 1c002dd4b536e7479fe34593e72e6c6c1819e53b
	$ git show 1c002dd4b536e7479f
	$ git show 1c002d

Git can actually figure out a short unique abbreviation for your SHA-1 values.  If you pass `--abbrev-commit` to the `git log` command, it will use shorter values, though still keeping them unique - it defaults to using 7 characters, but will make them longer if it needs to in order to keep it unambiguous.

	$ git log --abbrev-commit --pretty=oneline
	ca82a6d changed the verison number
	085bb3b removed unnecessary test code
	a11bef0 first commit

In most projects, 8 to 10 characters are more than enough to be unique within the project.  One of the largest Git projects, the Linux kernel, is beginning to need 12 characters out of the possible 40 to stay unique.

SIDEBAR: a short note about SHA-1

A lot of people become concerned at some point that they will, by random happenstance, have two objects in their repository that by coincidence hash to the same SHA-1 value. What then?

First of all, I'm not entirely positive what Git will do when you try to commit a blob that happens to collide.  It will likely see an object already in your Git database and will assume it was already written.  If you tried to check that out again at some point you would probably get the wrong data.  It is pretty difficult to test this - even getting these values artificially is time and resource intensive.

However, you should be aware of how ridiculously unlikely this scenario is.  The SHA-1 digest is 20 bytes or 160 bits.  The number of randomly hashed objects needed to ensure a 50% probability of a single collision is about 2^80 ( the formula for determining collision probability is p = (n(n-1)/2) * (1/2^160) ).  2^80 is 1.2 x 10^24 or 1 million billion billion.  It is 1,200 times the number of grains of sand on the earth.

To give you an idea of what it would actually take to get a SHA-1 collsion, if all 6.5 billion humans on earth were programming and every one of them was producing code the equivalent of the entire linux kernel history (1 million Git objects) per *second* and pushing all of that into one enormous Git repository, it would take 5 years until that repository had enough objects in it to have a 50% probability of a single SHA-1 object collision.  There is a much higher probability that every member of your programming team will be attacked and killed by wolves in unrelated incidents on the same night. 

ENDSIDEBAR

#### Branch References ####

The most straightforward way to specify a commit is if it has a branch reference pointed at it. Then you can then use a branch name in any Git command that expects a commit object or SHA-1 value.  For instance, if you wanted to show the last commit object on a branch, the following commands would be equivalent, assuming that the 'topic1' branch pointed to `ca82a6d`:

	$ git show ca82a6dff817ec66f44342007202690a93763949
	$ git show topic1

If you want to see which specific SHA a branch points to, or in fact if you want to see what any of these examples boil down to in terms of SHAs, you can use a Git plumbing tool called `rev-parse`.  You can see Chapter 9 for more information about plumbing tools, but basically it's a tool that exists for lower level operations and is not really designed to be used in day to day operations.  However, it can be really helpful sometimes to see what's really going on.  Here we can run `rev-parse` on our branch.

	$ git rev-parse topic1
	ca82a6dff817ec66f44342007202690a93763949

#### RefLog Shortnames ####

One of the things Git does in the background while you're working away is that it keeps a `reflog`, a log of where your HEAD and branch references have been for the last few months.

You can see your reflog by typing `git reflog`.

	$ git reflog
	734713b... HEAD@{0}: commit: fixed refs handling, added gc auto, updated
	d921970... HEAD@{1}: merge phedders/rdocs: Merge made by recursive.
	1c002dd... HEAD@{2}: commit: added some blame and merge stuff
	1c36188... HEAD@{3}: rebase -i (squash): updating HEAD
	95df984... HEAD@{4}: commit: # This is a combination of two commits.
	1c36188... HEAD@{5}: rebase -i (squash): updating HEAD
	7e05da5... HEAD@{6}: rebase -i (pick): updating HEAD

You can see that everytime your branch tip is updated for any reason, Git stores that information away for you in this temporary history. However, you can specify older commits with this data as well.  If you wanted to see what the 5th prior value of the HEAD of my repository was, you can use the '@{n}' reference that you see in the output of `reflog`.

	$ git show HEAD@{5}

You can also use it to see where a branch was some specific amount of time ago.  For instance, to see where our 'master' branch was yesterday, you can actually type:

	$ git show master@{yesterday}

That will show you where that branch tip was yesterday.  This will only work for data still in your reflog however, so you can't do it to look for commits older than a few months.

If you want to see reflog information with your normal log information to get a better idea of what is recorded in your reflog, you can run `git log -g`.

	$ git log -g master
	commit 734713bc047d87bf7eac9674765ae793478c50d3
	Reflog: master@{0} (Scott Chacon <schacon@gmail.com>)
	Reflog message: commit: fixed refs handling, added gc auto, updated 
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Fri Jan 2 18:32:33 2009 -0800

	    fixed refs handling, added gc auto, updated tests

	commit d921970aadf03b3cf0e71becdaab3147ba71cdef
	Reflog: master@{1} (Scott Chacon <schacon@gmail.com>)
	Reflog message: merge phedders/rdocs: Merge made by recursive.
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Thu Dec 11 15:08:43 2008 -0800

	    Merge commit 'phedders/rdocs'

It's important to note that the reflog information is strictly local - it is a log of what *you* have done in your repository.  The references will probably not be the same on someone else's copy of the repository and right after you initially clone a repository, you'll have almost no information in it.  Running `git show HEAD@{2.months.ago}` will only work if you cloned the project at least 2 months ago - if you cloned it 5 minutes ago you will get no results.

#### Ancestry References ####

The other main way you can specify a commit is via it's ancestry.  If you specify a '^' at the end of a reference, Git will resolve that to mean the parent of that commit.

If we take a look at the history of our project:

	$ git log --pretty=format:'%h %s' --graph
	* 734713b fixed refs handling, added gc auto, updated tests
	*   d921970 Merge commit 'phedders/rdocs'
	|\  
	| * 35cfb2b Some rdoc changes
	* | 1c002dd added some blame and merge stuff
	|/  
	* 1c36188 ignore *.gem
	* 9b29157 add open3_detach to gemspec file list

Then we can see the previous commit by specifying `HEAD^`, which means "the parent of HEAD".

	$ git show HEAD^
	commit d921970aadf03b3cf0e71becdaab3147ba71cdef
	Merge: 1c002dd... 35cfb2b...
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Thu Dec 11 15:08:43 2008 -0800

	    Merge commit 'phedders/rdocs'

We can also specify a number after the `^`, such as `d921970^2`, which means "the second parent of d921970".  This syntax is only useful for merge commits, which have more than one parent.  The first parent is the branch you were on when you merged and the second would be the commit on the branch that you merged in.

	$ git show d921970^
	commit 1c002dd4b536e7479fe34593e72e6c6c1819e53b
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Thu Dec 11 14:58:32 2008 -0800

	    added some blame and merge stuff

	$ git show d921970^2
	commit 35cfb2b795a55793d7cc56a6cc2060b4bb732548
	Author: Paul Hedderly <paul+git@mjr.org>
	Date:   Wed Dec 10 22:22:03 2008 +0000

	    Some rdoc changes
	
The other main ancestry specification is the `~`.  This also refers to the first parent, so `HEAD~` and `HEAD^` are equivalent.  The difference is when you specify a number.  `HEAD~2` means "the first parent of the first parent", or "the grandparent" - it just traverses the first parents that many times.  For example, in the history we listed out earlier, `HEAD~3` would be

	$ git show HEAD~3
	commit 1c3618887afb5fbcbea25b7c013f4e2114448b8d
	Author: Tom Preston-Werner <tom@mojombo.com>
	Date:   Fri Nov 7 13:47:59 2008 -0500

	    ignore *.gem

This can also be written as `HEAD^^^`, which again is the first parent of the first parent of the first parent.

	$ git show HEAD^^^
	commit 1c3618887afb5fbcbea25b7c013f4e2114448b8d
	Author: Tom Preston-Werner <tom@mojombo.com>
	Date:   Fri Nov 7 13:47:59 2008 -0500

	    ignore *.gem

You can also combine these syntaxes - you can get the second parent of the previous reference (assuming it was a merge commit) by using `HEAD~3^2`, and so on.

### Commit Ranges ###

So now that we can specify individual commits, let's see how to specify specific ranges of commits.  This is particularly useful with managing your branches - if you have a lot of branches, you can use range specifications to answer questions such as "what work is on this branch that I haven't merged into my main branch yet?".

#### Double Dot ####

The most common range specification is the 'double dot' syntax.  This basically asks Git to resolve a range of commits that are reachable from one commit but are not reachable from another.  For example, let's say we have a commit history that looks like this:

FIG: figure1

Now we want to see what is in our experiemnt branch that has not yet been merged into our master branch.  We can ask Git to show us a log of just those commits with `master..experiment` - that means "all commits reachable by 'experiment' that are not reachable by 'master'".  For the sake of brevity and clarity in these examples, I'll simply use the letters of the commit objects from the diagram in the order that they will display.

	$ git log master..experiemnt
	D
	C

If, on the other hand, we want to see the opposite - all commits in 'master' that are not in 'experiment', we can simply reverse the branch names. `experiment..master` will show us everything in master not reachable from experiment.

	$ git log experiment..master
	F
	E

This is really useful if you want to keep the experiment branch up to date and you want to preview what you are about to merge in.  Another very frequent use of this is to see what you're about to push to a remote.
	
	$ git log origin/master..HEAD

That command will show you any commits you have in your current branch that are not in the 'master' branch on your 'origin' remote.  If you ran a 'git push' and your current branch was tracking 'origin/master', those are the changes that are then going to be transferred to the server.

You can also leave off one side of the syntax to have Git assume HEAD. For example, I can get the same results as in the previous example by typing `git log origin/master..` - Git will substitute HEAD if one side is missing.

#### Multiple Points ####

The double dot syntax is useful as a shorthand, but perhaps you want to specify more than two branches to specify your revision, such as seeing what commits are in any of several branches that are not in the branch you're currently on.  Git allows you to do this by using either the '^' character or a `--not` before any reference you don't want to see reachable commits from.  So these three commands are equivalent

	$ git log refA..refB
	$ git log ^refA refB
	$ git log refB --not refA

This is nice because then you can specify more than two.  If you want to see all commits that are reachable from 'refA' or 'refB' but not by 'refC', you would type one of these:

	$ git log refA refB ^refC
	$ git log refA refB --not refC

This makes for a very powerful revision query system and should be helpful for you in figuring out what is in your branches.

#### Triple Dot ####

The last major range selection syntax is the 'triple dot' syntax, which specifies all the commit that are reachable by either of two references, but not by both of them.

FIG: fig1

So if we wanted to see what was in `master` or `experiment` but not any common references, we can run 

	$ git log master...experiment
	F
	E
	D
	C

Again, this would give you normal `log` output, but it will only show you the commit information for those four commits, in that order.  

A common switch to use with the `log` command in this case is the `--left-right` option, which shows you which side of the range each commit is in.  This really helps make this data more useful.

	$ git log --left-right master...experiment
	< F
	< E
	> D
	> C

With these tools, we can much more easily let Git know what commit or commits we want to inspect.

