## Remote Branches ##

Remote branches are references to the state of branches on your remote repositories.  They are really just local branches that you can't move. They are moved automatically whenever you do any network communication.  Remote branches are bookmarks to remind you where the branches of your remotes were the last time you connected to them.

DWP: What are remotes? I worry that you jump into this with slightly too little background for the reader.{: class=note}

They take the form of '(remote)/(branch)' so for instance if you wanted to see what the 'master' branch on your 'origin' remote looked like as of the last time you communicated with it, you would check the 'origin/master' branch.  If you were working on an issue with a partner and they pushed up a 'iss53' branch, you may have your own local 'iss53' branch, but the commit that the branch on the server points to is at 'origin/iss53'.

DWP: I don't quite understand this last paragraph. {: class=note}

sop: Maybe take another paragraph to better define what a remote is, and how they are named by shorthand?  {: class=note}

DWP: I agree. But is there going to be something on remotes in chapter 2 now? I don't currently understand how you access branches on a remote repository.{: class=note}

### Pushing ###

When you want to share a branch with the world, you'll need to push it up to a remote you have write access to.  Your local branches are not automatically syncronized to the remotes you write to. You have to explicitly push the branches you want to share.  That way you can do work that you don't want to share in private branches and push up only the topic branches you want to collaborate on.

If we have a branch named 'serverfix' that we want to work on with others, we can push it up the same way we pushed our first branch.  You can simply run 'git push (remote) (branch)'.

	$ git push origin serverfix
	Counting objects: 20, done.
	Compressing objects: 100% (14/14), done.
	Writing objects: 100% (15/15), 1.74 KiB, done.
	Total 15 (delta 5), reused 0 (delta 0)
	To git@github.com:schacon/simplegit.git
	 * [new branch]      serverfix -> serverfix

sop: I wonder if we should explain that serverfix is just short for refs/heads/serverfix:refs/heads/serverfix, which means take this local branch named serverfix and update the remote branch named serverfix.  it helps with weirder cases like when your local name doesn't match the remote name (you mention it later) or to understand delete (see my note below).  {: class=note}

Now the next time one of your collaborators fetches from the server, they will get a reference to where the server's version of 'serverfix' is under the remote branch 'origin/serverfix'.

	$ git fetch
	remote: Counting objects: 20, done.
	remote: Compressing objects: 100% (14/14), done.
	remote: Total 15 (delta 5), reused 0 (delta 0)
	Unpacking objects: 100% (15/15), done.
	From git@github.com:schacon/simplegit
	 * [new branch]      serverfix    -> origin/serverfix
	
It's important to note that when you do a fetch that brings down new remote branches, you do not automatically have local, editable copies of them.  In other words, in this case you do not have a new 'serverfix' branch, you only have a 'origin/serverfix' pointer that you cannot modify.

If you want to merge this work into your current working branch, you can just run 'git merge origin/serverfix'.  If you want your own 'serverfix' branch that you can work on, you can base it off your remote branch.

	$ git checkout -b serverfix origin/serverfix
	Branch serverfix set up to track remote branch refs/remotes/origin/serverfix.
	Switched to a new branch "serverfix"

This will give you a local branch that you can work on that starts where 'origin/serverfix' is at.  

### Tracking Branches ###

The other thing about checking out a local branch from a remote branch is that it will automatically create what is called a 'tracking branch'.  Tracking branches are local branches that have a direct relationship to a remote branch, so that if you are on it and type 'git push', Git will automatically know which server and branch to push to.  Also, 'git pull' will fetch all the remote references and then automatically merge in the tracked remote branch.

When you clone a repository, it will generally automatically create a 'master' branch that tracks 'origin/master'.  That's why 'git push' and 'git pull' work out of the box with no other arguments.  However, you can setup other tracking branches, ones that don't track branches on 'origin' and don't track the 'master' branch if you want.  The simple case is the example that we just saw, the other way of accomplishing the same thing is to use the '--track' option.

	$ git checkout --track origin/serverfix
	Branch serverfix set up to track remote branch refs/remotes/origin/serverfix.
	Switched to a new branch "serverfix"

sop: this shorthand first appeared in 1.6.1-rc1.  can we assume the reader has that? {: class=note}

If you want to setup a local branch with a different name than the name of the remote branch, you can easily use the first version with a different local branch name.

	$ git checkout -b sf origin/serverfix
	Branch sf set up to track remote branch refs/remotes/origin/serverfix.
	Switched to a new branch "sf"

Now your local branch 'sf' will automatically push to 'origin/serverfix' and pull from it.

### Deleting Remote Branches ###

If you are done with a remote branch - say you and your collaborators are complete with a feature and have merged it into your remotes 'master' branch (or whatever branch your stable codeline is in).  You can delete a remote branch with the rather obtuse syntax of 'git push remote :branch'.  So if we wanted to delete our 'serverfix' branch from the server, we would run the following:

	$ git push origin :serverfix
	To git@github.com:schacon/simplegit.git
	 - [deleted]         serverfix

Boom.  No more branch on our server.  You may want to dogear this page, because you _will_ need that command and you _will_ forget the syntax.

sop: the way i remember it is, i'm pushing nothing to the branch, and since the branch has nothing to store in it, it goes away.  but you didn't explain the refspec syntax of src:dst so that may not help the reader.  {: class=note}

