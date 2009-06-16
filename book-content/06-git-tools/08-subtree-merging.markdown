## Subtree Merging ##

Now that we've seen the difficulties of the submodule system, let's look at an alternate way to solve the same problem.  When Git merges, it looks at what it has to merge together and then chooses an appropriate merging strategy to use.  When you're merging two branches it will use a 'recursive' strategy.  When you're merging more than two branches it will pick the 'octopus' strategy.  These strategies are automatically choosen for you because the 'recursive' stategy can handle pretty complex 3-way merge situations - for example, when there is more than one common ancestor - however, it can only handle merging two branches.  The 'octopus' merge can handle multiple branches but is not quite as robust and so is choosen as the default stategy if you're trying to merge more than two branches together.

However, there are other strategies you can choose as well.  One of them is called the 'subtree' merge and it can be used to deal with this subproject issue that we have.  So, let's see how to do the same 'rack' embedding that we did in the last section, but using subtree merges instead.

The idea of the subtree merge is that you have two projects and one of the projects maps to a subdirectory of the other one and vice versa.  Git is smart enough when you specify a subtree merge to figure out that one is a subtree of the other and merge appropriately - it's actually pretty amazing.

So, let's add the Rack application to our project. What we're going to do is to add the Rack project as a remote reference in our own project, then check it out into it's own branch.

	$ git remote add rack_remote git@github.com:schacon/rack.git
	$ git fetch rack_remote
	warning: no common commits
	remote: Counting objects: 3184, done.
	remote: Compressing objects: 100% (1465/1465), done.
	remote: Total 3184 (delta 1952), reused 2770 (delta 1675)
	Receiving objects: 100% (3184/3184), 677.42 KiB | 4 KiB/s, done.
	Resolving deltas: 100% (1952/1952), done.
	From git@github.com:schacon/rack
	 * [new branch]      build      -> rack_remote/build
	 * [new branch]      master     -> rack_remote/master
	 * [new branch]      rack-0.4   -> rack_remote/rack-0.4
	 * [new branch]      rack-0.9   -> rack_remote/rack-0.9
	$ git checkout -b rack_branch rack_remote/master
	Branch rack_branch set up to track remote branch refs/remotes/rack_remote/master.
	Switched to a new branch "rack_branch"

Now we have the root of the Rack project in our 'rack\_branch' branch and our own project in the 'master' branch.  If we checkout one and then the other, we can see that they have different project roots.

	$ ls
	AUTHORS		KNOWN-ISSUES	Rakefile	contrib		lib
	COPYING		README		bin		example		test
	$ git checkout master
	Switched to branch "master"
	$ ls
	README

So now we want to pull the rack project into our master project as a subdirectory.  You can do that in Git with `git read-tree`.  We'll learn more about `read-tree` and it's friends in Chapter 9, but for now it just reads the root tree of one branch into your current index and working directory.  We just switched back to our 'master' branch and so now we're going to pull the 'rack' branch into the 'rack' subdirectory of our 'master' branch main project.

	$ git read-tree --prefix=rack/ -u rack_branch
	
Now we can commit and it just looks like we have all the Rack files under that subdirectory - as though we just copied them in from a tarball.  What gets interesting though is that we can fairly easily merge changes from one of the branches to the other.  So, if the Rack project updates, we can pull in upstream changes by switching to that branch and pulling.

	$ git checkout rack_branch
	$ git pull

Then we can merge those changes back into our 'master' branch.  Now, you can simply use `git merge -s subtree` and it will work fine, but then Git will merge the histories together too, which you likely don't want.  To pull the changes in and prepopulate the commit message you can use the `--squash` and `--no-commit` options as well as the `-s subtree` strategy option.
	
	$ git checkout master
	$ git merge --squash -s subtree --no-commit rack_branch
	Squash commit -- not updating HEAD
	Automatic merge went well; stopped before committing as requested

Now we have all the changes from our Rack project merged in and ready to be committed locally.  We can also do the opposite - make changes in the 'rack/' subdirectory of our 'master' branch and then merge them into our 'rack\_branch' branch later to submit them to the maintainers or push them upstream.

Now, to get a diff between what we have in our 'rack/' subdirectory and the code in our 'rack\_branch' branch - to see if we need to merge them - cannot be done with the normal 'diff' command, so you'll have to run this `git diff-tree` with the branch you want to compare to.

	$ git diff-tree -p rack_branch

Or, to compare what is in your 'rack/' subdirectory with what the 'master' branch on the server was the last time you fetched, you can run

	$ git diff-tree -p rack_remote/master

## Summary ##

Now we have seen a number of more advanced tools that allow you to manipulate your commits and staging area a bit more corefully.  If you notice issues you should be able to easily figure out what commit introduced them, when and by whom.  If you want to use subprojects in your project we've gone over a few ways to accommodate those needs.  At this point you should be able to do most of the things in Git that you will need to be doing on the command line day to day and should feel comfortable with it.


