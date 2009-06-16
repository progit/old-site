## The Refspec ##

Throughout this book we are using pretty simple mappings from remote branches to local references, but they can be more complex.

When you add a remote like this:

	$ git remote add origin git@github.com:schacon/simplegit-progit.git

Basically what that will do is add a section to your `.git/config` file specifying the name of the remote ('origin'), the url of the remote repository and the _refspec_ for fetching.

	[remote "origin"]
		url = git@github.com:schacon/simplegit-progit.git
		fetch = +refs/heads/*:refs/remotes/origin/*

The format of the refspec is an optional '+', followed by `<src>:<dst>`, where `<src>` is the pattern for references on the remote side and `<dst>` is where those references are going to be written locally.  The '+' tells Git to update the reference even if it is not a fast-forward.  
	
In the default case that is automatically written by a `git remote add` command, we can see that Git will fetch all of the references under 'refs/heads/' on the server and write them to 'refs/remotes/origin/' locally.  So if there is a 'master' branch on the server, you will be able to access the log of that branch locally via:

	$ git log origin/master
	$ git log remotes/origin/master
	$ git log refs/remotes/origin/master

They are all equivalent, since Git will expand each of them out to 'refs/remotes/origin/master'.

If you wanted Git instead to only pull down the master branch each time and not every other branch on the remote server, you can change the 'fetch' line to:

	fetch = +refs/heads/master:refs/remotes/origin/master

Now this is just the default refspec for `git fetch` for that remote.  If you want to do something one time, you can specify the refspec on the command line too.  If you wanted to pull the 'master' branch on the remote down to 'origin/mymaster' locally, you can run:

	$ git fetch origin master:refs/remotes/origin/mymaster

You can also specify multiple refspecs.  On the command line, you can pull down several branches like so:

	$ git fetch origin master:refs/remotes/origin/mymaster topic:refs/remotes/origin/topic
	From git@github.com:schacon/simplegit
	 ! [rejected]        master     -> origin/mymaster  (non fast forward)
	 * [new branch]      topic      -> origin/topic

In this case, our 'master' branch pull got rejected because it wasn't a fast forward reference.  You can override that by specifying the '+' in front of the refspec.

You can also specify multiple refspecs for fetching in your configuration file. If you wanted to always fetch the 'master' and 'experiment' branches, you can simply add two lines:

	[remote "origin"]
		url = git@github.com:schacon/simplegit-progit.git
		fetch = +refs/heads/master:refs/remotes/origin/master
		fetch = +refs/heads/experiment:refs/remotes/origin/experiment

You cannot use partial globs in the pattern, so this would be invalid:

	fetch = +refs/heads/qa*:refs/remotes/origin/qa*

However, you can use namespacing to accomplish something like that.  If you had a QA team that pushed a series of branches and you wanted to get the master branch and any of the QA teams branches but nothing else, you can use a config section like this:

	[remote "origin"]
		url = git@github.com:schacon/simplegit-progit.git
		fetch = +refs/heads/master:refs/remotes/origin/master
		fetch = +refs/heads/qa/*:refs/remotes/origin/qa/*

So if you have a complex workflow process that has a QA team pushing branches, developers pushing branches and integration teams pushing and collaborating on remote branches, you can namespace them pretty easily this way.  

### Pushing Refspec ###

It is nice that you can fetch namespaced references that way, but how does the QA team get their branches into a 'qa/' namespace in the first place?  You accomplish that by using refspecs to push.

If the QA team wants to push their master branch to 'qa/master' on the remote server, they can run:

	$ git push origin master:refs/heads/qa/master
	
If they want Git to do that automatically each time they run `git push origin`, they can add a 'push' value to their config file:

	[remote "origin"]
		url = git@github.com:schacon/simplegit-progit.git
		fetch = +refs/heads/*:refs/remotes/origin/*
		push = refs/heads/master:refs/heads/qa/master

DWP: May as well explain it. This means that a push from the local 'master' branch will go to the remote 'qa/master' branch. {: class=note}

### Deleting References ###

You can also use the refspec to delete references from the remote server by running something like:

	$ git push origin :topic

Since the refspec is `<src>:<dst>`, this is basically saying 'make the topic branch on the remote _nothing_', which deletes it.
