## Git and Subversion ##

Currently, the majority of open source development projects and a large number of corporate projects are using Subversion to manage their source code.  It is the most popular open source version control system out there and has been around for nearly a decade.  It is also very similar in many ways to CVS, which was the big boy of the source control world before that.

One of the really great features of Git is a bidirectional bridge to Subverison called `git svn`.  This tool allows you to use Git as a valid client to a Subversion server, so you can use all the great local features of Git and then push to a Subversion server as if you were using Subversion locally.  This means you can do local branching and merging, use the staging area, use rebasing and cherry-picking and all the other great features of Git, while your collaborators continue to work in thier dark and ancient ways.  It is a great way to sneak Git into the corporate environment and helping your fellow developers become more efficent while you lobby to get the infrastructure changed to support it fully.  It is the gateway drug to the DVCS world.

### git-svn ###

The base command in Git for all the Subversion bridging commands is `git svn`.  You will preface everything with that.  There are a pretty good number of commands that `git svn` will take, so we'll cover the common ones while going through a few small workflows.

Now, it's important to note that when using `git svn` you're interacting with Subversion, which is a system that is far less sophisticated than Git.  Though you can easily do local branching and merging, it's generally best to keep your history as linear as possible by rebasing your work and avoiding doing things like simultaneously interacting with a Git remote repository. 

Don't rewrite your history and try to push again, don't push to a parallel Git repository to collaborate with fellow Git developers at the same time.  Subversion can only have a single linear history and confusing it is very easy.  If you're working with a team of which some are using SVN and some are using Git, just make sure everyone is using the SVN server to collaborate, it will make your life easier.

### Setting Up ###

To demonstrate all of this functionality, I'm going to need a typical SVN repository that I have write access to.  If you want to copy these examples, you will have to make your own writeable copy of my test repository.  In order to do that easily, you'll have to use a tool called `svnsync` that comes with more recent versions of Subversion - it should be distributed with at least 1.4.  For these tests, I created a new repository on Google code and initialized `svnsync` to partially copy the `protobuf` project, which is a tool that encodes structured data for transmission.  To start, you call `svnsync` with the 'to' and 'from' repositories.

	svnsync init https://your-project.googlecode.com/svn/ http://schacon-test.googlecode.com/svn/ 

This sets up the properties to run the sync.  You can then actually clone the code by running 

	svnsync sync https://your-project.googlecode.com/svn/
	Committed revision 1.
	Copied properties for revision 1.
	Committed revision 2.
	Copied properties for revision 2.
	Committed revision 3.
	...

I should warn you that even with a relatively small repository like my copy of `protobuf`, which only has about 75 commits, this operation still takes close to an hour to accomplish.  Subversion has to basically clone one revision at a time and then push it back up to another server - it's ridiculously inefficent, but it's the only easy way to do this.
	

sop: Can we use a smaller project for the book example?  Readers shouldn't wait an hour for something, they will get bored quickly and stop reading!   {: class=note}

dwp: Agreed. There must be something we can use with fewer commits than that! {: class=note}

sop: I suggest just using a file:// URL rather than a googlecode url here.  People shouldn't be encouraged to abuse another hosting site just to test out an example in a book.  And file:// didn't take very long for my laptop to do the svnsync step.   {: class=note}

### Getting Started ###

So, now that we have a Subversion repository that we have write access to, we can go though a typical workflow. We're going to start off with the `git svn clone` command, which will import an entire Subversion repository into a local Git repository for you to start with.

	 git svn clone https://schacon-test.googlecode.com/svn/ -T trunk -b branches -t tags
	Initialized empty Git repository in /Users/schacon/projects/testsvnsync/svn/.git/
	Authentication realm: <https://schacon-test.googlecode.com:443> Google Code Subversion Repository
	Password for 'schacon': 
	r1 = b4e387bc68740b5af56c2a5faf4003ae42bd135c (trunk)
		A	m4/acx_pthread.m4
		A	m4/stl_hash.m4
	...
	r75 = d1957f3b307922124eec6314e15bcda59e3d9610 (trunk)
	Found possible branch point: https://schacon-test.googlecode.com/svn/trunk => https://schacon-test.googlecode.com/svn/branches/my-calc-branch, 75
	Found branch parent: (my-calc-branch) d1957f3b307922124eec6314e15bcda59e3d9610
	Following parent with do_switch
	Successfully followed parent
	r76 = 8624824ecc0badd73f40ea2f01fce51894189b01 (my-calc-branch)
	Checked out HEAD:
	  https://schacon-test.googlecode.com/svn/branches/my-calc-branch r76

sop: Fails if you use a file:// URL and your SVN repository is in the same directory as the current working directory, as git svn clone put the git repository *inside* the SVN repository.  Something to watch out for if you choose to switch to a file:// URL like I suggested above.   {: class=note}

This will run the equivalent of two commands - `git svn init` followed by `git svn fetch` on the URL that you provide.  Now, this can take quite a while.  On our test project, it only has around 75 commits and the codebase isn't that big, so it only takes a few minutes. However, what Git has to do is checkout each version one at a time and commit them individually.  For a project with hundreds or thousands of commits, this can literally take hour or even days to finish.
	
The `-T trunk -b branches -t tags` part tells Git that this Subversion repository follows the basic branching and tagging conventions.  If you name your trunk, branches or tags differently, you can change the options here.  Since this is so common, you can replace that whole part with `-s`, which means 'standard layout' and implies all those options.  So, the following command is equivalent:

	git svn clone https://schacon-test.googlecode.com/svn/ -s

At this point, you should have a valid Git repository that has even imported your branches and tags.

	$ git branch -a
	* master
	  my-calc-branch
	  tags/2.0.2
	  tags/release-2.0.1
	  tags/release-2.0.2
	  tags/release-2.0.2rc1
	  trunk

It is important to note how this tool namespaces your remote stuff differently.  When cloning a normal Git repository you'll get all the branches on that remote server available locally as something like `origin/[branch]`, namespaced by the name of the remote.  However, `git svn` assumes that you are not going to have multiple remotes and saves all it's references to points on the remote server with no namespacing.  If we look into the 'refs/remotes' directory in our Git directory, we can see that all our branches are not namespaced by the name of a remote.

sop: I would use `git show-ref` here instead.  Messing around in .git is better left for chapter 9.   {: class=note}

	$ find .git/refs/remotes -type f
	.git/refs/remotes/my-calc-branch
	.git/refs/remotes/tags/2.0.2
	.git/refs/remotes/tags/release-2.0.1
	.git/refs/remotes/tags/release-2.0.2
	.git/refs/remotes/tags/release-2.0.2rc1
	.git/refs/remotes/trunk

A normal 'refs/remotes' directory would look more like this:

	$ find .git/refs/remotes -type f
	.git/refs/remotes/gitserver/master
	.git/refs/remotes/origin/master
	.git/refs/remotes/origin/testing

We can see that we have two remote servers, one named 'gitserver' with a 'master' branch, and another named 'origin' with two branches, 'master' and 'testing'.

Also notice in our example of remote references imported from `git-svn` that tags are added in as remote branches, not as real Git tags.

### Committing Back to Subversion ###

Now that you have a working repository you can do some work on the project and push your commits back upstream, using Git effectively as a SVN client.  If we edit one of the files and commit it, we'll have a commit that exists in Git locally that does not exist on the Subversion server.

	$ git commit -am 'Adding git-svn instructions to the README'
	[master 97031e5] Adding git-svn instructions to the README
	 1 files changed, 1 insertions(+), 1 deletions(-)

So now we need to push our change upstream.  Notice already how this changes how you can work with Subversion - you can do several commits offline and then push them all at once to the Subversion server.  To push to a Subversion server, you run the `git svn dcommit` command.

	$ git svn dcommit
	Committing to https://schacon-test.googlecode.com/svn/trunk ...
		M	README.txt
	Committed r79
		M	README.txt
	r79 = 938b1a547c2cc92033b74d32030e86468294a5c8 (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

What this does is take all of the commits you have made on top of the Subversion server code and do a Subversion commit for each, then rewrites your local Git commit to include a unique identifier.  This is important because it means that the SHA-1 checksum for your commits will all change.  This is part of the reason why working with Git-based remote versions of your projects concurrently with a Subversion server is not a good idea.  If we look at the last commit, we can see the new `git-svn-id` that was added to our commit:

	$ git log -1
	commit 938b1a547c2cc92033b74d32030e86468294a5c8
	Author: schacon <schacon@4c93b258-373f-11de-be05-5f7a86268029>
	Date:   Sat May 2 22:06:44 2009 +0000

	    Adding git-svn instructions to the README

	    git-svn-id: https://schacon-test.googlecode.com/svn/trunk@79 4c93b258-373f-11de-be05-5f7a86268029

Notice that the SHA checksum that originally started with `97031e5` when we committed now begins with `938b1a5`.  This means that if you *do* want to push to both a Git server and a Subversion server, you have to push (`dcommit`) to the Subversion server first, because that action will actually change your commit data.

### Pulling in New Changes ###

If you are working with other developers, then at some point one of you will push and then the other one will try to push a change that conflicts and will be rejected until you merge their work in.  In `git svn` it looks like this:

	$ git svn dcommit
	Committing to https://schacon-test.googlecode.com/svn/trunk ...
	Merge conflict during commit: Your file or directory 'README.txt' is probably out-of-date: resource out of date; try updating at /Users/schacon/libexec/git-core/git-svn line 482

In order to resolve this, we can run `git svn rebase` which will pull down any changes on the server that we don't have yet and rebase any work that we have on top of what was on the server.

	$ git svn rebase
		M	README.txt
	r80 = ff829ab914e8775c7c025d741beb3d523ee30bc4 (trunk)
	First, rewinding head to replay your work on top of it...
	Applying: first user change

Now all of our work is on top of what is on the Subversion server, so we can successfully `dcommit`.

	$ git svn dcommit
	Committing to https://schacon-test.googlecode.com/svn/trunk ...
		M	README.txt
	Committed r81
		M	README.txt
	r81 = 456cbe6337abe49154db70106d1836bc1332deed (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

It is important to remember that unlike Git, where you have to merge upstream work that you don't yet have locally before you can push, `git svn` only makes you do that if the changes conflict.  If someone else pushes a change to one file and then you push a change to another file, your 'dcommit' will work fine.

	$ git svn dcommit
	Committing to https://schacon-test.googlecode.com/svn/trunk ...
		M	configure.ac
	Committed r84
		M	autogen.sh
	r83 = 8aa54a74d452f82eee10076ab2584c1fc424853b (trunk)
		M	configure.ac
	r84 = cdbac939211ccb18aa744e581e46563af5d962d0 (trunk)
	W: d2f23b80f67aaaa1f6f5aaef48fce3263ac71a92 and refs/remotes/trunk differ, using rebase:
	:100755 100755 efa5a59965fbbb5b2b0a12890f1b351bb5493c18 015e4c98c482f0fa71e4d5434338014530b37fa6 M	autogen.sh
	First, rewinding head to replay your work on top of it...
	Nothing to do.

This is important to remember, because the outcome will be a project state that didn't exist on either of your computers when you pushed.  If the changes are incompatible but not conflicting, you may get issues that are difficult to diagnose.  Yet another reason not to use Subversion if at all possible.

sop: I would phrase the last sentence more that this is an advantage of git, that you can fully test the state on your client system before publishing it, while in SVN, you can't ever be certain that the state immediately before commit and after commit are identical.   {: class=note}

You should also run this command to simply pull in changes from the Subversion server, even if you're not ready to commit yourself.  You can run `git svn fetch` to just grab the new data, but `git svn rebase` will do the fetch and then update your local stuff.

	$ git svn rebase
		M	generate_descriptor_proto.sh
	r82 = bd16df9173e424c6f52c337ab6efa7f7643282f1 (trunk)
	First, rewinding head to replay your work on top of it...
	Fast-forwarded master to refs/remotes/trunk.

Running `git svn rebase` every once in a while will make sure that your code is always up to date.  You will need to make sure that your working directory is clean when you run this, though.  If you have local changes, either stash your work, or temporarily commit it before running `git svn rebase` or it likely won't work properly.

sop: That doesn't sound good "won't work properly".  What happens here?  It i a fail-fast case, like anything else in git, it errors out and does nothing at all, until you safely store your changes with stash or commit.  Say that, rather than this scary message.   {: class=note}

### Git Branching Issues ###

If you have become comfortable with a Git workflow, you will likely be creating topic branches, doing work on them and then merging them in.  If you are pushing to a Subversion server via `git svn`, you may want to rebase your work onto a single branch each time and not merge them together.  The reason for to prefer rebase is that Subversion has a linear history and simply does not deal with merges like Git does, so `git svn` will follow only the first parent when converting the snapshots into Subversion commits.

If your history looks like this - you created a 'experiment' branch and did two commits and then merged them back into 'master', when you 'dcommit' you will see output like this:

	$ git svn dcommit
	Committing to https://schacon-test.googlecode.com/svn/trunk ...
		M	CHANGES.txt
	Committed r85
		M	CHANGES.txt
	r85 = 4bfebeec434d156c36f2bcd18f4e3d97dc3269a2 (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk
	COPYING.txt: locally modified
	INSTALL.txt: locally modified
		M	COPYING.txt
		M	INSTALL.txt
	Committed r86
		M	INSTALL.txt
		M	COPYING.txt
	r86 = 2647f6b86ccfcaad4ec58c520e369ec81f7c283c (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

It will work fine, except that when you look at your Git project history, it will not have rewritten either of the commits that you made on the 'experiment' branch - instead, all of those changes will just show up in the SVN version of the single merge commit. 

Then when someone else clones that work, all they will see is the merge commit with all the work squashed into it and none of the commit data about where it came from or when it was committed.

### Subversion Branching ###

Branching in Subversion is not the same as branching in Git and so if you can avoid using them much, it's probably best.  However, you can create and commit to branches in Subversion using `git svn`.

#### Creating a New SVN Branch ####

To create a new branch in Subversion you can simply run `git svn branch [branchname]`.

	$ git svn branch opera
	Copying https://schacon-test.googlecode.com/svn/trunk at r87 to https://schacon-test.googlecode.com/svn/branches/opera...
	Found possible branch point: https://schacon-test.googlecode.com/svn/trunk => https://schacon-test.googlecode.com/svn/branches/opera, 87
	Found branch parent: (opera) 1f6bfe471083cbca06ac8d4176f7ad4de0d62e5f
	Following parent with do_switch
	Successfully followed parent
	r89 = 9b6fe0b90c5c9adf9165f700897518dbc54a7cbf (opera)

That does the equivalent of an `svn copy trunk branches/opera` command in Subversion and operates on the Subversion server.  Importantly, it does not check you out into that branch, so if you commit at this point, it will go to 'trunk' on the server, not 'opera'.

#### Switching Active Branches ####

Git figures out what branch your `dcommit`s go to by looking for the tip of any of your Subversion branches in your history - you should only have one and it should be the last one with a `git-svn-id` in your current branch history.  If you want to switch to work on an active Subversion branch, you can make your 'master' branch switch to it by running `git reset`.

	$ git reset --hard opera

That will switch your 'master' branch to start `dcommit`ing to the 'opera' branch on the Subversion server.  When you want to switch back to working on `trunk` work, you can switch back with the same command (but with 'trunk' instead of 'opera').

Do not do this if you have uncommitted changes or local commits that have not been `dcommit`ed to the Subversion server or you will lose them.

sop: WTF?  Why not just encourage creating a git topic branch for the SVN branch like you do below.  Suggesting use of git reset --hard as a normal workflow is _W_R_O_N_G_.   {: class=note}

If you would like to be working on more than one branch simultaneously, you can setup local branches to `dcommit` to specific Subversion branches by just starting them at the imported Subversion commit for that branch.  If you wanted an 'opera' branch that you could work on seperately, you can just run 

	$ git branch opera remotes/opera
	
Now if you want to merge your 'opera' branch into 'trunk' (your 'master' branch) you can do so with a normal `git merge`, but you need to provide a descriptive commit message or the merge will simply say 'Merge branch opera' instead of something useful.

sop: But that doesn't merge history!  You need to make that clear here, that a merge like this is going to cause the history in opera to not become part of master, but the code will merge as a single massive SVN commit when you dcommit on master.  and doesn't the dcommit on master now rewrite and lose the merge?   {: class=note}

### Subversion Commands ###

The `git svn` toolset provides a number of commands to help you ease the transition to Git by providing some similar functionality to what you used to have in Subversion.  Here are a couple of commands that you have that give you what Subversion used to.

#### SVN Style History ####

If you are used to Subversion and want to see your history in SVN output style, you can run `git svn log` to view your commit history in SVN formatting.

	$ git svn log
	------------------------------------------------------------------------
	r87 | schacon | 2009-05-02 16:07:37 -0700 (Sat, 02 May 2009) | 2 lines

	autogen change

	------------------------------------------------------------------------
	r86 | schacon | 2009-05-02 16:00:21 -0700 (Sat, 02 May 2009) | 2 lines

	Merge branch 'experiment'

	------------------------------------------------------------------------
	r85 | schacon | 2009-05-02 16:00:09 -0700 (Sat, 02 May 2009) | 2 lines

	updated the changelog

There are two important things to know about `git svn log`.  It works offline, unlike the real `svn log` command which asks the Subversion server for the data.  The other thing is that it will only show you commits that have been committed up to the Subversion server.  Local Git commits that you haven't `dcommit`ed will not show up and commits that people have made to the Subversion server in the meantime will also not show up.  It's more like the last known state of the commits on the Subversion server.

#### SVN Annotation ####

Much like the `git svn log` command will simulate the `svn log` command offline, you can get the equivalent of `svn annotate` by running `git svn blame [FILE]`, getting output that looks like this:

	$ git svn blame README.txt 
	 2   temporal Protocol Buffers - Google's data interchange format
	 2   temporal Copyright 2008 Google Inc.
	 2   temporal http://code.google.com/apis/protocolbuffers/
	 2   temporal 
	22   temporal C++ Installation - Unix
	22   temporal =======================
	 2   temporal 
	79    schacon Committing in git-svn.
	78    schacon 
	 2   temporal To build and install the C++ Protocol Buffer runtime and the Protocol
	 2   temporal Buffer compiler (protoc) execute the following:
	 2   temporal 

Again, it does not show commits that you did locally in Git or that have been pushed to Subversion in the meantime.

#### SVN Server Information ####

You can also get the same sort of information that `svn info` gives you by running `git svn info`.

	$ git svn info
	Path: .
	URL: https://schacon-test.googlecode.com/svn/trunk
	Repository Root: https://schacon-test.googlecode.com/svn
	Repository UUID: 4c93b258-373f-11de-be05-5f7a86268029
	Revision: 87
	Node Kind: directory
	Schedule: normal
	Last Changed Author: schacon
	Last Changed Rev: 87
	Last Changed Date: 2009-05-02 16:07:37 -0700 (Sat, 02 May 2009)

This is like `blame` and `log` in that it runs offline and is only up to date as of the last time you communicated with the Subversion server.

#### Ignoring What Subversion Ignores ####

If you clone a Subversion repository that has svn:ignore properties set anywhere you will likely want to set corresponding `.gitignore` files so you don't accidentally commit files that you shouldn't.  `git svn` has two commands to help with this issue.  The first is `git svn create-ignore` and will automatically create corresponding `.gitignore` files for you so your next commit can include them.

The second command is `git svn show-ignore` which will print to stdout the lines you would need to put in a `.gitignore` file so you can redirect the output into your project exclude file.

	$ git svn show-ignore > .git/info/exclude

That way you don't litter the project with `.gitignore` files.  This is a good option if you are the only Git user on a Subversion team and they don't want  `.gitignore` files in the project.


### Git-Svn Summary ###

The `git svn` tools are useful if you are stuck with a Subversion server in the meantime or are otherwise in a development environment that neccessitates running a Subversion server.  You should consider it crippled Git, however, or you will hit issues in translation that may confuse you and your collaborators.  To stay out of trouble, try to follow these guidelines:

* Keep a linear Git history.  Rebase any work you do outside of your mainline branch back onto it, do not merge it in.

sop: Explain "linear git history" again for the reader.  Maybe "Keep a linear Git history, which does not contain merge commits made by `git merge`" as they may still not be fully familiar with that term.   {: class=note}

* Do not setup and collaborate on a seperate Git server.  Possibly set one up to speed up clones for new developers, but don't push anything to it that does not have a git-svn-id entry.  You may even want to add a `pre-receive` hook that checks each commit for one and rejects pushes that contain commits without one.

If you follow those guidelines, working with a Subversion server can be more bearable.  However, if it's possible to move to a real Git server, doing so can gain your team a lot more.

