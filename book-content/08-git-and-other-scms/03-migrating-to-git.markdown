## Migrating to Git ##

If you have an existing codebase in another VCS but you have made the decision that you are going to start using Git, you will have to migrate your project in one way or another.  This section will go over some importers that are included with Git for some common systems and then demonstrate how to develop our own custom importer.

### Importing ###

We will cover how to import data from two of the bigger professionally used SCM systems - Subversion and Perforce, both because they make up the majority of users that I hear of that are currently switching, and because high quality tools for both systems are distributed with Git itself.

#### Subversion ####

Now if you read the previous section on using `git svn` you can easily use those instructions to `git svn clone` a repository and then just stop using the Subversion server, push to a new Git server and start using that.  If you just want the history quickly, you can accomplish that as quickly as you can pull the data out of the Subversion server (which may actually take a pretty good amount of time).

However, the import is not perfect, and since it's going to take so long anyhow, we might as well do it right.  The first problem is the author information.  In Subversion each person committing has a user on the system that is recorded in the commit information.  In my examples in the previous section, you can see 'schacon' in some of the places, such as the `blame` output and the `git svn log`.  If we want to map this to better Git author data we'll need a mapping from the Subversion users to the Git authors.  If we create a file called `users.txt` that has this mapping in a format like this:

	schacon = Scott Chacon <schacon@geemail.com>
	selse = Someo Nelse <selse@geemail.com>

To get a list of the author names that SVN uses, you can run this:

	svn log --xml | grep author | sort -u | perl -pe "s/\<(.*?)\>//g"

sop: I suggest perl -pe 's/.*>(.*?)<.*/$1 = /' instead as it gets you lines like "spearce = " which is easier to edit. {: class=note}

That will give you your log output in xml format, look for just the authors, create a unique list of them and then strip out the XML.  This will fairly obviously only work on a Mac or Linux machine.  Then you can redirect that output into your users.txt file so you can add the equivalent Git user data next to each entry.

dwp: When you say "this will fairly obviously work on a Mac or Linux machine", do you really mean that it won't work on Windows? {: class=note}

Then we can provide this file to `git svn` to help it map the author data more accurately.  We can also tell `git svn` not to include the metadata that Subversion will normally import by passing `--no-metadata` to the `clone` or `init` commands.  This makes our import command look like this:

	$ git-svn clone http://my-project.googlecode.com/svn/ \
		--authors-file=users.txt --no-metadata -s my_project

Now we should have a nicer Subversion import in our `my_project` directory.  Instead of commits that look like this:

	commit 37efa680e8473b615de980fa935944215428a35a
	Author: schacon <schacon@4c93b258-373f-11de-be05-5f7a86268029>
	Date:   Sun May 3 00:12:22 2009 +0000

	    fixed install - go to trunk
    
	    git-svn-id: https://schacon-test.googlecode.com/svn/trunk@94 4c93b258-373f-11de-be05-5f7a86268029

They will instead now look like this:

	commit 03a8785f44c8ea5cdb0e8834b7c8e6c469be2ff2
	Author: Scott Chacon <schacon@geemail.com>
	Date:   Sun May 3 00:12:22 2009 +0000

	    fixed install - go to trunk

You can see that not only does the 'Author' field look a lot better, but the 'git-svn-id' is no longer there, either.

Now there is a bit of post-import cleanup to do. For one, we'll want to clean up the weird references that `git svn` setup.  First we want to move the tags to be actual tags rather than strange remote branches, and then we'll move the rest of the branches to be local branches.

dwp: Perhaps remind the reader why the references are weird, and what the remote branches look like. {: class=note}

To move the tags, all we have to do is run 

	$ cp -Rf .git/refs/remotes/tags/* .git/refs/tags/
	$ rm -Rf .git/refs/remotes/tags
	
This will make the references that were remote branches that just started with 'tag/' to be real (lightweight) tags.

The next thing is to move the rest of the references under 'refs/remotes' to be local branches rather than remote ones.

	$ cp -Rf .git/refs/remotes/* .git/refs/heads/
	$ rm -Rf .git/refs/remotes
	
Now we have all the old branches as real Git branches and all the old tags as real Git tags.  The last thing to do is add your new Git server as a remote and push to it.  Since you want all your branches and tags to go up, you can just run:

	$ git push origin --all

Now all your branches and tags should be on your new Git server in a nice, clean import.

#### Perforce ####

The next system we're going to look at importing from is Perforce.  A Perforce importer is also distributed with Git, but only in the 'contrib' section of the source code - it is not available by default like `git svn`.  In order to run it, you have to get the Git source code, which you can download via Git itself from `git.kernel.org`.

	$ git clone git://git.kernel.org/pub/scm/git/git.git
	$ cd git/contrib/fast-import

In this 'fast-import' directory, you should find an executable Python script named 'git-p4'.  You will have to have Python and the 'p4' tool installed on your machine for this import to work and it will most likely have to be a Mac or Linux machine. As an example, we'll import the Jam project from the Perforce Public Depot.  To set up your client you will have to export the `P4PORT` environment variable to point to the Perforce depot.

	$ export P4PORT=public.perforce.com:1666
	
Now you can run the `git-p4 clone` command to import the Jam project from the Perforce server, supplying the depot and project path and the path you want to import the project into.

	$ git-p4 clone //public/jam/src@all /opt/p4import
	Importing from //public/jam/src@all into /opt/p4import
	Reinitialized existing Git repository in /opt/p4import/.git/
	Import destination: refs/remotes/p4/master
	Importing revision 4409 (100%)

Now if you go to the `/opt/p4import` directory and run `git log`, you can see your imported work.

	$ git log -2
	commit 1fd4ec126171790efd2db83548b85b1bbbc07dc2
	Author: Perforce staff <support@perforce.com>
	Date:   Thu Aug 19 10:18:45 2004 -0800

	    Drop 'rc3' moniker of jam-2.5.  Folded rc2 and rc3 RELNOTES into
	    the main part of the document.  Built new tar/zip balls.

	    Only 16 months later.

	    [git-p4: depot-paths = "//public/jam/src/": change = 4409]

	commit ca8870db541a23ed867f38847eda65bf4363371d
	Author: Richard Geiger <rmg@perforce.com>
	Date:   Tue Apr 22 20:51:34 2003 -0800

	    Update derived jamgram.c

	    [git-p4: depot-paths = "//public/jam/src/": change = 3108]

You can see the `git-p4` identfier in each commit.  If you want to keep that there in case you need to reference it later, that's perfectly fine.  However, if you would like to remove it, now is the time to do so - before you start doing work on the new repository.  You can use `git filter-branch` to remove them en masse.

	$ git filter-branch --msg-filter '
	        sed -e "/^\[git-p4:/d"
	'
	Rewrite 1fd4ec126171790efd2db83548b85b1bbbc07dc2 (123/123)
	Ref 'refs/heads/master' was rewritten

Now if you run a `git log`, you can see that all of the SHA-1 checksums for the commits have changed, but you don't have the `git-p4` strings in the commits messages anymore.

	$ git log -2
	commit 10a16d60cffca14d454a15c6164378f4082bc5b0
	Author: Perforce staff <support@perforce.com>
	Date:   Thu Aug 19 10:18:45 2004 -0800

	    Drop 'rc3' moniker of jam-2.5.  Folded rc2 and rc3 RELNOTES into
	    the main part of the document.  Built new tar/zip balls.

	    Only 16 months later.

	commit 2b6c6db311dd76c34c66ec1c40a49405e6b527b2
	Author: Richard Geiger <rmg@perforce.com>
	Date:   Tue Apr 22 20:51:34 2003 -0800

	    Update derived jamgram.c

Now your import is ready to push up to your new Git server.

### A Custom Importer ###

If you have a system that is not Subversion or Perforce you should look for an importer online - there are quality importers for CVS, Clear Case, Visual Source Save, even a directory of archives.  If one of these tools doesn't work for you, you have a rarer tool or you otherwise need a more custom importing process, you will want to use `git fast-import`.  `fast-import` is a command that reads simple instructions from stdin to write specific Git data.  It is much easier to create Git objects this way than to run the raw Git commands or try to write the raw objects (see Chapter 9 for more information on that).  This way you can write an import script that simply reads the neccesary information out of whatever system you're importing from and prints straightforward instructions to stdout.  You can then run this program and pipe it's output through `git fast-import`.

dwp: It's Visual SourceSafe I think. {: class=note}

To quickly demonstrate this tool, we'll write an simple importer.  Let's say that we've been backing up our project by just copying the directory every once in a while into a timestamped backup directory and we want to import this into Git.  Our directory structure looks like this:

	$ ls /opt/import_from
	back_2009_01_02	
	back_2009_01_04	
	back_2009_01_14	
	back_2009_02_03	
	current
	
Where we work in 'current' and copy that directory into a 'back\_YYYY\_MM\_DD' directory occasionally. 

Now, in order to import a Git directory, we'll have to review how Git actually stores it's data.  As you may remember, Git is fundamentally a linked list of commit objects that point to a snapshot of content, so all we have to do is tell `fast-import` what the content snapshots are and then what the commit data is that points to them and in what order they go.  So our strategy will be to go through the snapshots one at a time and create commits with the contents of each directory, linking each commit back to the previous one.

dwp: Say what language you're going to write this in. You're going to need to explain the code quite carefully - don't rely on your reader knowing Ruby. I would say what the output you are coding towards is before giving the code each time. That way, someone who doesn't understand Ruby will be able to follow it more easily. {: class=note}

To begin we'll change into the target directory and identify every subdirectory in there, each of which will be a snapshot that we want to import as a commit.  So for each subdirectory, we'll change into that and print the commands neccesary to export it.  Our basic main loop will look like this:

	last_manifest = {}
	last_mark = nil

	# loop through the directories
	Dir.chdir(ARGV[0]) do
	  Dir.glob("*").each do |dir|
	    next if File.file?(dir)

	    # move into the target directory
	    Dir.chdir(dir) do 
	      (last_mark, last_manifest) = print_export(dir, last_mark, last_manifest)
	    end
	  end
	end

So now we're running `print_export` inside each directory, which takes the manifest and mark of the previous snapshot and returns the manifest and mark of this one, so we can link them up properly.  A 'mark' is the `fast-export` term for an identifier that we can give to commits to refer to them later, so as we create commits, we give each one a mark that we can use to link to them from other commits later.  So, the first thing we want to do in our `print_export` method is to generate a mark from the directory name.

	mark = convert_dir_to_mark(dir)

The way we're going to do this is to have an array of directories and just use the index value as the mark, since marks have to be an integer.  Our method will look like this:

	$marks = []
	def convert_dir_to_mark(dir)
	  if !$marks.include?(dir)
	    $marks << dir
	  end
	  ($marks.index(dir) + 1).to_s
	end

Now that we have an integer representation of our commit, we need a date for the commit metadata.  Since the date is expressed in the name of the directory, we'll parse it out.  So, our next line in our `print_export` file will be:

	date = convert_mark_to_date(dir)

where `convert_mark_to_date` is defined as:

	def convert_mark_to_date(mark)
	  if mark == 'current'
	    return Time.now().to_i
	  else
	    mark = mark.gsub('back_', '')
	    (year, month, day) = mark.split('_')
	    return Time.local(year, month, day).to_i
	  end
	end

dwp: I may have missed the point, but aren't all your marks integers? How can one be equal to 'current'? {: class=note}

That will return an integer value for the date of each directory.  The last peice of meta-information we'll need for each commit is the committer data, which we'll just hardcode in a global variable.

	$author = 'Scott Chacon <schacon@geemail.com>'

Now we're ready to start printing out the commit data for our importer.  The initial information will state that we're defining a commit object and what branch it will be on, followed by the mark we've generated for it, followed by the committer information and commit message, then the commit that came before it, if there was one.  Our code will look like this.

	# print the import information
	puts 'commit refs/heads/master'
	puts 'mark :' + mark
	puts "committer #{$author} #{date} -0700"
	export_data('imported from ' + dir)
	puts 'from :' + last_mark if last_mark

dwp: What's the -700 in there? Is that a timezone? {: class=note}

Now, the commit message has to be expressed in a special format, which is like this:

	data <<MARKER
	(data)
	MARKER

If you're familiar with Perl or other scripting languages, you may have seen this syntax before.  Since we'll need to use the same format for specifying file contents later, I've created a helper method for us, `export_data`, which looks like this:

	def export_data(string)
	  puts "data <<EOF"
	  puts string
	  puts 'EOF'
	end

sop: Its better to use the "data nnnn\n" format, where nnnn is the number of bytes being written.  This avoids issues with the chosen delimiter appearing in the data string itself.  {: class=note}

So now all that is left is specifying the file contents for each of the snapshots.  Since many systems you might be importing from think of their revisions as changes from one commit to another, `fast-import` takes commands with each commit to specify which files have been added or removed or modified and what the new contents are.  This is why we're passing a manifest from one call to the next, so we can review what has changed and tell the importer.  The first thing we have to do is build the current manifest.

	current_manifest = {}
	Dir.glob("**/*").each do |dir|
	  next if !File.file?(dir)
	  sha = Digest::SHA1.file(dir).hexdigest
	  current_manifest[dir] = sha
	end

sop: This is way more complex than it needs to be.  Just use "deleteall" and dump the entire file contents, and let fast-import deal with it.  Though you should mention as an aside that you can also feed only the list of modified files (adds, deletes, changes) if the source is better matched that way.   {: class=note}

That snippet will go through every file in that directory recursively, generate a checksum of the contents of that file and putting it in an associative array so we can compare manifests later to see if anything has changed. We're using SHA-1 checksums of the file contents here to determine if contents have changed from revision to revision, but it could just as easily be MD5 or anything else - this is not the same as the Git SHA-1 checksum of file contents, Git appends other information - see Chapter 9 for details on that.

So, now that we have an associative array that looks something like this:

	{ 'file1' => 7b310e7390046d753a0883445866a77ab0a4e68a,
	  'file2' => b45124a5212dc84c42435f8f4061ceb8fea041cf }

We want to compare it to the `last_manifest` variable that was passed in that will have the same structure to determine what files have been added, removed and modified.  First let's look for the added files.  All we have to do is see which keys are in the current manifest that were not in the last one. Then for each file, we have to tell the importer what the contents of that new file are.

	new_files = current_manifest.keys - last_manifest.keys
	new_files.each do |newf|
	  inline_data(newf)
	end

The format for listing out new file contents or specifying a modified file with the new contents is:

	M 644 inline path/to/file
	data <<EOF
	(file contents)
	EOF

Where '644' is the mode (if you have executable files, you'll need to detect and specify '755' instead), 'inline' says you're going to list the contents right after this line and then the path of the file.  So, our `inline_data` method looks like this:

	def inline_data(file, code = 'M', mode = '644')
	  content = File.read(file)
	  puts "#{code} #{mode} inline #{file}"
	  export_data(content)
	end

Where we're reusing our export\_data method, since it's the same as the way we had to specify our commit message data.  Next we have to list out the files that have been deleted since the last commit, so we obtain that list in much the same way, by finding the keys in the last manifest that aren't in the current one.

	# remove deleted files
	removed_files = last_manifest.keys - current_manifest.keys
	removed_files.each do |del|
	  puts "D #{del}"
	end

The format for specifying files that have been deleted is simply:

	D path/to/file

So our code accomplishes that.  All that is left is to tell the importer which files have been modified.  To do that, we'll iterate through the current manifest and look for files that were also in the last manifest but had a different checksum.  For each file that we find, we can simply call the same `inline_data` method that we called to add a file.

	# modify changed files
	current_manifest.each do |filenm, sha| 
	  next if !last_manifest.has_key?(filenm)
	  if last_manifest[filenm] != sha
	    inline_data(filenm)
	  end
	end

The last thing we need to do is to return the current mark and manifest so it can be passed to the next iteration:

	return [mark, current_manifest]

That's it.  To view the full version of this script, see Appendix A. Now if we run this script, we will get content that looks something like this:

	$ ruby import.rb /opt/import_from 
	commit refs/heads/master
	mark :1
	committer Scott Chacon <schacon@geemail.com> 1230883200 -0700
	data <<EOF
	imported from back_2009_01_02
	EOF
	M 644 inline file.rb
	data <<EOF
	(file contents)
	EOF

	commit refs/heads/master
	mark :2
	committer Scott Chacon <schacon@geemail.com> 1231056000 -0700
	data <<EOF
	imported from back_2009_01_04
	EOF
	from :1
	M 644 inline new.rb
	data <<EOF
	(file contents)
	EOF
	M 644 inline file.rb
	data <<EOF
	(file contents)
	EOF
	...

To actually run the importer, we simply pipe this output through `git fast-import` while in the Git directory we want to import into.  We can just create a new directory and then run `git init` in it for a starting point, then run our script.

	$ git init
	Initialized empty Git repository in /opt/import_to/.git/
	$ ruby import.rb /opt/import_from | git fast-import
	git-fast-import statistics:
	---------------------------------------------------------------------
	Alloc'd objects:       5000
	Total objects:           18 (         1 duplicates                  )
	      blobs  :            7 (         1 duplicates          0 deltas)
	      trees  :            6 (         0 duplicates          1 deltas)
	      commits:            5 (         0 duplicates          0 deltas)
	      tags   :            0 (         0 duplicates          0 deltas)
	Total branches:           1 (         1 loads     )
	      marks:           1024 (         5 unique    )
	      atoms:              3
	Memory total:          2255 KiB
	       pools:          2098 KiB
	     objects:           156 KiB
	---------------------------------------------------------------------
	pack_report: getpagesize()            =       4096
	pack_report: core.packedGitWindowSize =   33554432
	pack_report: core.packedGitLimit      =  268435456
	pack_report: pack_used_ctr            =          9
	pack_report: pack_mmap_calls          =          5
	pack_report: pack_open_windows        =          1 /          1
	pack_report: pack_mapped              =       1356 /       1356
	---------------------------------------------------------------------

As you can see, when it completes successfully it will give you a bunch of statistics about what it accomplished.  In this case we imported 18 objects total for 5 commits into 1 branch.  Now we can run `git log` to see our new history.

	$ git log -2
	commit 10bfe7d22ce15ee25b60a824c8982157ca593d41
	Author: Scott Chacon <schacon@geemail.com>
	Date:   Sun May 3 12:57:39 2009 -0700

	    imported from current

	commit 7e519590de754d079dd73b44d695a42c9d2df452
	Author: Scott Chacon <schacon@geemail.com>
	Date:   Tue Feb 3 01:00:00 2009 -0700

	    imported from back_2009_02_03

There you go, a nice clean Git repository. It's important to note that you won't have anything checked out - you will not have any files in your working directory at first.  If you want to get them, you need to reset your branch to where 'master' is now.

	$ ls
	$ git reset --hard master
	HEAD is now at 10bfe7d imported from current
	$ ls
	file.rb  lib

There is a lot more that you can do with the `fast-import` tool - handle different modes, binary data, multiple branches and merging, tags, progress indicators and more.  There are a number of examples of more complex scenarios in the `contrib/fast-import` directory of the Git source code, one of the better ones being the `git-p4` script that we just covered.

### Summary ###

Now you should feel comfortable using Git with Subversion or being able to import nearly any existing repository into a new Git one without losing data.  If these examples still don't meet your needs, the next Chapter will cover the raw internals of Git so you can craft every single byte if needs be.
