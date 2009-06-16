## Packfiles ##

Let's go back to our objects database for our little test Git repository.  At this point we have 11 objects - 4 blobs, 3 trees, 3 commits and 1 tag.

	$ find .git/objects -type f
	.git/objects/01/55eb4229851634a0f03eb265b69f5a2d56f341 # tree 2
	.git/objects/1a/410efbd13591db07496601ebc7a059dd55cfe9 # commit 3
	.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a # test.txt v2
	.git/objects/3c/4e9cd789d88d8d89c1073707c3585e41b0e614 # tree 3
	.git/objects/83/baae61804e65cc73a7201a7252750c76066a30 # test.txt v1
	.git/objects/95/85191f37f7b0fb9444f35a9bf50de191beadc2 # tag
	.git/objects/ca/c0cab538b970a37ea1e769cbbde608743bc96d # commit 2
	.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
	.git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579 # tree 1
	.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
	.git/objects/fd/f4fc3344e67ab068f836878b6c4951e3b15f3d # commit 1

Git compresses the contents of these files with Zlib and we're not storing much, so all of these files collectively take up only 925 bytes.  So, let's add some larger content to our repository to demonstrate an interesting feature of Git.  We'll add the repo.rb file from the Grit library we worked with earlier - this is about a 12k source code file.

	$ curl http://github.com/mojombo/grit/raw/master/lib/grit/repo.rb > repo.rb
	$ git add repo.rb 
	$ git commit -m 'added repo.rb'
	[master 484a592] added repo.rb
	 3 files changed, 459 insertions(+), 2 deletions(-)
	 delete mode 100644 bak/test.txt
	 create mode 100644 repo.rb
	 rewrite test.txt (100%)

If we look at the tree that created, we can see the SHA-1 value for the blob object that our repo.rb file got.

	$ git cat-file -p master^{tree}
	100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
	100644 blob 9bc1dc421dcd51b4ac296e3e5b6e2a99cf44391e	repo.rb
	100644 blob e3f094f522629ae358806b17daf78246c27c007b	test.txt

We can then use `git cat-file` to see how big that object is:

	$ git cat-file -s 9bc1dc421dcd51b4ac296e3e5b6e2a99cf44391e
	12898

Now we're going to modify that file a little and see what happens.

	$ echo '# testing' >> repo.rb 
	$ git commit -am 'modified repo a bit'
	[master ab1afef] modified repo a bit
	 1 files changed, 1 insertions(+), 0 deletions(-)
	
Now if we check the tree created by that commit, we'll see something interesting.

	$ git cat-file -p master^{tree}
	100644 blob fa49b077972391ad58037050f2a75f74e3671e92	new.txt
	100644 blob 05408d195263d853f09dca71d55116663690c27c	repo.rb
	100644 blob e3f094f522629ae358806b17daf78246c27c007b	test.txt

The blob is now a different blob, which means that although we only added a single simple line to the end of a 400 line file, Git stores that new content as a completely new object.

	$ git cat-file -s 05408d195263d853f09dca71d55116663690c27c
	12908

We now have two nearly identical 12k objects on our disk.  Wouldn't it be nice if Git could store those as a single version and then just the difference from the first version to the second?  

It turns out that it does.  The initial format that Git will save objects in on disk is called a 'loose' object format.  However, occasionally Git will pack up several of these objects into a single binary file called a 'packfile' in order to save space and be more efficient.  Git will do this if you have too many loose objects around, if you run the `git gc` command manually, or if you push to a remote server.  To see what happens, we can manually ask Git to pack up the objects by calling the `git gc` command.

	$ git gc
	Counting objects: 17, done.
	Delta compression using 2 threads.
	Compressing objects: 100% (13/13), done.
	Writing objects: 100% (17/17), done.
	Total 17 (delta 1), reused 10 (delta 0)

Now if we look in our objects directory, we will find most of our objects gone and a new pair of files appear.

	$ find .git/objects -type f
	.git/objects/71/08f7ecb345ee9d0084193f147cdad4d2998293
	.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
	.git/objects/info/packs
	.git/objects/pack/pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.idx
	.git/objects/pack/pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.pack

The objects that remain are the blobs that are not pointed to by any commit - in this case the 'what is up, doc?' example and the 'test content' example blobs we created earlier.  Since we never added them to any commits, they are considered 'dangling' and are not packed up in our new packfile.

The other files are our new packfile and an index.  The packfile is a single file containing the contents of all the objects that were removed from our filesystem.  The index is a file that contains offsets into that packfile so we can quickly seek to a specific object. What is cool is that although the objects on disk before running the `gc` were collectively about 12k in size, the new packfile is only 6k.  We halved our disk usage by packing our objects. 

So, how did Git do this? When Git packs its objects, it looks for files that are named and sized similarly and will store just the deltas from one version of the file to the next.  We can actually look into the packfile and see what it did to save space.  The `git verify-pack` plumbing command allows us to see what was packed up.

	$ git verify-pack -v pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.idx
	0155eb4229851634a0f03eb265b69f5a2d56f341 tree   71 76 5400
	05408d195263d853f09dca71d55116663690c27c blob   12908 3478 874
	09f01cea547666f58d6a8d809583841a7c6f0130 tree   106 107 5086
	1a410efbd13591db07496601ebc7a059dd55cfe9 commit 225 151 322
	1f7a7a472abf3dd9643fd615f6da379c4acb3e3a blob   10 19 5381
	3c4e9cd789d88d8d89c1073707c3585e41b0e614 tree   101 105 5211
	484a59275031909e19aadb7c92262719cfcdf19a commit 226 153 169
	83baae61804e65cc73a7201a7252750c76066a30 blob   10 19 5362
	9585191f37f7b0fb9444f35a9bf50de191beadc2 tag    136 127 5476
	9bc1dc421dcd51b4ac296e3e5b6e2a99cf44391e blob   7 18 5193 1 05408d195263d853f09dca71d55116663690c27c
	ab1afef80fac8e34258ff41fc1b867c702daa24b commit 232 157 12
	cac0cab538b970a37ea1e769cbbde608743bc96d commit 226 154 473
	d8329fc1cc938780ffdd9f94e0d364e0ea74f579 tree   36 46 5316
	e3f094f522629ae358806b17daf78246c27c007b blob   1486 734 4352
	f8f51d7d8a1760462eca26eebafde32087499533 tree   106 107 749
	fa49b077972391ad58037050f2a75f74e3671e92 blob   9 18 856
	fdf4fc3344e67ab068f836878b6c4951e3b15f3d commit 177 122 627
	chain length = 1: 1 object
	pack-7a16e4488ae40c7d2bc56ea2bd43e25212a66c45.pack: ok

We can see that the `9bc1d` blob, which if you remember back was the first version of our repo.rb file, is referencing the `05408` blob, which was the second version of the file.  The third column in that output is the size of the object in the pack, so we can see that `05408` takes up 12k of the file but that `9bc1d` only takes up 7 bytes.  What is also interesting about this is that the second version of the file is the one that is stored intact while the original version is the one stored as a delta - this is because the most recent version of the file is the one you're most likely to need faster access to.

The really nice thing about this is that it can be repacked at any time.  Git will occasionally repack your database, always trying to save more space.  You can also manually repack at any time by running `git gc` by hand.
