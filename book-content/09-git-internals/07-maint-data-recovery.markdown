## Maintenance and Data Recovery ##

Occasionally you may have to do some cleanup - make a repository more compact, clean up an imported repository or recover lost work.  This section will cover some of these scenarios.

## Maintenance ##

DWP: Is this the right header level? I was expecting ###. {: class=note}

Git will automatically run a command called 'auto gc' occasionally.  Most of the time, this command will do nothing.  However, if there are too many loose objects (objects not in a packfile) or there are too many packfiles, then Git will launch a full fledged `git gc` command. The 'gc' stands for 'garbage collect' and it does a number of things - it will gather up all the loose objects and place them in packfiles, it will consolidate packfiles into one big packfile and it will remove objects that are not reachable from any commit and are a few months old.

You can run the 'auto gc' manually by running

	$ git gc --auto
	
which again will generally do nothing.  There have to be around 7000 loose objects or over 50 packfiles for it to fire up a `gc` command.  You can modify these limits with the `gc.auto` and `gc.autopacklimit` config settings respectively.

The other thing that `gc` will do is pack up your references into a single file.  Let's say we have the following branches and tags in our repository:

	$ find .git/refs -type f
	.git/refs/heads/experiment
	.git/refs/heads/master
	.git/refs/tags/v1.0
	.git/refs/tags/v1.1

If we run `git gc`, we will no longer have these files in the 'refs' directory, Git will move them for sake of efficiency into a file named `.git/packed-refs` that looks like this:

	$ find .git/refs -type f
	$ cat .git/packed-refs 
	# pack-refs with: peeled 
	cac0cab538b970a37ea1e769cbbde608743bc96d refs/heads/experiment
	ab1afef80fac8e34258ff41fc1b867c702daa24b refs/heads/master
	cac0cab538b970a37ea1e769cbbde608743bc96d refs/tags/v1.0
	9585191f37f7b0fb9444f35a9bf50de191beadc2 refs/tags/v1.1
	^1a410efbd13591db07496601ebc7a059dd55cfe9

DWP: Do you really need the find line there? {: class=note}

If you update a reference, it will not edit this file, but instead write a new file to 'refs/heads', so to get the appropriate SHA for a given reference, Git checks for that reference in the 'refs' directory, then will check the 'packed-refs' file as a fallback.  However, if you cannot find a reference in the 'refs' directory, it is probably in your 'packed-refs' file.

Notice the last line of the file, which begins with a '^'.  This means that the tag directly above it is an annotated tag and that line is the commit that the annotated tag points to.

## Data Recovery ##

At some point in your Git journey, you may accidentally lose a commit.  Generally this happens because you force delete a branch that had work on it that it turns out you wanted after all, or you hard reset a branch abandoning commits that you actually wanted something from.  Assuming this happens, how can you get your commits back?

Let's see an example.  We will hard reset our master branch in our test repository to an older commit and then recover the commits that we lost.  First, let's review where our repository is at this point:

	$ git log --pretty=oneline
	ab1afef80fac8e34258ff41fc1b867c702daa24b modified repo a bit
	484a59275031909e19aadb7c92262719cfcdf19a added repo.rb
	1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
	cac0cab538b970a37ea1e769cbbde608743bc96d second commit
	fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit

Now let's move our master branch back to the middle commit:

	$ git reset --hard 1a410efbd13591db07496601ebc7a059dd55cfe9
	HEAD is now at 1a410ef third commit
	$ git log --pretty=oneline
	1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
	cac0cab538b970a37ea1e769cbbde608743bc96d second commit
	fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
	
OK, so now we've effectively lost the top two commits - we have no branch from which those commits are reachable.  So, what we need to do is find the latest commit SHA and then add a branch that points to it.  The trick is finding that latest commit SHA - it's not like we've memorized it, right?

The quickest way is often using a tool called `git reflog`.  As you are working, Git is silently recording what your HEAD is at everytime you change it.  Every time you commit or change branches, the reflog is getting updated.  You can see where you have been at any time by simply running `git reflog`.

	$ git reflog
	1a410ef HEAD@{0}: 1a410efbd13591db07496601ebc7a059dd55cfe9: updating HEAD
	ab1afef HEAD@{1}: ab1afef80fac8e34258ff41fc1b867c702daa24b: updating HEAD

To inspect them, we'll have to run `git show` on each SHA here, but it turns out that the bottom one is actually the commit that we lost, so we can recover that commit by running :

	$ git branch recover ab1afef
	$ git log --pretty=oneline recover
	ab1afef80fac8e34258ff41fc1b867c702daa24b modified repo a bit
	484a59275031909e19aadb7c92262719cfcdf19a added repo.rb
	1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
	cac0cab538b970a37ea1e769cbbde608743bc96d second commit
	fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit

DWP: So that has made us a new branch based called 'recover' and with the head at ab1afe? {: class=note}

Now, let's say our loss was for some reason not in the reflog - we can simulate that by expiring the reflog.

	$ git reset --hard 1a410e
	$ rm -Rf .git/logs/

Since the reflog data is kept in the '.git/logs/' directory, we now effectively have no reflog.  So, how can we recover that commit at this point? One way is to use a utility called `git fsck` which checks your database for integrity.  If we run it with the `--full` option, it will show us all objects that are not pointed to by another object.  

	$ git fsck --full
	dangling blob d670460b4b4aece5915caf5c68d12f560a9fe3e4
	dangling commit ab1afef80fac8e34258ff41fc1b867c702daa24b
	dangling tree aea790b9a58f6cf6f2804eeac9f0abbe9631e4c9
	dangling blob 7108f7ecb345ee9d0084193f147cdad4d2998293

In this case we can see our missing commit there after 'dangling commit'.  At this point we could recover it the same way, by adding a branch that points to it.

## Removing Objects ##

There are a lot of great things about Git, but one feature that can cause issues is the fact that a Git clone will download the _entire_ history of the project, including every version of every file.  This is fine if the whole thing is source code, since Git is highly optimized to compress that data efficiently.  However, if someone at any point in the history of your project added a single huge file, every clone for all time in the future will be forced to download that large file, even if it was removed from the project in the very next commit.  Since it is reachable from the history, it will always be there.  

This is a huge problem when converting Subversion or Perforce repositories into Git, since you don't download the whole history in those systems this type of addition carries few consequences.  If you find you did an import from another system or otherwise find that your repository is much larger than it should be, this is how you can find and remove large objects from your repository.  

Be warned - this is destructive to your commit objects - it will rewrite every commit object upstream from the earliest tree you have to modify to remove a large file reference.  If you do this immediately after an import before anyone has started to base work on it, you're fine - otherwise you have to notify all contributors that they will have to rebase their work onto your new commits.

To demonstrate this, we will add a large file into our test repository, then remove it in the next commit, then we will find and remove it permenantly from our repository.  First let's add our large object to our history:

	$ curl http://kernel.org/pub/software/scm/git/git-1.6.3.1.tar.bz2 > git.tbz2
	$ git add git.tbz2
	$ git commit -am 'added git tarball'
	[master 6df7640] added git tarball
	 1 files changed, 0 insertions(+), 0 deletions(-)
	 create mode 100644 git.tbz2
	
Oops, we didn't want to add a huge tarball to our project, let's get rid of it:

	$ git rm git.tbz2 
	rm 'git.tbz2'
	$ git commit -m 'oops - removed large tarball'
	[master da3f30d] oops - removed large tarball
	 1 files changed, 0 insertions(+), 0 deletions(-)
	 delete mode 100644 git.tbz2

Now let's `gc` our database and see how much space we're using up:

	$ git gc
	Counting objects: 21, done.
	Delta compression using 2 threads.
	Compressing objects: 100% (16/16), done.
	Writing objects: 100% (21/21), done.
	Total 21 (delta 3), reused 15 (delta 1)

We can run a command called `count-objects` to quickly see how much space we're using:

	$ git count-objects -v
	count: 4
	size: 16
	in-pack: 21
	packs: 1
	size-pack: 2016
	prune-packable: 0
	garbage: 0

The `size-pack` entry is the size of our packfiles floored to the nearest kilobyte, so we're using 2 megs.  Before that last commit we were using closer to 2k, so clearly removing the file from our last commit did not remove it from our history - every time anyone clones this repository they will have to clone all 2 megs just to get this tiny project because I accidentally added a big file one time.  Let's get rid of it.

First, we have to find it - in this case we already know what file it is, but let's say that we didn't, how would we identify what file or files are taking up so much space?  If we run `git gc`, all the objects should be in a packfile and we should be able to identify the big objects by running `git verify-pack` and sorting on the third field, which is file size:

	$ git verify-pack -v .git/objects/pack/pack-3f8c0...bb.idx | sort -k 3 -n | tail -3
	e3f094f522629ae358806b17daf78246c27c007b blob   1486 734 4667
	05408d195263d853f09dca71d55116663690c27c blob   12908 3478 1189
	7a9eb2fba2b1811321254ac360970fc169ba2330 blob   2056716 2056872 5401

So there is our big object there at the bottom - 2 megs.  Let's find our what file that is.  We are going to use a command called `rev-list` that we use briefly in Chapter 7 as well.  If you pass `--objects` to `rev-list` it will not only list out all the commit SHAs but also the blob SHAs with the file paths associated with them.  We can use this to find what our blob was named.

	$ git rev-list --objects --all | grep 7a9eb2fb
	7a9eb2fba2b1811321254ac360970fc169ba2330 git.tbz2

So now we need to remove this file from all trees in our past.  We can see what commits modified this file pretty easily:

	$ git log --pretty=oneline -- git.tbz2
	da3f30d019005479c99eb4c3406225613985a1db oops - removed large tarball
	6df764092f3e7c8f5f94cbe08ee5cf42e92a0289 added git tarball
	
So we need to rewrite all the commits upstream from `6df76` to fully remove this file from our Git history.  We will use `filter-branch` to do this relatively easily.

	$ git filter-branch --index-filter 'git rm --cached --ignore-unmatch git.tbz2' -- 6df7640^..
	Rewrite 6df764092f3e7c8f5f94cbe08ee5cf42e92a0289 (1/2)rm 'git.tbz2'
	Rewrite da3f30d019005479c99eb4c3406225613985a1db (2/2)
	Ref 'refs/heads/master' was rewritten

DWP: So what do all the options in there do? {: class=note}

So now our history does not contain a reference to that file anymore.  However, our reflog and a new refs directory Git added when we did the filter-branch called 'original' still do, so we'll have to remove them and then repack the database.

	$ rm -Rf .git/refs/original
	$ rm -Rf .git/logs/
	$ git gc
	Counting objects: 19, done.
	Delta compression using 2 threads.
	Compressing objects: 100% (14/14), done.
	Writing objects: 100% (19/19), done.
	Total 19 (delta 3), reused 16 (delta 1)

Now let's see how much space we saved.

	$ git count-objects -v
	count: 8
	size: 2040
	in-pack: 19
	packs: 1
	size-pack: 7
	prune-packable: 0
	garbage: 0

Now our packed repository size is down to 7k, which is much better than 2 megs. You can see from the 'size' value that the big object is still in our loose objects, so it's not _gone_, but it will not be transferred on a push or subsequent clone, which is what is important.  If you really wanted to, you could remove the object completely by running `git prune --expire now`.
