## Working with Remotes ##

To be able to collaborate on any Git project, you'll need to know to manage your remote repositories.  Remote repositories are versions of your project that are hosted on the internet or network somewhere.  You can have several of them, each of which will generally be either read-only or read/write for you. Collaborating with others will involve managing these remote repositories and pushing and pulling data to and from them when you need to share work.

Managing these remote repositories includes knowing how to add remote repositories, remove remotes that are no longer valid, manage various remote branches and define them as being tracked or not, and more.  In this section we will cover these remote management skills.

### Showing Your Remotes ###

To see which remote servers you have, you can simply run the `git remote` command.  That will list out the shortnames of each remote handle that you have specified.  If you have cloned your repository, you should at least see 'origin' - that is the default name that Git gives to the server you cloned from.

	$ git clone git://github.com/schacon/ticgit.git
	Initialized empty Git repository in /private/tmp/ticgit/.git/
	remote: Counting objects: 595, done.
	remote: Compressing objects: 100% (269/269), done.
	remote: Total 595 (delta 255), reused 589 (delta 253)
	Receiving objects: 100% (595/595), 73.31 KiB | 1 KiB/s, done.
	Resolving deltas: 100% (255/255), done.
	$ cd ticgit
	$ git remote -v
	origin

You can also specify a '-v' which will show you the URL that Git has stored to expand that shortname to.

	$ git remote -v
	origin	git://github.com/schacon/ticgit.git

If you have more than one remote, it will simply list them all out.  For example, my Grit repository looks something like this.

	$ cd grit
	$ git remote -v
	bakkdoor	git://github.com/bakkdoor/grit.git
	cho45		git://github.com/cho45/grit.git
	defunkt	git://github.com/defunkt/grit.git
	koke		git://github.com/koke/grit.git
	origin	git@github.com:mojombo/grit.git

This means I can pull contributions from any of these users pretty easily, but you'll notice that only the 'origin' remote is an SSH URL, so it is the only one I can push to.

### Adding Remote Repositories ###

We have mentioned and given some demonstrations of adding remote repositories in previous sections, but here is how to do it explicitly.  To add a new remote Git repository as a shortname you can reference easily, simply run `git remote add [url]`.

	$ git remote
	origin
	$ git remote add pb git://github.com/paulboone/ticgit.git
	$ git remote -v
	origin	git://github.com/schacon/ticgit.git
	pb	git://github.com/paulboone/ticgit.git
	
Now you can use the string 'pb' on the command line in lieu of that whole URL.  For example, if we wanted to now fetch all the information that Paul has that we do not yet have in our repository, we can simply run `git fetch pb`.

	$ git fetch pb
	remote: Counting objects: 58, done.
	remote: Compressing objects: 100% (41/41), done.
	remote: Total 44 (delta 24), reused 1 (delta 0)
	Unpacking objects: 100% (44/44), done.
	From git://github.com/paulboone/ticgit
	 * [new branch]      master     -> pb/master
	 * [new branch]      ticgit     -> pb/ticgit

Now Pauls master branch is accessible locally as 'pb/master' - we could merge it into one of our branches, or we could checkout a local branch at that point if we wanted to inspect it.

### Fetching and Pulling from your Remotes ###

As we just saw, to get data from your remote projects, you can simply run 

	$ git fetch [remote-name]
	
That will go out to that remote project and pull down all the data that is at that remote project that you don't have yet.  After you do that, you should have references to all the branches that were on that remote which you can merge in or inspect at any time.  We'll go over what branches are and how to use them in the next chapter in much more detail.

If you cloned a repository, it will automatically add that remote repository under the name 'origin', so `git fetch origin` will fetch down any new work that has been pushed to that server since you cloned (or last fetched from) it.  It's important to note that the `fetch` command will simply pull the data to your local repository, it will not automatically merge it with any of your work or modify what you're currently working on in any way.  You'll have to manually merge it into your work when you're ready.

If you have a branch setup to track a remote branch (see the next section and Chapter 3 for more information on that), then you can use the `git pull` command to automatically fetch and then merge a remote branch into your current branch.  This may be an easier or more comfortable workflow for you and by default a `git clone` command will automatically setup your local 'master' branch to track the remote 'master' branch on the server you cloned from.  So running `git pull` will generally fetch data from the server you originally cloned from and automatically try to merge it into the code you're currently working on.

### Pushing to your Remotes ###

When you have your project at a point that you want to share it, you'll have to push it upstream.  The command for this is pretty simple, `git push [remote-name] [branch-name]`.  So if you want to push your 'master' branch to your 'origin' server (again, cloning will generally setup both of those names automatically for you), then you can simply run this to push your work back up to the server.

	$ git push origin master

This will only work if you cloned from a server that you have write access to and if nobody has pushed in the meantime.  If you and someone else cloned at the same time and then they push upstream and then you push upstream, your push will rightly be rejected.  You'll have to pull their work down first and incorporate it into yours before you'll be allowed to push. See Chapter 3 for more detailed information on how to push to remote servers.

### Inspecting a Remote ###
	
If you want to see more information about a particular remote, you can use the `git remote show [shortname]` command. If you run that command with a particular shortname such as 'origin', you'll get something like this:

	$ git remote show origin
	* remote origin
	  URL: git://github.com/schacon/ticgit.git
	  Remote branch merged with 'git pull' while on branch master
	    master
	  Tracked remote branches
	    master
        ticgit

It lists the URL for that remote repository, as well as the tracking branch information.  It helpfully will tell you fairly explicitly that if you are on the 'master' branch and run 'git pull', that it will automatically merge in the 'master' branch on the remote after it fetches all the remote references.  It will also list out all the remote references it has pulled down.

That is a simple example that you are likely to encounter.  Once you have been using Git more heavily, however, you may see much more information from 'git remote show'.

	$ git remote show origin
	* remote origin
	  URL: git@github.com:defunkt/github.git
	  Remote branch merged with 'git pull' while on branch issues
	    issues
	  Remote branch merged with 'git pull' while on branch master
	    master
	  New remote branches (next fetch will store in remotes/origin)
	    caching
	  Stale tracking branches (use 'git remote prune')
	    libwalker
	    walker2
	  Tracked remote branches
	    acl
	    apiv2
	    dashboard2
        issues
        master
	    postgres
	  Local branch pushed with 'git push'
	    master:master

You'll see that this command will also show you which branch will be automatically pushed when you run `git push`, it shows you which remote branches are on the server that you don't yet have, which remote branches you have that have been removed from the server, and multiple branches that are automatically merged when you run `git pull``.

### Removing and Renaming Remotes ###

If you want to rename a reference, in newer versions of Git you can run `git remote rename` to change the shortname of a remote.  For instance, if we want to rename 'pb' to 'paul', we can do that with 'git remote rename'

	$ git remote rename pb paul
	$ git remote
	origin
	paul
	
It is worth mentioning that this changes our remote branch names, too.  So what used to be referenced at 'pb/master' is now at 'paul/master'
	
If you want to remove a reference for some reason - you've moved the server or are no longer using a particular mirror, or perhaps a contributor is simply not contributing anymore, you can do that with 'git rm'.

	$ git remote rm paul
	$ git remote
	origin
