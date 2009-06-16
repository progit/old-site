## Viewing Commit History ##

Once you have done this several times, or if you have simply cloned a repository with an existing commit history, you will probably want to look back to see what has happened. The most basic and powerful tool to do this is the `git log` command.  

For these examples, I will use a very simple project that I often use for demonstrating things called ‘simplegit’. To get the project yourself, run `git clone git://github.com/schacon/simplegit-progit.git`.

** TODO: change simplegit to simplegit-progit at github with master at 5e872693dcc722c75fa0ad87c02a4ba11a813438 **

When you run `git log` in that project, you should get output that looks something like this:

	$ git log
	commit 5e872693dcc722c75fa0ad87c02a4ba11a813438
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Mon Mar 17 21:52:11 2008 -0700

	    changed the verison number

	commit 839853458dfaba79499e3dca6981b83e11a285c2
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Sat Mar 15 16:40:33 2008 -0700

	    removed unnecessary test code

	commit a11bef06a3f659402fe7563abf99ad00de2209e6
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Sat Mar 15 10:31:28 2008 -0700

	    first commit


** TODO: replace gmail with geemail? **

DWP: I'm assuming you don't mind a lot of spam on your gmail account... We usually advise authors not to put their own email address in places like this. {: class=note}

By default, with no arguments, `git log` will list the commits made in that repository in reverse chronological order. That is, the most recent commits show up first. As you can see, this command lists each commit with their SHA-1 checksum, the author’s name and email, the date written and the commit message.

There are a huge number and variety of options to the `git log` command to show you exactly what you’re looking for. Here we will show you some of the most often used options. 

One of the more helpful options is the `-p`, which shows the diff introduced in each commit.  I’m also going to use `-2`, which will limit the output to only the last 2 entries.

	$ git log –p -2
	commit 5e872693dcc722c75fa0ad87c02a4ba11a813438
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Mon Mar 17 21:52:11 2008 -0700

	    changed the verison number

	diff --git a/Rakefile b/Rakefile
	index a874b73..8f94139 100644
	--- a/Rakefile
	+++ b/Rakefile
	@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
	 spec = Gem::Specification.new do |s|
	-    s.version   =   "0.1.0"
	+    s.version   =   "0.1.1"
	     s.author    =   "Scott Chacon"

	commit 839853458dfaba79499e3dca6981b83e11a285c2
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Sat Mar 15 16:40:33 2008 -0700

	    removed unnecessary test code

	diff --git a/lib/simplegit.rb b/lib/simplegit.rb
	index a0a60ae..47c6340 100644
	--- a/lib/simplegit.rb
	+++ b/lib/simplegit.rb
	@@ -18,8 +18,3 @@ class SimpleGit
	     end
   
	 end
	-
	-if $0 == __FILE__
	-  git = SimpleGit.new
	-  puts git.show
	-end
	\ No newline at end of file

So, you can see that displays the same information, but with a diff directly following each entry.  This is very helpful for code review or to quickly browse what exactly has happened on a series of commits that a collaborator has added. 

There are also a series of summarizing options you can use with `git log`.  For example, if you want to see some abbreviated stats on each commit, you can use the `--stat` option, which will display something like this:

	$ git log --stat 
	commit 5e872693dcc722c75fa0ad87c02a4ba11a813438
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Mon Mar 17 21:52:11 2008 -0700

	    changed the verison number

	 Rakefile |    2 +-
	 1 files changed, 1 insertions(+), 1 deletions(-)

	commit 839853458dfaba79499e3dca6981b83e11a285c2
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Sat Mar 15 16:40:33 2008 -0700

	    removed unnecessary test code

	 lib/simplegit.rb |    5 -----
	 1 files changed, 0 insertions(+), 5 deletions(-)

	commit a11bef06a3f659402fe7563abf99ad00de2209e6
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Sat Mar 15 10:31:28 2008 -0700

	    first commit

	 README           |    6 ++++++
	 Rakefile         |   23 +++++++++++++++++++++++
	 lib/simplegit.rb |   25 +++++++++++++++++++++++++
	 3 files changed, 54 insertions(+), 0 deletions(-)
	
As you can see here, the `--stat` option will print below each commit entry a list of which files were modified, how many files were changed and in those how many lines were added and removed.  It also puts a simple summary of all that information at the end.

Another really useful option is `--pretty`.  This option will change the log output to different formats from the default.  There are a few prebuilt options for you to use. The ‘oneline’ option will print each commit on a single line, which is useful if you are looking at a lot of commits.  There is also ‘short’, ‘full’ and ‘fuller’ which show the output in roughly the same format but with less or more information respectively.  

	$ git log --pretty=oneline
	5e872693dcc722c75fa0ad87c02a4ba11a813438 changed the verison number
	839853458dfaba79499e3dca6981b83e11a285c2 removed unnecessary test code
	a11bef06a3f659402fe7563abf99ad00de2209e6 first commit

The really interesting one is ‘format’, which allows you to specify your own log output format.  This is especially useful when generating output for machine parsing – since you specify the format explicitly; you know it won’t change with updates to Git.

	$ git log --pretty=format:"%h - %an, %ar : %s"
	5e87269 - Scott Chacon, 11 months ago : changed the verison number
	8398534 - Scott Chacon, 11 months ago : removed unnecessary test code
	a11bef0 - Scott Chacon, 11 months ago : first commit

Here are some of the more useful options that 'format' will take.

Option | Description of Output |
-------|-----------------------|
cell   | center-align          | 
%H     | commit hash |
%h	   | abbreviated commit hash |
%T	   | tree hash |
%t	   | abbreviated tree hash |
%P	   | parent hashes |
%p	   | abbreviated parent hashes |
%an	   | author name |
%ae	   | author email |
%ad	   | author date (format respects --date= option) |
%ar	   | author date, relative |
%cn	   | committer name |
%ce	   | committer email |
%cd	   | committer date |
%cr	   | committer date, relative |
%s	   | subject |

You may be wondering what the difference between 'author' and 'committer' is here.  The 'author' is the person who originally committed the work, where the 'committer' is the person who last committed the work.  So if you send in a patch to a project and one of the core members applies the patch, you both get credit - you as the 'author' and the core member as the 'committer'.  We'll cover this distinction a bit more in Chapter 5.

The ‘oneline’ and ‘format’ options are particularly useful with another log option called `--graph`.  This option will add a nice little ASCII graph showing your branch and merge history.

	$ git log --pretty=format:"%h %s" --graph
	* 2d3acf9 ignore errors from SIGCHLD on trap
	*  5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
	|\  
	| * 420eac9 Added a method for getting the current branch.
	* | 30e367c timeout code and tests
	* | 5a09431 add timeout protection to grit
	* | e1193f8 support for heads with slashes in them
	|/  
	* d6016bc require time for xmlschema
	*  11d191e Merge branch 'defunkt' into local

Those are only some simple output formatting options to ‘git log’, but there are many more.  Here is a list of these and some other common options, along with how they change the output of the ‘log’ command.

| Option	| Description  |
|-----------|--------------|
|	`-p`		| Show the patch introduced with each commit |
|	`--stat`	| Show statistics on files modified in each commit | 
|	`--shortstat`	| Display only the ‘changed/insertions/deletions’ line from the `--stat` command
|	`--name-only`	| Show the list of files modified after the commit information
|	`--abbrev-commit`	| Only show the first few characters of the SHA-1 checksum instead of all 40
|	`--relative-date`	| Display the date in a relative format (eg: “2 weeks ago”) instead of the full date format
|	`--graph`	| Display an ASCII graph of the branch and merge history beside the log output
|	`--pretty` |	Show commits in an alternate format.  Options include ‘oneline’, ‘short’, ‘full’, ‘fuller’ and ‘format’ (where you specify your own format)

### Ranging and limiting

In addition to output formatting options, ‘git log’ takes a number of useful limiting options.  That is, options that let you show only a particular subset of commits.  We’ve seen one such option already – the ‘-2’ option, which only showed us the last two commits.  In fact, you can do ‘-n’ where n is any integer to show the last n commits.  In reality, you’re probably unlikely to use that too often, since Git by default will pipe all output through a pager so you only see one page of log output at a time anyhow.

However, some very useful options are the time limiting options such as `--since` and `--until`.  For example, to get the list of commits made in the last two weeks:

	$ git log --since=2.weeks

This works with lots of formats – you can specify a specific date (“2008-01-15”) or a relative date such as “2 years 1 day 3 minutes ago”.

You can also filter the list to commits that match some search criteria. The `--author` option will allow you to filter on a specific author, or the `--grep` option will let you search for keywords in the commit messages.  (Note that if you want to specify both author and grep options, you have to add a `--all-match` or it will match commits with either).

The last really useful option to pass to `git log` as a filter is a path.  If you specify a directory or file name you can limit the log output only to commits that introduced a change to those files.  This is always the last option and is generally preceded by a ` -- ` to separate the paths from the options. 
 
|	Option	| Description |
|-----------|-------------|
|	`-(n)`	| Show only the last n commits |
|	`--since`, `--after` | Limit the commits to only commits made after the specified date
|	`--until`, `--before` |	Limit the commits to only commits made before the specified date
|	`--author` | Only show commits in which the author entry matches the specified string
|	`--committer` | Only show commits in which the committer entry matches the specified string
|	`--grep`	| Limit the output to commits where the commit message matches the specified string
|	`--no-merges`	| Do not show merge commits

For example, if we wanted to see which commits in the Git source code history Junio Hamano had commited that were not merges in the month of October in 2008, I can run something like this:

	$ git log --pretty="%h:%s" --author=gitster --since="2008-10-01" --before="2008-11-01" --no-merges t/
	5610e3b - Fix testcase failure when extended attribute
	acd3b9e - Enhance hold_lock_file_for_{update,append}()
	f563754 - demonstrate breakage of detached checkout wi
	d1a43f2 - reset --hard/read-tree --reset -u: remove un
	51a94af - Fix "checkout --track -b newbranch" on detac
	b0ad11e - pull: allow "git pull origin $something:$cur

Of the nearly 20,000 commits in the Git source code history, this command will show me the 6 that match those criteria. 

### Using a GUI to visualize history

If you like to use a more graphical tool to visualize your commit history, you may want to take a look at a Tcl/Tk program that is distributed with git called ‘gitk’.  Gitk is basically a visual ‘git log’ tool and in fact will accept nearly all of the filtering options that ‘git log’ does.  If you type ‘gitk’ on the command line in your project, you should see something like this:

![Figure 2.1](/images/fig21.png)


You can see the commit history in the top half along with a nice ancestry graph, and a diff viewer in the bottom half that will show you the changes introduced at any commit you click on.
 
