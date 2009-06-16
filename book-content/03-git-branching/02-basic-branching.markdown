## Basic Branching and Merging

So, let's go through a pretty simple example of branching and merging with a workflow that we might use in the real world.  We'll go through the following steps.

* Do work on a website
* Create a branch for a new story we're working on
* Do some work in that branch
* A call comes in that another issue is critical and we need a hotfix
* We revert back to our production branch
* We create a branch to add the hotfix
* Once it is tested, we merge the hotfix branch and push to production
* We switch back to our original story and continue working

### Basic Branching 

So first, let's say we're working on our project and have a couple of commits already.  

![Figure 3.10](/images/branch-ex1.png)

We have decided we're going to work on issue #53 in whatever issue tracking system our company uses.  To be clear, Git isn't tied into any particular issue tracking system, but since that is a focused topic we want to work on, we'll create a new branch to do our work in. To create a branch and switch to it at the same time, we can run the 'git checkout' command with the '-b' switch.

	$ git checkout -b iss53
	Switched to a new branch "iss53"
	
which is equivalent to 

	$ git branch iss53
	$ git checkout iss53

![Figure 3.10a](/images/branch-ex1a.png)

Then we can work on our website and do some commits, which will move the 'iss53' branch forward, since we're on it.

	$ vim index.html
	$ git commit -a -m 'added a new footer [issue 53]'
	
![Figure 3.11](/images/branch-ex2.png)

Now we get the call that there is an issue with the website and we need to fix it right now.  Luckily we don't have to deploy our fix along with the 'iss53' changes that we've made and we don't have to put a lot of effort into reverting those changes before we can work on applying our fix to what is in production.  All we have to do is switch back to our 'master' branch.

sop: I strongly suggest checking that the directory is clean with `git status` or something.  Users are going to think that dirty changes stay with the old branch, and will run into problems when they haven't finished committing to the current iss53 branch. git stash or commit then later reset --soft HEAD^ are two different solutions, and probably too complex for this stage.  so maybe just say something about making sure your directory is clean with `git status` and if not commit everything for safekeeping is a good idea at this stage of the book.  {: class=note}

	$ git checkout master
	Switched to branch "master"
	
Now our project working directory is exactly the way it was before we started working on issue 53 and we can concentrate on our hotfix.  This is an important point to remember - Git resets your working directory to look like the snapshot of the commit the branch you check out points to.  It will add, remove and modify files automatically to make sure your working copy is what the branch looked like on your last commit to it.  

You should also note that if your working directory or staging area have uncommited changes that conflict with the branch you're checking out, Git will not let you switch branches.  It's best to have a clean working state when you switch branches.

sop: Oh, I see, you put the dirty state comment after the checkout, not before.  I'd move it before.  {: class=note}

Now, we have a hotfix to make. Let's create a 'hotfix' branch to do that work on until it's completed.

	$ git checkout -b 'hotfix'
	Switched to a new branch "hotfix"
	$ vim index.html
	$ git commit -a -m 'fixed the broken email address'
	[hotfix]: created 3a0874c: "fixed the broken email address"
	 1 files changed, 0 insertions(+), 1 deletions(-)

![Figure 3.12](/images/branch-ex3.png)

Now we can run our tests and make sure that it's what we want and go ahead and merge it back into our 'master' branch to deploy to production.  We do this with the 'git merge' command.

	$ git checkout master
	$ git merge hotfix
	Updating f42c576..3a0874c
	Fast forward
	 README |    1 -
	 1 files changed, 0 insertions(+), 1 deletions(-)

You'll notice the phrase 'Fast forward' in that merge.  Since the commit pointed to by the branch we merged in was a direct ancestor of the commit we are on, it simply moves the pointer forward.

sop: nope, language is wrong.  the commit we are merging is a strict superset of the branch we are on.  if superset is too technical, try the commit we are merging already contains the commit we have checked out.  {: class=note}

![Figure 3.13](/images/branch-ex4.png)

sop: wtf is this red coloring?  it doesn't make sense to me, even with the context of the text you have {: class=note}

So, now our change is in the snapshot of the commit pointed to by the 'master' branch and we can deploy our change.

![Figure 3.14](/images/branch-ex5.png)

Now that our super important fix is deployed, we can switch back to the work we were doing before we were interuppted and continue it.  However, first we will delete the 'hotfix' branch since we no longer need it anymore.  We can do that with the '-d' option to 'git branch'.

	$ git branch -d hotfix
	Deleted branch hotfix (3a0874c).

Now we can switch back to our work in progress branch on issue 53 and continue working on it.

	$ git checkout iss53
	Switched to branch "iss53"
	$ vim index.html
	$ git commit -a -m 'finished the new footer [issue 53]'
	[iss53]: created ad82d7a: "finished the new footer [issue 53]"
	 1 files changed, 1 insertions(+), 0 deletions(-)

![Figure 3.15](/images/branch-ex6.png)

sop: I think its worth pointing out at this point that our hotfix isn't in iss53.  If we need it, we would want to `git merge master` after the checkout in order to bring our iss53 branch up-to-date with the production code.  But if its not critical for iss53, we may want to defer doing the merge, to save a little typing, and to make the history graphs a little less verbose.  {: class=note}

### Basic Merging 

OK, now we've decided that our issue 53 work is complete and ready to be merged into our 'master' branch.  Merging in Git is just about as easy as branching and is also done entirely locally.  All we have to do is checkout the branch we wish to merge into and then run the 'git merge' command.

sop: You seem to forget you did a merge earlier for the hotfix branch? Yes, yes, its a fast-forward vs. a recursive merge, but the reader doesn't quite realize that distinction yet so he's wondering why merging was already done and now we're talking about merging and what did he miss? {: class=note}

DWP: Agreed. I think you could beef up the explanation earlier that that merge was an easy one, and then reiterate here why this one is harder. {: class=note}

	$ git checkout master
	$ git merge iss53
	Merge made by recursive.
	 README |    1 +
	 1 files changed, 1 insertions(+), 0 deletions(-)

Now instead of 'fast forward', since the commit on the branch we're on is not  a direct ancestor of the branch we're merging in, Git actually has to do some work.   What Git will do in this case is a simple 3 way merge of the two snapshots pointed to by the branch tips and the common ancestor of the two.

sop: This is really awkward to read.  But at least correct.  {: class=note}

sop: It may be worth pointing out that unlike other VCS tools such as CVS or SVN prior to version 1.5 that you do not need to remember the merge base yourself; git figures it out automatically during each merge, producing the correct result each time.  {: class=note}

![Figure 3.16](/images/branch-ex7.png)

sop: Huh.  What about marking the C2 common ancestor differently then the C4/C5 tips? {: class=note}

So instead of just moving the branch pointer forward, Git actually creates a new snapshot that resulted from the 3 way merge and automatically creates a new commit that points to it.  This is referred to as a 'merge commit' and is a special type of commit in that it has two parents and introduces no new work.

sop: I'd scratch the "introduces no new work" part.  Its only special because nparents > 1.  {: class=note}

DWP: I wasn't sure what "introduces no new work" meant either. {: class=note}

![Figure 3.17](/images/branch-ex8.png)

Now that our work is merged in, we have no need for the issue 53 branch and can now delete it and close the ticket in our ticket tracking system.

	$ git branch -d iss53
	
## Basic Merge Conflicts ##

Occasionally, this process will not go so smoothly.  If you changed the same part of the same file differently in the two branches you are merging together, Git will not be able to merge them cleanly.  If our issue 53 fix modified the same part of a file as the hotfix, we'll get a merge conflict that will look something like this:

	$ git merge iss53
	Auto-merging index.html
	CONFLICT (content): Merge conflict in index.html
	Automatic merge failed; fix conflicts and then commit the result.

Now Git has _not_ automatically created a new merge commit.  It has paused the process while you resolve the conflict.  If you want to see which files are unmerged at any point, you can run 'git status'.

	[master*]$ git status
	index.html: needs merge
	# On branch master
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#	unmerged:   index.html
	#

Anything that had merge conflicts and has not been resolved yet will be listed as 'unmerged'.  So, now we have to resolve the conflicts.  Git will add standard conflict resolution markers to the files that had conflicts in them, so you can just open them manually and resolve the conflicts.  Our file will have a section in it that looks something like this.

DWP: It may sound obvious, but I think you should say what you mean by manually resolve the conflicts - i.e. usually, choose the version that you want, or occasionally merge the two together. {: class=note}

	<<<<<<< HEAD:index.html
	<div id="footer">contact : email.support@github.com</div>
	=======
	<div id="footer">
	  please contact us at support@github.com
	</div>
	>>>>>>> iss53:index.html

When you resolve it, just run 'git add' on the file to mark it as resolved.  Staging the file is what resolves it.  

The other way you can see what has conflicted is to run 'git diff --merge' at this point.  That will show you a nice overview of what is being merged from each point of the 3 way merge. That is, for each conflicted file what came from the branch you're on, the branch you're merging in and the common ancestor.

	$ git diff --merge
	diff --cc index.html
	index 25d648a,4d07108..4335f49
	--- a/index.html
	+++ b/index.html
	@@@ -16,9 -16,7 +16,7 @@@ Date Finde
  
	  <p id="out">...</p>
  
	- <div id="footer">
	-   please contact us at support@github.com
	- </div>
	 -<div id="footer">contact : support@github.com</div>
	++<div id="footer">contact : email.support@github.com</div>
  
	  <script type="text/javascript">
    

sop: I wonder if git diff --merge is too technical here, and shouldn't just be deferred until a later time.  {: class=note}

DWP: If you keep this here I think you should describe the output a bit. {: class=note}

If you want to use a graphical tool to resolve this, you can run 'git mergetool' which will fire up an appropriate visual merge tool for you and walk you through the conflicts.

	$ git mergetool
	merge tool candidates: kdiff3 tkdiff xxdiff meld gvimdiff opendiff emerge vimdiff
	Merging the files: index.html

	Normal merge conflict for 'index.html':
	  {local}: modified
	  {remote}: modified
	Hit return to start merge resolution tool (opendiff):

If you want to use a different merge tool, you can see all the supported ones listed at the top after 'merge tool candidates'. After each file it will ask you if the merge was successful.  If you tell the script that it was, it will stage the file to mark it as resolved for you.

Now you can run 'git status' again to verify that all your conflicts have been resolved.

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	modified:   index.html
	#

Now you just type 'git commit' to finalize the merge commit.  The commit message will by default look something like this:

	Merge branch 'iss53'

	Conflicts:
	  index.html
	#
	# It looks like you may be committing a MERGE.
	# If this is not correct, please remove the file
	# .git/MERGE_HEAD
	# and try again.
	#

You can modify that message with details on how you resolved the merge if you think it would be helpful to others looking at this merge in the future - why you did what you did, if it's not obvious.
