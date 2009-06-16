## Git References ##

So now we can run something like `git log 1a410e` to look through our whole history, but we still would have to remember that `1a410e` was the last commit in order to walk that history to find all those objects.  What we need is a file where we can store that SHA-1 value under a simple name so we can use that pointer rather than the raw SHA-1 value itself.

In Git, these are called 'references' or 'refs' and you can find the files that contain the SHA-1 values in the '.git/refs' directory.  In our current project, this directory contains no files, but it does contain a simple structure.

	$ find .git/refs
	.git/refs
	.git/refs/heads
	.git/refs/tags
	$ find .git/refs -type f
	$

If we want to create a new reference to remember where our latest commit is, we can technically do something as simple as this:

	$ echo "1a410efbd13591db07496601ebc7a059dd55cfe9" > .git/refs/heads/master

Now we can use the head reference we just created instead of the SHA-1 value in our Git commands.

	$ git log --pretty=oneline  master
	1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
	cac0cab538b970a37ea1e769cbbde608743bc96d second commit
	fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit

It is not encouraged to actually directly edit the reference files, Git provides a safer command to do this if you want to update a reference called 'update-ref'.

	$ git update-ref refs/heads/master 1a410efbd13591db07496601ebc7a059dd55cfe9

So that's basically what a branch in Git is, a simple pointer - a reference to the head of a line of work.  If we wanted to create a branch back at the second commit, we could do this:

	$ git update-ref refs/heads/test cac0ca

Then our branch will only contain work from that commit down.

	$ git log --pretty=oneline test
	cac0cab538b970a37ea1e769cbbde608743bc96d second commit
	fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit

Now our Git database conceptually looks something like Figure 9.4.

Fig4

DWP: I notice the hashes for the three commits are all the same in the diagram. That can't be right, can it? Ditto the figure 3. {: class=note}

DWP: Need a link to the diagram here too. {: class=note}

When you run commands like `git branch (branchname)` what Git is basically doing is running that `update-ref` command to add the SHA-1 of the last commit of the branch you are on into whatever new reference you want to create.

### The HEAD ###

The question now is, when I run `git branch (branchname)`, how does Git know what the SHA-1 of the last commit is?  The answer is the HEAD file.  The HEAD file is a symbolic reference to the branch you are currently on.  By 'symbolic reference' I mean that unlike a normal reference, it does not generally contain a SHA-1 value, but instead a pointer to another reference.  If we look at the file itself, we will normally see something like this:

	$ cat .git/HEAD 
	ref: refs/heads/master

If you run `git checkout test`, Git will update this file to look like this:

	$ cat .git/HEAD 
	ref: refs/heads/test

When you run `git commit`, it will create the commit object specifying the parent of that commit object to be whatever SHA-1 value the reference in HEAD points to.

sop: Should we mention here that `git symbolic-ref` is a better way to read/edit this file, just like `git update-ref` is a better way to edit a ref file?  {: class=note}

### Tags ###

We've just gone over the three main object types in Git, but there is actually a fourth one.  The 'tag' object is very much like a commit object - it contains a tagger, a date, a message and a pointer.  The main difference is that a tag object points to a commit rather than a tree.  It is like a branch reference but it never moves - it always points to the same commit, just giving it a more friendly name.  

As we cover in Chapter 2, there are two types of tags - annotated and lightweight.  A lightweight tag can basically be made by running something like this:

	$ git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d

That is all a lightweight tag is - a branch that never moves.  An annotated tag is more complex, however.  If we create an annotated tag, Git will create a 'tag' object and then write a reference to point to _that_, rather than directly to the commit.  We can see this by creating an annotated tag

	$ git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9
	
DWP: Where -a means add it rather than just show it? {: class=note}

Then seeing what the object SHA-1 value it created is

	$ cat .git/refs/tags/v1.1 
	9585191f37f7b0fb9444f35a9bf50de191beadc2

Then we can run the `cat-file` command on that SHA-1 value.

	$ git cat-file -p 9585191f37f7b0fb9444f35a9bf50de191beadc2
	object 1a410efbd13591db07496601ebc7a059dd55cfe9
	type commit
	tag v1.1
	tagger Scott Chacon <schacon@gmail.com> Sat May 23 16:48:58 2009 -0700

	test tag

DWP: I'm confused. How did the text 'test tag' get in here? {: class=note}

Notice that the 'object' entry points to the commit SHA-1 value that we tagged.  Also notice that it doesn't even need to point to a commit, you can tag any Git object.  In the Git source code, for example, the maintainer has added his GPG public key as a blob object and then tagged it.  You can view the public key by running:

	$ git cat-file blob junio-gpg-pub

in the Git source code.  The Linux kernel also has a non-commit pointing tag object - the very first tag created points to the initial tree of the import of the source code.

### Remotes ###

The third type of reference that you will see are remote references.  If you add a remote and push to it, Git will store what value you last pushed to that remote for each branch in the 'refs/remotes' directory.  For instance, if I add a remote called 'origin' and push my 'master' branch to it:

	$ git remote add origin git@github.com:schacon/simplegit-progit.git
	$ git push origin master
	Counting objects: 11, done.
	Compressing objects: 100% (5/5), done.
	Writing objects: 100% (7/7), 716 bytes, done.
	Total 7 (delta 2), reused 4 (delta 1)
	To git@github.com:schacon/simplegit-progit.git
	   a11bef0..ca82a6d  master -> master

Then I can see what the 'master' branch on the 'origin' remote was the last time I communicated with the server by checking the 'refs/remotes/origin/master' file:

	$ cat .git/refs/remotes/origin/master 
	ca82a6dff817ec66f44342007202690a93763949

Remote references differ from branches (refs/heads references) mainly in that they cannot be checked out.  Git simply moves them around as bookmarks to the last known state of where those branches were on those servers.

DWP: It feels like there ought to be slightly more on remotes here. For all the other things you've shown exactly what is stored in the file. Where and how does git store the remote url? {: class=note}
