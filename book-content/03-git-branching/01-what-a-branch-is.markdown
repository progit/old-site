# Git Branching #

Nearly every version control system out there has some form of branching support.  Branching is where you can diverge from the main line of development without messing with that main line.  In many VCS tools, this is a somewhat expensive process, often requiring you to create an entirely new copy of your source code directory, which can take quite a long time for large projects.

Some people have referred to the branching model in Git as its “killer feature” and it certainly sets it apart in the VCS community.  Why is it so special?  The way Git branches is incredibly lightweight, making branching operations nearly instant and switching back and forth between them generally just as fast.  Unlike many other VCSs, Git encourages a workflow that branches and merges quite often, even multiple times every day.  Understanding and mastering this feature gives you a very powerful and unique tool and can quite literally change the way that you develop. 

## What a Branch Is

To really understand the way Git does branching, we need to take a step back and examine how Git stores its data.  As you may remember from Chapter 1, Git does not store its data as a series of changesets or deltas, but instead as a series of snapshots. 

When you commit in Git, it stores a commit object, which contains a pointer to the snapshot of the content you had staged, the author and message metadata, and zero or more pointers to the commit or commits that were the direct parents of this commit.  Zero parents for the first commit, one for normal commits, and multiple parents for commits that result from a merge of two or more branches.

To visualize this, let’s assume that we have a directory with three files in it and we stage them all and commit.  Staging them is going to checksum each of the files (the SHA-1 hash we mentioned in Chapter 1), store that version of the file in the Git repository (Git refers to them as ‘blobs’) and add that checksum to the staging area.  

	$ git add README test.rb LICENSE2
	$ git commit -m 'initial commit of my project'
	
When you create the commit but running 'git commit', it will checksum each subdirectory (in this case, just the root project directory) and store those ‘tree’ objects in the Git repository.  It then creates a commit object that has the metadata and a pointer to the root project tree so it can recreate that snapshot when needed.  

There are now five objects in your Git repository – one ‘blob’ for the contents of each of your files, one ‘tree’ that lists the contents of the directory and specifies which file names are stored as which blobs, and one commit with the pointer to that root tree and all the commit metadata. Conceptually, the data in your Git repository looks something like Figure 3.1.

![Figure 3.1](/images/branch2.png)

If you make some changes and commit again, the next commit will store a pointer to the commit that came immediately before it.  So after two more commits, our history might look something like Figure 3.2.

![Figure 3.2](/images/branch1.png)

Now, a 'branch' in Git is simply a lightweight movable pointer to one of these commits.  The default branch name in Git is called 'master'.  As you initially make commits, you will be given a 'master' branch which will just point to the last commit you made.  Every time you commit, it will move forward automatically.

![Figure 3.3](/images/branch3.png)

Now, what happens if we create a new branch?  Well, it just creates a new pointer for us to move around.  Let's say we create a new branch called 'testing'.  We do this with the 'git branch' command:

	$ git branch testing
	
This will create a new pointer at the same commit we are currently on.

![Figure 3.4](/images/branch4.png)

So, how does Git know what branch we are currently 'on'?  There is a special pointer it keeps called 'HEAD'.  Note that this is a lot different than the  concept of HEAD in other version control systems that you might be used to such as Subversion or CVS.  In Git, this is a pointer to the local branch we are currently on.  In this case we are still on 'master'.  The 'git branch' command only created a new branch, it did not switch to it.

![Figure 3.5](/images/branch5.png)

To switch to one of our existing branches, we run the 'git checkout' command.  So, let's switch to our new 'testing' branch.
	
	$ git checkout testing
	
This will simply move our HEAD to point to the 'testing' branch.

![Figure 3.6](/images/branch6.png)

So what is the significance of that?  Well, let's do another commit.

	$ vim test.rb
	$ git commit -a -m 'made a change'

![Figure 3.7](/images/branch7.png)
	
This is interesting, because now our 'testing' branch has moved forward, but our 'master' branch still points to the commit we were on when we did our 'git checkout' to switch branches.  So, let's switch back to our 'master' branch.

	$ git checkout master
	
![Figure 3.8](/images/branch8.png)

That command actually did two things.  It moved the HEAD pointer back to point to the 'master' branch, and it reverted the files in our working directory back to the snapshot that 'master' points to.  This also means that the changes we make from this point forward will diverge from an older version of the project.  It essentially rewinds the work we've done in our 'testing' branch temporarily so we can go in a different direction.

So, let's make a few changes and commit again.

	$ vim test.rb
	$ git commit -a -m 'made other changes'
	
Now our project history has diverged.  We created and switched to a branch, did some work on it, then switched back to our main branch and did seperate work.  Now both of those changes are isolated in 'branches' that we can switch back and forth between and then merge together when we are ready to.  All just with simple branch and checkout commands.

![Figure 3.9](/images/branch9.png)

## Why Git Branching is Different

Since the branches in Git are in actuality simple files that contain the 40 character SHA-1 checksum of the commit it points to, they are very cheap to create and destroy.  It is as quick and simple to create a new branch as writing 41 bytes to a file (40 characters and a newline). 

This is in sharp contrast to how most SCM systems branch, which involves copying all the files of the project into a second directory.  This can take several seconds or even minutes, depending on the size of the project, whereas in Git it is always instantaneous.  This speed helps encourage developers to create and use branches often.

So, let's see why we should.



