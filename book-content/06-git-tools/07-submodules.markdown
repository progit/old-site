## Submodules ##

It often happens that in working on one project, you need to use another project from within it.  Perhaps it is a library that a third party developed or that you are developing seperately and using it in multiple parent projects.  A common issue arises in these scenarios that you want to be able to treat them like two seperate projects yet still be able to use one from within the other. 

Let's look at an example.  Let's say that we are developing a website and we're creating ATOM feeds and instead of writing our own ATOM generating code, we decide to use a library.  Now we're likely going to have to either include this code from a shared library like a CPAN install or Ruby gem, or simply copy the source code into our own project tree.  The issue with including the library is that it is difficult to customize it in any way and often more difficult to deploy it, since you need to make sure every client has that library available.  The issue with vendoring the code into your own project is that any custom changes that you make become difficult to merge when upstream changes become available.

Submodules is how Git addresses this issue. Submodules allow you to keep a Git repository as a subdirectory of another Git repository.  This lets you clone another repository into your project, then let you keep your commits seperate.

### Starting with Submodules ###

Let's say you want to add the Rack library (a Ruby web server gateway interface) to your project, possibly maintain your own changes to it but continue to merge in upstream changes.  The first thing you're going to want to do is clone the external repository into your subdirectory.  You add external projects as submodules with the `git submodule add` command.

	$ git submodule add git://github.com/chneukirchen/rack.git rack
	Initialized empty Git repository in /opt/subtest/rack/.git/
	remote: Counting objects: 3181, done.
	remote: Compressing objects: 100% (1534/1534), done.
	remote: Total 3181 (delta 1951), reused 2623 (delta 1603)
	Receiving objects: 100% (3181/3181), 675.42 KiB | 422 KiB/s, done.
	Resolving deltas: 100% (1951/1951), done.
	
Now you have the Rack project under a subdirectory named 'rack' within your project.  At this point you can actually go into that subdirectory, make changes, add your own writable remote repository to push your changes into, fetch and merge from the original repository and more.  If we run `git status` right after we add the submodule, we'll see two things

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   .gitmodules
	#	new file:   rack
	#

The first thing you'll notice is the `.gitmodules` file.  This is a configuration file that stores the mapping between the url of the project and the local subdirectory you've pulled it into.

	$ cat .gitmodules 
	[submodule "rack"]
		path = rack
		url = git://github.com/chneukirchen/rack.git

If you have multiple submodules, you will have multiple entries in this file.  It is important to note that this file is version controlled with the rest of your files, like your `.gitignore` file.  It is pushed and pulled with the rest of your project. This is how other people that clone this project know where to get the submodule projects from.

The other listing in the `git status` output is the 'rack' entry.  If you run `git diff` on that, you'll see something interesting.

	$ git diff --cached rack
	diff --git a/rack b/rack
	new file mode 160000
	index 0000000..08d709f
	--- /dev/null
	+++ b/rack
	@@ -0,0 +1 @@
	+Subproject commit 08d709f78b8c5b0fbeb7821e37fa53e69afcf433

Although 'rack' is a subdirectory in your working directory, Git sees it as a submodule and does not track it's contents when you're not in that directory.  Instead it records it as a particular commit from that repository.  When you make changes and commit in that subdirectory, the superproject will notice that the HEAD there has changed and record the exact commit you are currently working off of, so when others clone this project they can recreate the environment exactly. 

This is an important point with submodules - you record them as the exact commit they are at, you cannot record a submodule at 'master' or some other symbolic reference.  

So, when you commit, you'll see something like this

	$ git commit -m 'first commit with submodule rack'
	[master 0550271] first commit with submodule rack
	 2 files changed, 4 insertions(+), 0 deletions(-)
	 create mode 100644 .gitmodules
	 create mode 160000 rack

Notice the '160000' mode for the 'rack' entry.  That is a special mode in Git that basically means you're recording a _commit_ as a directory entry, rather than a subdirectory or a file. 

Now we can treat the 'rack' directory as a seperate project and then just update or superproject with a pointer to what the latest commit in that subproject is from time to time.  You can see that all the Git commands work independently in the two directories.

	$ git log -1
	commit 0550271328a0038865aad6331e620cd7238601bb
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Thu Apr 9 09:03:56 2009 -0700

	    first commit with submodule rack
	$ cd rack/
	$ git log -1
	commit 08d709f78b8c5b0fbeb7821e37fa53e69afcf433
	Author: Christian Neukirchen <chneukirchen@gmail.com>
	Date:   Wed Mar 25 14:49:04 2009 +0100

	    Document version change

### Cloning a Project with Submodules ###

So, now let's clone a project with a submodule in it.  When you receive a project with a submodule in it, you'll get the directories that contain the submodules, but none of the files yet.

	$ git clone git://github.com/schacon/myproject.git
	Initialized empty Git repository in /opt/myproject/.git/
	remote: Counting objects: 6, done.
	remote: Compressing objects: 100% (4/4), done.
	remote: Total 6 (delta 0), reused 0 (delta 0)
	Receiving objects: 100% (6/6), done.
	$ cd myproject
	$ ls -l
	total 8
	-rw-r--r--  1 schacon  admin   3 Apr  9 09:11 README
	drwxr-xr-x  2 schacon  admin  68 Apr  9 09:11 rack
	$ ls rack/
	$

So the 'rack' directory is there, but empty.  Now we have to run two commands, `git submodule init` to initialize our local configuration file, and `git submodule update` to fetch all the data from that project and checkout the appropriate commit listed in our superproject.

	$ git submodule init
	Submodule 'rack' (git://github.com/chneukirchen/rack.git) registered for path 'rack'
	$ git submodule update
	Initialized empty Git repository in /opt/myproject/rack/.git/
	remote: Counting objects: 3181, done.
	remote: Compressing objects: 100% (1534/1534), done.
	remote: Total 3181 (delta 1951), reused 2623 (delta 1603)
	Receiving objects: 100% (3181/3181), 675.42 KiB | 173 KiB/s, done.
	Resolving deltas: 100% (1951/1951), done.
	Submodule path 'rack': checked out '08d709f78b8c5b0fbeb7821e37fa53e69afcf433'

Now our 'rack' subdirectory is at the exact state that it was when we committed earlier.  Now if another developer makes changes to the rack code and commits and we pull that reference down and merge it in, we'll get something a bit odd.

	$ git merge origin/master
	Updating 0550271..85a3eee
	Fast forward
	 rack |    2 +-
	 1 files changed, 1 insertions(+), 1 deletions(-)
	[master*]$ git status
	# On branch master
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#	modified:   rack
	#

We merged in what is basically a change to the pointer for our submodule, but it does not actually update the code in the submodule directory, so it now looks like we have a dirty state in our working directory.  

	$ git diff
	diff --git a/rack b/rack
	index 6c5e70b..08d709f 160000
	--- a/rack
	+++ b/rack
	@@ -1 +1 @@
	-Subproject commit 6c5e70b984a60b3cecd395edd5b48a7575bf58e0
	+Subproject commit 08d709f78b8c5b0fbeb7821e37fa53e69afcf433

This is because the pointer we have for the submodule is not what is actually in the submodule directory.  To fix this, we have to run `git submodule update` again.

	$ git submodule update
	remote: Counting objects: 5, done.
	remote: Compressing objects: 100% (3/3), done.
	remote: Total 3 (delta 1), reused 2 (delta 0)
	Unpacking objects: 100% (3/3), done.
	From git@github.com:schacon/rack
	   08d709f..6c5e70b  master     -> origin/master
	Submodule path 'rack': checked out '6c5e70b984a60b3cecd395edd5b48a7575bf58e0'

You'll have to do this every time you pull a submodule change down in the main project.  It's a bit weird, but it does work.

One common problem with this happens when a developer makes a change locally in a submodule but does not push it to a public server.  Then they commit a pointer to that non-public state and push the superproject up.  When other developers try to run `git submodule update`, the submodule system will not be able to find the commit that is referenced, because it only exists on the first developers system at that point.  If that happens, you'll see an error like this.

	$ git submodule update
	fatal: reference is not a tree: 6c5e70b984a60b3cecd395edd5b48a7575bf58e0
	Unable to checkout '6c5e70b984a60b3cecd395edd5b48a7575bf58e0' in submodule path 'rack'

At that point, you just have to see who last changed the submodule:

	$ git log -1 rack
	commit 85a3eee996800fcfa91e2119372dd4172bf76678
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Thu Apr 9 09:19:14 2009 -0700

	    added a submodule reference I will never make public. hahahahaha!

Then you have to email that guy and yell at him.

### Superprojects ###

Sometimes developers want to get some combination of subdirectories of their large project depending on what team they are on.  This is common if you're coming from CVS or Subversion where you've defined a module, some collection of subdirectories, and want to keep this type of workflow. 

A good way to do this in Git is to make each of the subfolders a seperate Git repository, then create superproject Git repositories that have multiple submodules in them.  A benefit of this is that you can more specifically define the relationships between the projects with tags and branches in the superprojects.  

### Issues with Submodules ###

Submodules is not without it's hiccups, however.

First of all, you have to be relatively careful when working in the submodule directory.  When you run 'git submodule update', it's going to checkout the specific version of the project, but not within a branch.  This is called having a 'detached head' - it means that the HEAD file points directly to a commit, not to a symbolic reference.  The issue here is that you generally don't want to work in a 'detached head' environment because it's easy to lose changes.  In fact, if you do an initial `submodule update`, then commit in that submodule directory without creating a branch to work in, then run 'submodule update' again from the superproject without committing in the meantime, Git will overwrite your changes without telling you.

To avoid this, simply create a branch when you do work in a submodule directory with a `git checkout -b work` or something equivalent.  When you do the `submodule update` a second time it will still revert your work, but at least you'll have a pointer to get back to.

Switching branches with submodules in them can also be tricky.  If you create a new branch, add a submodule there and then switch back to a branch without that submodule, you'll still have the submodule directory there as an untracked directory.  

	$ git checkout -b rack
	Switched to a new branch "rack"
	$ git submodule add git@github.com:schacon/rack.git rack
	Initialized empty Git repository in /opt/myproj/rack/.git/
	...
	Receiving objects: 100% (3184/3184), 677.42 KiB | 34 KiB/s, done.
	Resolving deltas: 100% (1952/1952), done.
	$ git commit -am 'added rack submodule'
	[rack cc49a69] added rack submodule
	 2 files changed, 4 insertions(+), 0 deletions(-)
	 create mode 100644 .gitmodules
	 create mode 160000 rack
	$ git checkout master
	Switched to branch "master"
	$ git status
	# On branch master
	# Untracked files:
	#   (use "git add <file>..." to include in what will be committed)
	#
	#	rack/

You either have to move it out of the way or remove it, in which case you both have to clone it again when you switch back and you may also lose local changes or branches you had that you didn't push up.

The last main caveat that many people run into is in switching from subdirectories to submodules.  If you have been tracking files in your project and you want to move them out into a submodule, you have to be careful or Git will get a bit angry at you.  Let's assume that you have the rack files in a subdirectory of your project and you want to switch it to a submodule.  If you simply delete the subdirectory and then run `submodule add` it will yell at you.

	$ rm -Rf rack/
	$ git submodule add git@github.com:schacon/rack.git rack
	'rack' already exists in the index

So you have to unstage the rack directory first, then you can add the submodule.

	$ git rm -r rack
	$ git submodule add git@github.com:schacon/rack.git rack
	Initialized empty Git repository in /opt/testsub/rack/.git/
	remote: Counting objects: 3184, done.
	remote: Compressing objects: 100% (1465/1465), done.
	remote: Total 3184 (delta 1952), reused 2770 (delta 1675)
	Receiving objects: 100% (3184/3184), 677.42 KiB | 88 KiB/s, done.
	Resolving deltas: 100% (1952/1952), done.

However, now let's assume that we did that in a branch.  If you try to switch back to a branch where you haven't done that yet - where those files are still in the actual tree rather than a submodule, you'll get this error:

	$ git checkout master
	error: Untracked working tree file 'rack/AUTHORS' would be overwritten by merge.

You basically have to move the 'rack' submodule directory out of the way before you can switch branches to one that doesn't have it.

	$ mv rack /tmp/
	$ git checkout master
	Switched to branch "master"
	$ ls
	README	rack

Then when you switch back, you'll get an empty 'rack' directory that you can either run `git submodule update` to reclone, or you can move your `/tmp/rack` directory back into.

