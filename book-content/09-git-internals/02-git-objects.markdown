## Git Objects ##

So, Git is a content addressable filesystem.  Great.  So what does that even mean?

It means that at the core of Git is a simple key-value data store.  You can insert any kind of content into it and it will give you a key back that you can use to retrieve that content again at any time.  To demonstrate this, we can use the plumbing command 'hash-object' which will take some data, store it in our .git directory and give us back the key it stored it as.  First of all, let's initialize a new Git repository and verify that there is nothing in the 'objects' directory'.

	$ mkdir test
	$ cd test
	$ git init
	Initialized empty Git repository in /tmp/test/.git/	
	$ find .git/objects
	.git/objects
	.git/objects/info
	.git/objects/pack
	$ find .git/objects -type f
	$
	
So, Git has initialized the 'objects' directory and created 'pack' and 'info' subdirectories in it, but there are no regular files there.  Now let's store some text in our Git database.

	$ echo 'test content' | git hash-object -w --stdin
	d670460b4b4aece5915caf5c68d12f560a9fe3e4

The `-w` tells `hash-object` to store the object, otherwise it will simply tell you what the key _would_ be and `--stdin` obviously tells it to read the content from stdin, otherwise it expects the path to a file.  Notice that the output from the command is a 40 character checksum hash.  This is the SHA-1 checksum of the content you're storing plus a simple header, which we'll cover in a bit.  However, now we can see how Git has stored our data:

	$ find .git/objects -type f 
	.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4

Now we can see a file in the 'objects' directory.  This is how Git actually stores the content initially - as a single file per piece of content, named by the SHA-1 checksum of the content and its header, in a subdirectory of the the first two characters of the SHA, with the actual filename being the remaining 38 characters.

We can now pull that content back out of Git with a command called 'cat-file'.  We're going to pass a `-p` to it which instructs the `cat-file` command to figure out what type of content it is and display it nicely for us.

	$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
	test content

Now we can add content to Git and pull it back out again.  We can also do this with content in files.  Let's so do some simple version control on a file.  First we'll create a new file and save its contents in our database.

	$ echo 'version 1' > test.txt
	$ git hash-object -w test.txt 
	83baae61804e65cc73a7201a7252750c76066a30

Then we'll write some new content to the file and save it again.

	$ echo 'version 2' > test.txt
	$ git hash-object -w test.txt 
	1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

Now we can see that our database has the two new versions of the file saved in it as well as the first content we stored there.

	$ find .git/objects -type f 
	.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
	.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
	.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4

Now I can revert my file back to the first version:

	$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt 
	$ cat test.txt 
	version 1

Or the second version:

	$ git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a > test.txt 
	$ cat test.txt 
	version 2

However, remembering the SHA-1 key for each version of our file is not practical, plus we aren't storing the actual filename in our system anywhere, just the content.  This object type is called a 'blob'.  In fact, we can have Git tell us what the content type for any SHA-1 is with `cat-file -t`.

	$ git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
	blob

## Tree Objects ##

The next type we're going to look at is the 'tree' object, which solves our next problem of storing the name of the file and also allows us to store a group of files together.  Git actually stores its content in a similar manner to a UNIX filesystem, though a bit simplified.  All of the content is stored as 'tree' and 'blob' objects, with 'trees' corresponding to UNIX directory entries and 'blobs' corresponding more or less to inodes or file contents.  A single tree object will contain one or more tree entries, each of which containing a SHA-1 pointer to a blob or subtree with its associated mode, type and filename.  For example, the most recent tree in our `simplegit` project may look something like this:

	$ git cat-file -p master^{tree}
	100644 blob a906cb2a4a904a152e80877d4088654daad0c859	README
	100644 blob 8f94139338f9404f26296befa88755fc2598c289	Rakefile
	040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0	lib

DWP: Will the reader know what master^{tree} means by now? {: class=note}

Notice that the 'lib' subdirectory is not a blob but a pointer to another tree.

	$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
	100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b	simplegit.rb

So conceptually, the data that Git is storing is something like this:

Fig1

DWP: I assume there should be an actual link to a file here for Fig1. {: class=note}

So, let's create our own trees.  To create a tree from the first verison of the file, we can use a plumbing command called 'update-index' to artificially add our earlier version of the test.txt file to our staging area. We'll have to pass it `--add` since the file doesn't exist in our staging area yet (in fact, we don't have a staging area setup yet) and `--cacheinfo` since the file we're adding isn't in our directory right now but it is in our database.  Then we specify the mode, SHA-1 and file name.

	$ git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt

DWP: Where does the 100644 come from? Is that permissions? {: class=note}

Now we can use a command called `write-tree` to write the staging area out to a tree object.

DWP: So unlike hash-object above, write-tree automatically adds this to the database. {: class=note}

	$ git write-tree
	d8329fc1cc938780ffdd9f94e0d364e0ea74f579
	$ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
	100644 blob 83baae61804e65cc73a7201a7252750c76066a30	test.txt

We can also verify that this is a tree object:

	$ git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579
	tree

Now let's create a new tree with the second version of test.txt and a new file as well.

	$ echo 'new file' > new.txt
	$ git update-index test.txt 
	$ git update-index --add new.txt 

Our staging area now has the new version of test.txt as well as a new file `new.txt`.  Let's write that tree out and see what it looks like.

DWP: So 'write that tree out' is shifting it from the staging area to objects? {: class=note}

	$ git write-tree
	0155eb4229851634a0f03eb265b69f5a2d56f341
	$ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
	100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
	100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	test.txt

Notice that this tree has both file entries and also that the 'test.txt' SHA is the 'version 2' SHA from earlier (`1f7a7a`).  Just for fun, let's add the first tree as a subdirectory into this one.  We can read trees into our staging area by calling `read-tree`.

DWP: So --prefix=bak gives a name in the staging area for the directory that you're making out of the tree which you've just fetched from objects? {: class=note}

	$ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
	$ git write-tree
	3c4e9cd789d88d8d89c1073707c3585e41b0e614
	$ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
	040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579	bak
	100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
	100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a	test.txt

If we created a working directory from this new tree we just wrote, we would get the two files and a subdirectory named 'bak' that contained the first version of the 'test.txt' file.  The data that Git contains for these structures can be thought of like Figure 2.

Fig2

DWP: As obove, should there be a link here? {: class=note}

## Commit Objects ##

Now we have three trees that specify the different snapshots of our project that we wanted to track, but we have the same problem that we had earlier in that we have to remember all three SHA-1 values in order to recall the snapshots.  Also, we don't have any information about who saved the snapshots, when they saved them or why they saved them.  This is the basic information that the 'commit' object stores for us.

To create a commit object, we can call 'commit-tree' and simply specify a tree SHA-1 and which commit objects, if any, directly preceeded it.  We'll start with the first tree that we wrote.

	$ echo 'first commit' | git commit-tree d8329f
	fdf4fc3344e67ab068f836878b6c4951e3b15f3d

So, now we can look at our new commit object with `cat-file`.

	$ git cat-file -p fdf4fc3
	tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
	author Scott Chacon <schacon@gmail.com> 1243040974 -0700
	committer Scott Chacon <schacon@gmail.com> 1243040974 -0700

	first commit

You can see that the format for a commit object is very simple.  It just specifies the top level tree for the snapshot of the project at that point, then the author/committer information pulled from our `user.name` and `user.email` configuration settings, with the current timestamp then a blank line and then the commit message.

Now let's write the other two commit objects, each referencing the commit that came directly before it.

	$ echo 'second commit' | git commit-tree 0155eb -p fdf4fc3
	cac0cab538b970a37ea1e769cbbde608743bc96d
	$ echo 'third commit'  | git commit-tree 3c4e9c -p cac0cab
	1a410efbd13591db07496601ebc7a059dd55cfe9

So now we have three commit objects, each pointing to one of the three snapshot trees we created.  Oddly enough, we have a real Git history now that we can even view with the `git log` command if we run it on the last commit SHA-1.

	$ git log --stat 1a410e
	commit 1a410efbd13591db07496601ebc7a059dd55cfe9
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Fri May 22 18:15:24 2009 -0700

	    third commit

	 bak/test.txt |    1 +
	 1 files changed, 1 insertions(+), 0 deletions(-)

	commit cac0cab538b970a37ea1e769cbbde608743bc96d
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Fri May 22 18:14:29 2009 -0700

	    second commit

	 new.txt  |    1 +
	 test.txt |    2 +-
	 2 files changed, 2 insertions(+), 1 deletions(-)

	commit fdf4fc3344e67ab068f836878b6c4951e3b15f3d
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Fri May 22 18:09:34 2009 -0700

	    first commit

	 test.txt |    1 +
	 1 files changed, 1 insertions(+), 0 deletions(-)

Amazing.  We've now just done the low level operations to build up a Git history without using any of the nice frontends.  This is essentially what Git is doing when you're running the `git add` and `git commit` commands - storing blobs for the files that have changed, updating the index, writing out trees and writing commit objects that reference the top level trees and the commits that came immediately before them.  Those are the three main Git objects - the 'blob', the 'tree' and the 'commit' and they are initially stored as simple seperate files in your '.git/objects' directory.  Here are all the objects in our directory now, commented with what they actually store.

	$ find .git/objects -type f
	.git/objects/01/55eb4229851634a0f03eb265b69f5a2d56f341 # tree 2
	.git/objects/1a/410efbd13591db07496601ebc7a059dd55cfe9 # commit 3
	.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a # test.txt v2
	.git/objects/3c/4e9cd789d88d8d89c1073707c3585e41b0e614 # tree 3
	.git/objects/83/baae61804e65cc73a7201a7252750c76066a30 # test.txt v1
	.git/objects/ca/c0cab538b970a37ea1e769cbbde608743bc96d # commit 2
	.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
	.git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579 # tree 1
	.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
	.git/objects/fd/f4fc3344e67ab068f836878b6c4951e3b15f3d # commit 1


If you follow all the internal pointers, you get an object graph something like Fig 3.

Fig3

DWP: Need link for figure. {: class=note}

## Object Storage ##

I mentioned earlier that a header is stored with the content - let's take a minute to look at how exactly Git stores its objects.  Let's go through how to store an actual blob object, let's say the string "what is up, doc?" - we'll do it interactively in the Ruby scripting language.

	$ irb
	>> content = "what is up, doc?"
	=> "what is up, doc?"

Git  constructs a header that starts with the type of the object, in this case a 'blob'.  Then it adds a space followed by the size of the content and finally a null byte.

	>> header = "blob #{content.length}\0"
	=> "blob 16\000"

Then it concatenates the header and the original content and then it calculates the SHA-1 checksum of that new content.  

	>> store = header + content
	=> "blob 16\000what is up, doc?"
	>> require 'digest/sha1'
	=> true
	>> sha1 = Digest::SHA1.hexdigest(store)
	=> "bd9dbf5aae1a3862dd1526723246b20206e5fc37"

DWP: What does 'require' do? Is that how things are imported in Ruby? Please don't assume anyone knows Ruby. {: class=note}

Git will then compress the new content with Zlib.

	>> require 'zlib'
	=> true
	>> zlib_content = Zlib::Deflate.deflate(store)
	=> "x\234K\312\311OR04c(\317H,Q\310,V(-\320QH\311O\266\a\000_\034\a\235"

Finally, we're going to write our Zlib deflated content to an object on disk.

	>> path = '.git/objects/' + sha1[0,2] + '/' + sha1[2,38]
	=> ".git/objects/bd/9dbf5aae1a3862dd1526723246b20206e5fc37"
	>> require 'fileutils'
	=> true
	>> FileUtils.mkdir_p(File.dirname(path))
	=> ".git/objects/bd"
	>> File.open(path, 'w') { |f| f.write zlib_content }
	=> 32
	
DWP: Please be careful to explain thoroughly anything written in Ruby. I assume |f| just gets you a local identifier for the file... {: class=note}

That's it - we've now created a valid Git blob object.  All Git objects are stored the same way, just with different types - instead of the string 'blob' at the beginning of the header, it will be 'commit' or 'tree'.  Also, while the blob content can be nearly anything, the commit and tree content are very specifically formatted.


