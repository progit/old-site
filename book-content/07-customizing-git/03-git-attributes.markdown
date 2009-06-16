## Git Attributes ##
 
Some of these settings can also be set for a specific path, so that Git only applies those settings for some subdirectory or subset of files.  These path specific settings are called Git 'attributes' and are set either in a `.gitattribute` file in one of your directories, normally the root of your project, or in the `.git/info/attributes` file if you don't want it committed with your project.

With this you can do things like specify seperate merge strategies for individual files or directories in your project, tell Git how to diff non-text files or even filter content before you check it in or out of Git.  In this section, we will cover some of the attributes you can set on your paths in your Git project and show a few examples of using this feature in practice.

### Binary Files ###

One pretty cool trick you can use Git attributes for is telling Git which files are binary that it may not be able to figure out and even give Git special instructions on how to handle them.  

DWP: Perhaps remind the reader how Git treats binary files. {: class=note}

#### Identifying Binary Files ####

Some files look like text files, but for all intents and purposes, are to be treated as binary data.  For instance, XCode projects on the Mac have a file in them that ends in '.pbxproj', which is basically a JSON dataset that is written out to disk by the IDE that records your build settings and such.  Though it is technically a text file, as it's all ASCII, you don't really want to treat it as such because it's really a lightweight database - you can't really merge contents if two people changed it and diffs generally aren't that helpful - the file is meant to be consumed by a machine.  In essence, you want to treat it like a binary file.

To tell Git to treat all `pbxproj` files as binary data, you can add the following line to your `.gitattributes` file.

	*.pbxproj -crlf -diff

Now Git will not try to convert or fix CRLF issues nor will it try to compute or print a diff for changes in this file when you run `git show` or diff on your project.  In the 1.6 series of Git, you can also use a macro that is provided that means '-crlf -diff':

	*.pbxproj binary
	

#### Diffing Binary Files ####

In the 1.6 series of Git, you can use the Git attributes functionality to effectively diff binary files.  The way you do this is by telling Git how to convert your binary data to a text format that can be compared via the normal 'diff'.

Since this is a pretty cool and not widely known feature, we'll go over a few examples.  First, let's use this to solve one of the most annoying problems known to man - version controlling Word documents. Everyone knows that Word is the most horrific editor around, but oddly, everyone uses it.  If we want to verison control Word documents, we can stick them in a Git repository and commit every once in a while, but what good will that do?  If we run `git diff` normally, we'll only see something like this:

	$ git diff 
	diff --git a/chapter1.doc b/chapter1.doc
	index 88839c4..4afcb7c 100644
	Binary files a/chapter1.doc and b/chapter1.doc differ

We can't really directly compare them unless you checkout two versions and scan them manually, right?  It turns out we can actually do this fairly well using Git attributes.  If we put the following line in our `.gitattributes` file:

	*.doc diff=word
	
This tells Git that any file that matches this pattern ('*.doc') should use the 'word' filter when we try to view a diff that contains changes to files like this.  So what is the 'word' filter?  Next we have to set that up.  Here we're going to configure Git to use the `strings` program to convert Word documents into readable text files, which it will then diff properly.  

	$ git config diff.word.textconv strings

Now Git knows that if it tries to do a diff between two snapshots and any of the files end in '*.doc', it should run those files through the 'word' filter, which is defined as the 'strings' program.  This will effectively make nice text based versions of our Word files before attempting to diff them.

Let's see an example.  Here I added Chapter 1 of this book into Git, added some text to a paragraph and saved the document.  Now I can run `git diff` and see what it is that I changed.
	
	$ git diff
	diff --git a/chapter1.doc b/chapter1.doc
	index c1c8a0a..b93c9e4 100644
	--- a/chapter1.doc
	+++ b/chapter1.doc
	@@ -8,7 +8,8 @@ re going to cover Version Control Systems (VCS) and Git basics
	 re going to cover how to get it and set it up for the first time if you don
	 t already have it on your system.
	 In Chapter Two we will go over basic Git usage - how to use Git for the 80% 
	-s going on, modify stuff and contribute changes. If the book spontaneously 
	+s going on, modify stuff and contribute changes. If the book spontaneously 
	+Let's see if this works.
	 Chapter Three is about the branching model in Git, often described as Git

It successfully and succinctly tells me that I added the string "Lets see if this works", which is correct. It's not perfect, however - it adds a bunch of random stuff at the end, but it certainly works.  If you can find or write a Word to plain text converter that works well enough, that solution would likely be incredibly effective.  However, 'strings' is available on most Mac and Linux systems, so it might be a good first try to do this with many binary formats.

Another interesting problem you can solve this way is in diffing image files.  One way to do this is to run JPEG files through a filter that extracts it's EXIF information - meta data that is recorded with most image formats. If you download and install the `exiftool` program, you can use it to convert your images into text about the meta data, so at least the diff will show you some sort of textual representation of any changes that happened.

	$ echo '*.png diff=exif' >> .gitattributes
	$ git config diff.exif.textconv exiftool

Now if you replace an image in your project, then when you run `diff` you will see something like this:

	diff --git a/image.png b/image.png
	index 88839c4..4afcb7c 100644
	--- a/image.png
	+++ b/image.png
	@@ -1,12 +1,12 @@
	 ExifTool Version Number         : 7.74
	-File Size                       : 70 kB
	-File Modification Date/Time     : 2009:04:21 07:02:45-07:00
	+File Size                       : 94 kB
	+File Modification Date/Time     : 2009:04:21 07:02:43-07:00
	 File Type                       : PNG
	 MIME Type                       : image/png
	-Image Width                     : 1058
	-Image Height                    : 889
	+Image Width                     : 1056
	+Image Height                    : 827
	 Bit Depth                       : 8
	 Color Type                      : RGB with Alpha

So you can easily see that the file size and image dimensions have both changed.

### Keyword Expansion ###

One thing that is asked for a lot is SVN or CVS style keyword expansion.  The main problem with this is that you can't modify a file with information about the commit after you've committed, since it checksums the file first.  However, you can inject text into a file when it's checked out and remove it again before it's added to a commit.  Git attributes offers you two ways to do this.

The first is the ability to inject the SHA-1 checksum of a blob into an `$Id$` field in the file automatically.  If you set this attribute on a file or set of files, the next time that you checkout that branch, Git will replace that field with the SHA-1 of the blob.  It's important to notice that it is not the SHA of the commit, but the blob itself.

	$ echo '*.txt ident' >> .gitattributes
	$ echo '$Id$' > test.txt

Now the next time you check this file out, Git will inject the SHA of the blob.

	$ rm text.txt
	$ git checkout -- text.txt
	$ cat test.txt 
	$Id: 42812b7653c7b88933f8a9d6cad0ca16714b9bb3 $

However, that is limitedly useful.  If you've used keyword substitution in CVS or Subversion, you can get a datestamp or something in there - the SHA is not all that helpful, since it's pretty random and you can't tell if one is older or newer than another.

However, it turns out that you can write your own filters for doing substitutions in files on commit/checkout.  These are the 'clean' and 'smudge' filters.  You can set in the `.gitattributes` file a 'filter' for particular paths and then setup scripts that will process files right before they are committed ('clean') and right before they are checked out ('smudge').  These filters can be set to do all sorts of fun things.

The original commit message for this functionality gives a simple example of running all your C source code through the 'indent' program before committing.  You can set it up by setting the 'filter' attribute in your `.gitattributes` file to filter `*.c` files with the 'indent' filter:

	(in .gitattributes)
	*.c     filter=indent
	
Then telling Git what the 'indent' filter does on 'smudge' and 'clean':

	$ git config --global filter.indent.clean indent
	$ git config --global filter.indent.smudge cat

In this case, when you commit files that match `*.c`, Git will run them through the `indent` program before it commits them, and then run them through the `cat` program before it checks them back out onto disk.  The `cat` program is basically a no-op, it just spits out the same data that it gets in. This effectively filters all C source code files through `indent` before committing.

Another interesting example is to get `$Date$` keyword expansion, RCS style.  To do this properly, we'll want a small script that takes a filename, figures out the last commit date for this project and inserts the date into the file.  Here is a small Ruby script that will do that for us. 

	#! /usr/bin/env ruby
	data = STDIN.read
	last_date = `git log --pretty=format:"%ad" -1`
	puts data.gsub('$Date$', '$Date: ' + last_date.to_s + '$')

We can name this file `expand_date` and put it in our path somewhere.  Now we need to setup a filter in Git, we'll call it 'dater' and tell it to use our 'expand_date' filter to smudge the files on checkout, and then we'll use a simple Perl expression to clean that up on commit.

	$ git config filter.dater.smudge expand_date
	$ git config filter.dater.clean 'perl -pe "s/\\\$Date[^\\\$]*\\\$/\\\$Date\\\$/"'

DWP: I think you should probably spell out what those bits of Ruby and Perl do. Your reader may well know nothing about Perl or Ruby. In fact, I'd prefer it if you didn't use Ruby really: I'm assuming the average reader of this book is using a linux box (or maybe a mac). You can be pretty sure perl and python and shellscript will be installed (on linux - I can't comment on Macs!), but Ruby is definitely an extra. {: class=note}

Now that the filter is setup, we can test it by setting up a file with our $Date$ keyword, then setting up a Git attribute for that file that engages our new filter.

	$ echo '# $Date$' > date_test.txt
	$ echo 'date*.txt filter=dater' >> .gitattributes

Now if we commit those changes and check the file back out again, we'll see the keyword being properly substituted.

	$ git add date_test.txt .gitattributes
	$ git commit -m "Testing date expansion in Git"
	$ rm date_test.txt
	$ git checkout date_test.txt
	$ cat date_test.txt
	# $Date: Tue Apr 21 07:26:52 2009 -0700$

So you can see how powerful this can be for customized applications.  You have to be a bit careful though, since the `.gitattributes` file is committed and passed around with the project, but the driver (in this case, 'dater') is not, so it's not neccesarily going to work everywhere.  When you design these, they should be able to fail gracefully and have the project still work properly.
	

### Exporting your Repository ###

The Git attributes data also allows you to do some interesting things when exporting an archive of your project.

#### export-ignore ####

One thing you can do is tell Git not to export certain files or directories when generating an archive.  If there is a subdirectory or file that you don't want to include in your archive file but that you do want checked into your  project, you can use Git attributes to determine those files via the `export-ignore` attribute.

For example, say you had some test files in a 'test/' subdirectory that simply didn't make sense to include in the tarball export of your project.  You can add the following line to your Git attributes file:

	test/ export-ignore
	
Now when you run `git archive` to create a tarball of your project, that directory will simply not be included in the archive.

#### export-subst ####

Another thing you can do for your archives is some simple keyword substitution.  Git lets you put the string `$Format:$` in any file with any of the `--pretty=format` formatting shortcodes, many of which we covered in Chapter 2.  For instance, if we wanted to have a file named `LAST_COMMIT` in our project which automatically had injected into it the last commit date when `git archive` was run, we can setup the file like this:

	$ echo 'Last commit date: $Format:%cd$' > LAST_COMMIT
	$ echo "LAST_COMMIT export-subst" >> .gitattributes
	$ git add LAST_COMMIT .gitattributes
	$ git commit -am 'adding LAST_COMMIT file for archives'

Now when we run `git archive`, the contents of that file when people open up that archive file will look like this:

	$ cat LAST_COMMIT
	Last commit date: $Format:Tue Apr 21 08:38:48 2009 -0700$
	
### Merge Strategies ###

You can also use Git attributes to tell Git to use different merge strategies for specific files in your project.  One very useful option is to tell Git to not try to merge specific files when they have conflicts, but to simply use your side of the merge over theirs.  

This is helpful if you have a branch in your project that has diverged or is specialized but that you want to be able to merge changes back in from and you basically want to ignore certain files.  Say you have a database settings file called `database.xml` that is setup differently in two different branches and you want to merge in your other branch without having it mess up your database file.  You can set up an attribute like this:

	database.xml merge=ours
	
Then if you merge in the other branch, instead of having merge conflicts with the database.xml file, you will simply see something like this:

	$ git merge topic
	Auto-merging database.xml
	Merge made by recursive.	

Where `database.xml` simply stays at whatever version you originally had.

DWP: Is there something you can use instead of 'ours' to always get what comes from the branch? {: class=note}


