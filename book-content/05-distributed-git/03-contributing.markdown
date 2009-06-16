## Contributing to a Project ##

Now you know what the different workflows are and you should have a pretty good grasp of fundamental Git usage.  In this section we will cover a few common patterns for contributing to a project.  

Now, the main difficulty with describing this is that there are a huge number of variations on how this is done.  Since Git is very flexible there are many ways that people can and do work together, so it is somewhat problematic to describe how you should contribute to a project - every project is going to be a bit different. Some of the variables involved are active contributor size, choosen workflow, your commit access and possibly external contribution method.

The first variable is *active contributor size*.  How many users are actively contributing code to this project and how often?  In many instances this will be just two or three developers with a few commits a day or possibly less for slightly dormant projects.  For really large companies or projects this could be in the thousands with dozens or even hundreds of patches coming in each day.  This is important because with more and more developers you run into more issues with making sure your code applies cleanly or can be easily merged. Changes you do submit may be rendered obsolete or severely broken by work that is merged in while you were working on it or while it was waiting to be approved or applied.  How can you keep your code consistently up to date and your patches valid?

The next variable is the *workflow* that is in use for the project.  Is it centralized with each developer having equal write access to the main codeline?  Does the project have a maintainer or integration manager that checks all of the patches?  Are all of the patches peer-reviewed and approved somehow?  Are you involved in that process?  Is there a lieutenant system in place and do you have to submit your work to them first?

The next issue is your *commit access*.  The workflow required in order to contribute to a project will be a lot different if you have write access to the project than if you don't.  If you do not have write access, how does the project prefer to accept contributed work? Do they even have a policy? How much work are you contributing at a time?  How often will you be contributing? 

All of these questions can effect how you contribute effectively to a project and what workflows are preferred or even available to you.  We'll cover aspects of each of these in a series of use cases, moving from simple to more complex - hopefully you will be able to construct whatever specific workflows you'll need in practice from these examples.

### Commit Guidelines ###

Before we start looking at the specific use cases, a quick note about commit messages.  Having a good guideline for creating commits and sticking to it makes working with Git itself and collaborating with others a lot easier.  The Git project provides a nice document that lays out a number of good tips for creating commits to submit patches from - you can read it in the Git source code at:

	Documentation/SubmittingPatches

First, you don't want to submit any whitespace errors.  Git provides an easy way to check for this - before you commit, run `git diff --check`, which will identify possible whitespace errors and list them out for you.  Here is an example, where I've replaced a red terminal color with 'X's.

	$ git diff --check
	lib/simplegit.rb:5: trailing whitespace.
	+    @git_dir = File.expand_path(git_dir)XX
	lib/simplegit.rb:7: trailing whitespace.
	+ XXXXXXXXXXX
	lib/simplegit.rb:26: trailing whitespace.
	+    def command(git_cmd)XXXX

If you run that before committing, you can tell if you're about to commit whitespace issues that may annoy other developers.

Next, try to make each commit a logically separate changeset.  If you can, try to make your changes digestable - don't code for a whole weekend on five different issues and then submit them all as one massive commit on Monday.  Even if you don't commit during the weekend, use the staging area on Monday to split up your work into at least one commit per issue with a useful message per commit.  If some of the changes modified the same file, try to use `git add --patch` to partially stage files (covered in detail in Chapter 6).  The project snapshot at the tip of the branch will be identical whether you do one commit or five as long as all the changes get added at some point, so try to make things easier on your fellow developers when they have to review your changes.  It also makes it easier to pull out or revert one of the changesets if you need to later.  Chapter 6 describes a number of useful Git tricks for rewriting history and interactively staging files - make use of these tools to help craft a clean and understandable history.

The last thing to keep in mind is the commit message itself.  Getting in the habit of creating quality commit messages makes using and collaborating with Git a lot easier.  As a general rule, your messages should start with a single line, no more than about 50 characters describing the changeset concisely, followed by a blank line, followed by a more detailed explanation.  The Git project requires the more detailed explanation to include your motivation for the change, and to contrast its implementation with previous behaviour - this is a good guideline to follow.  It is also a good idea to use the imperative present tense in these messages.  In other words, use commands.  Instead of "I added tests for" or "Adding tests for", use "Add tests for".

Here is a nice template, originally written by Tim Pope at tpope.net.

	Short (50 chars or less) summary of changes

	More detailed explanatory text, if necessary.  Wrap it to about 72
	characters or so.  In some contexts, the first line is treated as the
	subject of an email and the rest of the text as the body.  The blank
	line separating the summary from the body is critical (unless you omit
	the body entirely); tools like rebase can get confused if you run the
	two together.

	Further paragraphs come after blank lines.

	 - Bullet points are okay, too

	 - Typically a hyphen or asterisk is used for the bullet, preceded by a
	   single space, with blank lines in between, but conventions vary here

	 - Use a hanging indent


DWP: I'm not sure what the last suggestion (use a hanging indent) is actually sugesting. Use a hanging indent where? {: class=note}

If all of your commit messages look like this, things are going to be a lot easier for you and the developers you work with.  The Git project itself has very well formatted commit messages - I encourage you to run `git log --no-merges` there to see what a nicely formatted project commit history looks like.

In the following examples, and indeed throughout most of this book, for the sake of brevity I do not format messages nicely like this but instead use the `-m` option to `git commit`.  Do as I say, not as I do.

### Private Small Team ###

The simplest setup that you're likely to encouter is a private project with one or two other developers.  By 'private', I mean closed source - not read accessible to the outside world.  You and the other developers all have push access to the repository.

In this environment you can basically follow a similar workflow to what you might do when using Subversion or another centralized system.  You still get the advantages of things like offline committing and vastly simpler branching and merging, but the workflow itself can be very similar - the main difference being that merges happen client side rather than on the server at commit time.

So let's see what two developers just starting to work together with a shared repository might look like. The first developer, John, clones the repository, makes a change and commits locally.  I'll be replacing the protocol messages with '...' in these examples to shorten them somewhat.

	# John's Machine
	$ git clone john@githost:simplegit.git
	Initialized empty Git repository in /home/john/simplegit/.git/
	...
	$ cd simplegit/
	$ vim lib/simplegit.rb 
	$ git commit -am 'removed invalid default value'
	[master 738ee87] removed invalid default value
	 1 files changed, 1 insertions(+), 1 deletions(-)
	
The second developer, Jessica, then does the same thing - clones the repository and commtis a change.

	# Jessica's Machine
	$ git clone jessica@githost:simplegit.git
	Initialized empty Git repository in /home/jessica/simplegit/.git/
	...
	$ cd simplegit/
	$ vim TODO 
	$ git commit -am 'add reset task'
	[master fbff5bc] add reset task
	 1 files changed, 1 insertions(+), 0 deletions(-)

Now Jessica pushes her work up to the server.

	# Jessica's Machine
	$ git push origin master
	...
	To jessica@githost:simplegit.git
	   1edee6b..fbff5bc  master -> master

Now John tries to push his change up too.

	# John's Machine
	$ git push origin master
	To john@githost:simplegit.git
	 ! [rejected]        master -> master (non-fast forward)
	error: failed to push some refs to 'john@githost:simplegit.git'

John is not allowed to push because Jessica has pushed in the meantime.  This is especially important to understand if you are used to Subversion, because you'll notice that they did not edit the same file.  While Subversion will automatically do this merge on the server if different files are edited, in Git we'll have to merge the commits locally.  Now John will have to fetch Jessica's changes and merge them in before he will be allowed to push.

	$ git fetch origin
	...
	From john@githost:simplegit
	 + 049d078...fbff5bc master     -> origin/master

So at this point John's local repository looks something like this:

FIG: simplea-1

John has a reference to the changes Jessica pushed up, but has to merge them into his own work before he is allowed to push.

	$ git merge origin/master
	Merge made by recursive.
	 TODO |    1 +
	 1 files changed, 1 insertions(+), 0 deletions(-)
	
The merge went smoothly - John's commit history now looks like this:

FIG: simplea-2

Now John can test his code to make sure it still works properly and then he can push his new merged work up to the server.

	$ git push origin master
	...
	To john@githost:simplegit.git
	   fbff5bc..72bbc59  master -> master

Finally, John's commit history looks like this:

FIG: simplea-3

In the meantime, Jessica has been working on a topic branch.  She created a topic branch called 'issue54' and has done three commits on that branch.  She hasn't fetched John's changes yet, so her commit history looks like this:

FIG: simpleb-1

Now Jessica wants to sync up with John, so she fetches.  

	# Jessica's Machine
	$ git fetch origin
	...
	From jessica@githost:simplegit
	   fbff5bc..72bbc59  master     -> origin/master

That pulls down the work that John has pushed up in the meantime and her history now looks like this:

FIG: simpleb-2

Now Jessica thinks her topic branch is ready, but she wants to know what she will have to merge her work into so that she can push.  She can run `git log` to find that out.

	$ git log --no-merges origin/master ^issue54
	commit 738ee872852dfaa9d6634e0dea7a324040193016
	Author: John Smith <jsmith@example.com>
	Date:   Fri May 29 16:01:27 2009 -0700

	    removed invalid default value


Now Jessica can merge her topic work into her master branch, then merge John's work ('origin/master') into her master branch, and then push back to the server again.  First, she switches back to her master branch to integrate all of this work.

	$ git checkout master
	Switched to branch "master"
	Your branch is behind 'origin/master' by 2 commits, and can be fast-forwarded.

Now she can either merge 'origin/master' or 'issue54' first - they are both upstream so the order doesn't really matter - the end snapshot should be identical no matter which order she does it, only the history will be slightly different.  She chooses to merge in 'issue54' first.

sop: I'd merge origin/master first.   {: class=note}

	$ git merge issue54
	Updating fbff5bc..4af4298
	Fast forward
	 README           |    1 +
	 lib/simplegit.rb |    6 +++++-
	 2 files changed, 6 insertions(+), 1 deletions(-)
	
No problems and we can see it was a simple fast forward.  Now to merge in John's work ('origin/master').

	$ git merge origin/master
	Auto-merging lib/simplegit.rb
	Merge made by recursive.
	 lib/simplegit.rb |    2 +-
	 1 files changed, 1 insertions(+), 1 deletions(-)

Everything merged cleanly, so now Jessica's history looks like this:

FIG: simpleb-3

Now 'origin/master' is reachable from her 'master' branch, so she should be able to successfully push (assuming John has not pushed again in the meantime).

	$ git push origin master
	...
	To jessica@githost:simplegit.git
	   72bbc59..8059c15  master -> master

Now each developer has committed a few times and merged each others work successfully.

FIG: simpleb-4

So that is really one of the simplest workflows.  You work for a while, generally in a topic branch, and merge into your master branch when it's ready to be integrated.  When you want to share that work, merge it into your own master branch, then fetch and merge 'origin/master' if it has changed and finally push to the master branch on the server.  The general sequence is something like:

FIG: sequence1

### Private Managed Team ###

In this next scenario, we'll look at contributor roles in a larger private group.  Here we'll look at how to work in an environment where small groups collaborate on features and then those team based contributions are integrated by a another party.

Let's say that John and Jessica are working together on one feature, while Jessica and Josie are working on a second.  In this case the company is using a type of integration manager workflow where the work of the individual groups is integrated only by certain engineers and the master branch of the main repo can be updated only by those engineers.  In this scenario, all work is done in team based branches and then pulled together by the integrators later.

Let's follow Jessica's workflow as she works on her two features, collaborating in parallel with two different developers in this environment.  Assuming she already has her repository cloned, she decides to work on 'featureA' first.  She will create a new branch for the feature and do some work on it there.

	# Jessica's Machine
	$ git checkout -b featureA
	Switched to a new branch "featureA"
	$ vim lib/simplegit.rb
	$ git commit -am 'add limit to log function'
	[featureA 3300904] add limit to log function
	 1 files changed, 1 insertions(+), 1 deletions(-)

At this point she needs to share this with John, so she'll push her featureA branch commits up to the server.  Jessica does not have push access to the 'master' branch, only the integrators do, so she'll have to push to another branch in order to collaborate with John.

	$ git push origin featureA
	...
	To jessica@githost:simplegit.git
	 * [new branch]      featureA -> featureA

Jessica now emails John to tell him that she pushed some work into a branch named 'featureA' and he can look at it now. While she waits for feedback from John, Jessica decides to start working on 'featureB' with Josie.  To begin, she starts a new feature branch, basing it off the server's 'master' branch.

	# Jessica's Machine
	$ git fetch origin
	$ git checkout -b featureB origin/master
	Switched to a new branch "featureB"

Now Jessica makes a couple of commits on the 'featureB' branch.

	$ vim lib/simplegit.rb
	$ git commit -am 'made the ls-tree function recursive'
	[featureB e5b0fdc] made the ls-tree function recursive
	 1 files changed, 1 insertions(+), 1 deletions(-)
	$ vim lib/simplegit.rb
	$ git commit -am 'add ls-files'
	[featureB 8512791] add ls-files
	 1 files changed, 5 insertions(+), 0 deletions(-)

Jessica's repository looks like this now:

	FIG: simplec-1

Now she's ready to push her work up, but gets an email from Josie that a branch with some initial work on it was already pushed to the server as 'featureBee'.  Jessica will first need to merge those changes in with her own before she can push to the server. She can then fetch Josie's changes down with 'git fetch':

	$ git fetch origin
	...
	From jessica@githost:simplegit
	 * [new branch]      featureBee -> origin/featureBee

Jessica can now merge this into the work she did with 'git merge': 

	$ git merge origin/featureBee
	Auto-merging lib/simplegit.rb
	Merge made by recursive.
	 lib/simplegit.rb |    4 ++++
	 1 files changed, 4 insertions(+), 0 deletions(-)

Now there is a bit of a problem - she needs to push the merged work in her 'featureB' branch to the 'featureBee' branch on the server.  That can be done by specifying the local branch followed by a ':' followed by the remote branch to the 'git push' command.

	$ git push origin featureB:featureBee
	...
	To jessica@githost:simplegit.git
	   fba9af8..cd685d1  featureB -> featureBee

This is called a 'refspec' - see Chapter 9 for more detailed discussion of Git refspecs and different things that you can do with them.

Now John emails Jessica to say he's pushed some changes to the 'featureA' branch and asks her to verify them.  She can simply run a 'git fetch' to pull down those changes.

	$ git fetch origin
	...
	From jessica@githost:simplegit
	   3300904..aad881d  featureA   -> origin/featureA

Then she can see what has been changed with `git log`.

	$ git log origin/featureA ^featureA
	commit aad881d154acdaeb2b6b18ea0e827ed8a6d671e6
	Author: John Smith <jsmith@example.com>
	Date:   Fri May 29 19:57:33 2009 -0700

	    changed log output to 30 from 25

Finally, she merges John's work into her own 'featureA' branch.

	$ git checkout featureA
	Switched to branch "featureA"
	$ git merge origin/featureA
	Updating 3300904..aad881d
	Fast forward
	 lib/simplegit.rb |   10 +++++++++-
 	1 files changed, 9 insertions(+), 1 deletions(-)

Jessica wants to tweak something, so she commits again and then pushes this back up to the server.

	$ git commit -am 'small tweak'
	[featureA ed774b3] small tweak
	 1 files changed, 1 insertions(+), 1 deletions(-)	
	$ git push origin featureA
	...
	To jessica@githost:simplegit.git
	   3300904..ed774b3  featureA -> featureA

Jessica's commit history now looks something like this:

FIG: simplec-2

Now Jessica, Josie and John inform the integrators that the 'featureA' and 'featureBee' branches on the server are ready for integration into the mainline.  Once they integrate these branches into the mainline, a fetch will bring down the new merge commits.

FIG: simplec-3

Many groups switch to Git because of this ability to have multiple teams working in parallel, merging the different lines of work late in the process.  The ability for smaller subgroups of a team to collaborate together via remote branches without neccesarily having to involve or impede the entire team is a huge benefit of Git.

The sequence for the workflow we saw here was something like this:

FIG: sequence2

### Public Small Project ###

Contributing to public projects is a bit different.  Since you don't have the permissions to directly update branches on their project, you have to get the work to the maintainers somehow else.  This first example will describe contributing via forking on Git hosts that support easy forking.  The repo.or.cz and GitHub hosting sites both support this and many project maintainers may expect this style of contribution.  In the next section we'll deal with projects that prefer to accept contributed patches via email.

First, you'll probably want to clone the main repository, create a topic branch for the patch or patch series you are planning to contribute and do your work there.  The sequence will look basically like this:

	$ git clone (url)
	$ cd project
	$ git checkout -b featureA
	$ (work)
	$ git commit
	$ (work)
	$ git commit

You may want to use `rebase -i` here to squash your work down to a single commit, or rearrange the work in the commits to make the patch easier for the maintainer to review - see Chapter 6 for more information on interactive rebasing.

When your branch work is finished and you are ready to contribute it back to the maintainers, go to the original project page and click the 'fork' button, creating your own writable fork of the project.  You then need to add this new repository URL in as a second remote, we'll name it 'myfork'.

	$ git remote add myfork (url)
	
Now we want to push our work up to it.  It's probably easiest to simply push the remote branch you're working on up to your repository, rather than merging into your master branch and pushing that up.  The reason for this is that if the work is not accepted or is cherry picked, you don't have to rewind your master branch.  If the maintainers merge, rebase or cherry-pick your work, you will get it back via pulling from their repository eventually anyhow.

	$ git push myfork featureA

Now that your work has been pushed up to your fork, you need to notify the maintainer.  This is often called a 'pull request' and you can either generate it via the website itself - GitHub has a 'pull request' button that will automatically message the maintainer - or you can run the `git request-pull` command and email the output to the project maintainer manually.  

The request-pull command takes the base branch that you want your topic branch pulled into and the Git repository url you want them to pull from and will output a nice summary of all the changes you are asking them to pull in.  For instance, if Jessica wanted to send John a pull request and she had done two commits on the topic branch she just pushed up, she could run this:

	$ git request-pull origin/master myfork
	The following changes since commit 1edee6b1d61823a2de3b09c160d7080b8d1b3a40:
	  John Smith (1):
	        added a new function

	are available in the git repository at:

	  git://githost/simplegit.git featureA

	Jessica Smith (2):
	      add limit to log function
	      change log output to 30 from 25

	 lib/simplegit.rb |   10 +++++++++-
	 1 files changed, 9 insertions(+), 1 deletions(-)

sop: Yes, really, git request-pull knows how to expand a remote name to its URL.   {: class=note}

Then the output of that can be sent to the maintainer - it tells them where the work was branched from, summarizes the commits and tells them where to pull this work from.

On a project that you're not the maintainer for, it's generally easier to have a branch like 'master' always be tracking 'origin/master' and to do your work in topic branches that you can easily discard if they are rejected, or rebased if the tip has moved and they no longer apply cleanly.  For example, if you wanted to submit a second topic of work to the project, do not continue working on the topic branch you just pushed up - start over from the main repository's master branch.

	$ git fetch origin
	$ git checkout -b featureB origin/master
	$ (work)
	$ git commit
	$ git push myfork featureB
	$ (email maintainer)

Now each of your topics are contained within silos - similar to patch queues - that you can rewrite, rebase and modify without interfering or interdepending on each other.  

FIG: simpled-1

sop: Given the fetch that started the sequence, shouldn't branch featureB be based on commit 33009 in this diagram?  Move the fetch to after the push, then you can keep the diagram (and your later example) as-is. {: class=note}

Let's say that the project maintainer has pulled in a bunch of other patches and tried your first branch, but it no longer cleanly merges.  In this case we can try to rebase that branch on top of origin/master, resolve the conflicts for the maintainer, and then resubmit our changes.

	$ git checkout featureA
	$ git rebase origin/master
	$ git push myfork +featureA

FIG simpled-2

sop: Explain why I had to add the + in front of featureA there (force push due to rewind).   {: class=note}

Let's look at one more possible scenario - the maintainer has looked at work in your second branch and likes the concept but would like an implementation detail changed.  We will take this opportunity to move the work to be based off the current project's master branch, too.  We will start a new branch based off the current 'origin/master' branch, squash the 'featureB' changes there, resolve any conflicts, make the implementation change and then push that up as a new branch.

	$ git checkout -b featureBv2 origin/master
	$ git merge --no-commit --squash featureB
	$ (change implementation)
	$ git commit
	$ git push myfork featureBv2

The `--squash` option means that it will take all the work on the merged branch and squash it into one non-merge commit on top of the branch you're on.  The `--no-commit` option tells Git not to automatically record a commit.  This allows us to introduce all the changes from another branch and then make some more changes before recording the new commit.

Now you can send the maintainer a message that you've made the requested changes and they can find them in your 'topicBv2' branch.

FIG simpled-3

### Public Large Project ###

Many larger projects have established procedures for accepting patches - you'll need to check the specific rules for each project as they will differ a little.  However, many larger public projects accept patches via a developer mailing list, so we'll go over an example of that now.

The workflow itself is very similar to the previous use case - create topic branches for each patch series you work on - the difference is how you submit them to the project.  Instead of forking the project and pushing to your own writable version, we're going to generate email versions of each commit series  and email them to the developer mailing list.

	$ git checkout -b topicA
	$ (work)
	$ git commit
	$ (work)
	$ git commit

Now we have two commits that we want to send to the mailing list.  We will use `git format-patch` to generate the mbox formatted files that we can email to the list - it turns each commit into an email message with the first line of the commit message as the subject and the rest of the message plus the patch that the commit introduces as the body.  The nice thing about this is that applying a patch from an email generated with `format-patch` will preserve all the commit information properly, as we'll see more of in the next section where we apply these commits.

	$ git format-patch -M origin/master
	0001-add-limit-to-log-function.patch
	0002-changed-log-output-to-30-from-25.patch

The `format-patch` command will print out the names of the patch files that it created. The `-M` switch tells Git to look for renames. The files end up looking like this:

	$ cat 0001-add-limit-to-log-function.patch 
	From 330090432754092d704da8e76ca5c05c198e71a8 Mon Sep 17 00:00:00 2001
	From: Jessica Smith <jessica@example.com>
	Date: Sun, 6 Apr 2008 10:17:23 -0700
	Subject: [PATCH 1/2] add limit to log function

	Limit log functionality to the first 20
	
	---
	 lib/simplegit.rb |    2 +-
	 1 files changed, 1 insertions(+), 1 deletions(-)

	diff --git a/lib/simplegit.rb b/lib/simplegit.rb
	index 76f47bc..f9815f1 100644
	--- a/lib/simplegit.rb
	+++ b/lib/simplegit.rb
	@@ -14,7 +14,7 @@ class SimpleGit
	   end

	   def log(treeish = 'master')
	-    command("git log #{treeish}")
	+    command("git log -n 20 #{treeish}")
	   end

	   def ls_tree(treeish = 'master')
	-- 
	1.6.2.rc1.20.g8c5b.dirty

You can also edit these patch files to add more information for the email list that you don't want to show up in the commit message.  If you add text between the '---' line and the beginning of the patch (the 'lib/simplegit.rb' line) then developers can read it, but applying the patch will exclude it.

Now, to email this to a mailing list, you can either paste the file into your email program or send it via a command line program.  Pasting the text often causes formatting issues, especially with "smarter" clients that don't preserve newlines and other whitespace appropriately.  Luckily, Git provides a tool to help send properly formatted patches via IMAP, which may be a bit easier for you.  Here I'll demonstrate how to send a patch via Gmail, which happens to be the email agent that I use, though you can read detailed instructions for a number of mail programs at the end of the afore mentioned `Documentation/SubmittingPatches` file in the Git source code.

First you need to setup the 'imap' section in your ~/.gitconfig file.  You can set each value seperately with a series of `git config` commands or you can add them manually, but in the end your config file should look something like this:

	[imap]
	  folder = "[Gmail]/Drafts"
	  host = imaps://imap.gmail.com
	  user = user@gmail.com
	  pass = p4ssw0rd
	  port = 993
	  sslverify = false

If your IMAP server does not use SSL, the last two lines are probably not neccesary and the host value will be 'imap://' instead of 'imaps://'.

When that is setup, you can then use `git send-email` to place the patch series in the Drafts folder of the specified IMAP server.

	$ git send-email *.patch
	0001-added-limit-to-log-function.patch
	0002-changed-log-output-to-30-from-25.patch
	Who should the emails appear to be from? [Jessica Smith <jessica@example.com>] 
	Emails will be sent from: Jessica Smith <jessica@example.com>
	Who should the emails be sent to? jessica@example.com
	Message-ID to be used as In-Reply-To for the first email? y
	
Then it will spit out a bunch of log information looking something like this for each patch you are sending.

	(mbox) Adding cc: Jessica Smith <jessica@example.com> from line 'From: Jessica Smith <jessica@example.com>'
	OK. Log says:
	Sendmail: /usr/sbin/sendmail -i jessica@example.com
	From: Jessica Smith <jessica@example.com>
	To: jessica@example.com
	Subject: [PATCH 1/2] added limit to log function
	Date: Sat, 30 May 2009 13:29:15 -0700
	Message-Id: <1243715356-61726-1-git-send-email-jessica@example.com>
	X-Mailer: git-send-email 1.6.2.rc1.20.g8c5b.dirty
	In-Reply-To: <y>
	References: <y>

	Result: OK

At this point, you should be able to go to your Drafts folder, change the To field to be the mailing list your sending the patch to, possibly CC the maintainer or person responsible for that section and then send it off.

### Summary ###

In this section we've covered a number of common workflows for dealing with several very different types of Git projects you're likely to encounter and introduced a couple of new tools to help you manage this process. Next we'll see how to work the other side of the coin, maintaining a Git project - how to be a benevolent dictator or integration manager.
