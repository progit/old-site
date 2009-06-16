## An Example Git Enforced Policy ##

So now we're going to use what we've learned to establish a Git workflow that checks for a custom commit message format, enforces fast-forward only pushes and only allows certain users to modify certain subdirectories in a project.  We'll build client scripts that help the developer know if their push will be rejected and server scripts that actually enforce the policies.

 * enforced custom commit message format
 * only allows certain users to modify certain subdirectories in a project
 * enforces fast-forward only pushes 

I'll be using Ruby to write these, both because it's my preferred scripting language and I feel it's the most pseudo-code looking of the scripting languages so it should be roughly followable even if you don't use it. However, any language will work fine.  All of the sample hook scripts distributed with Git are either in Perl or Bash scripting, so you also have plenty of examples of hooks in those languages just by looking at the sample ones.

### Server Side Hook ###

All of the server side work will go into the 'update' file in our hooks directory.  The `update` file runs once per branch being pushed and takes the reference being pushed to, the old revision that branch was at and the new revision being pushed.  We also have access to the user doing the pushing if it's being run over SSH.  If you've allowed everyone to connect with a single user (like 'git') via public key authentication, you may have to give that user a shell wrapper that determines which user is connecting based on the public key and set an environment variable for that. Here we'll assume the connecting user is in the `$USER` environment variable, so our `update` script will begin by gathering all the information we need.

	#!/usr/bin/env ruby

	$refname = ARGV[0]
	$oldrev  = ARGV[1]
	$newrev  = ARGV[2]
	$user    = ENV['USER']
	
	puts "Enforcing Policies... \n(#{$refname}) (#{$oldrev[0,6]}) (#{$newrev[0,6]})"

Yes, I'm using global variables - don't judge me, it's just easier to demonstrate simply in this manner.  

#### Enforcing a Specific Commit Message Format ####

So, our first challenge is to enforce that each commit message must adhere to a particular format.  Just to have a target, let's say that each message has to have a string in it that looks like "[ref: 1234]" because we want each commit to link to a work item in our ticketing system.  So, what we have to do is look at each commit being pushed up, see if that is in the commit message and if it is absent from any of them, exit non-zero so the push is rejected.

Now, we can get a list of the SHA-1 values of all the commits that are being pushed by taking the `$newrev` and `$oldrev` values and passing them to a Git plumbing command called `git rev-list`.  This is basically the `git log` command, but by default it only prints out the SHA-1 values and no other information.  So, to get a list of all the commit SHAs introduced between one commit SHA and another, we can run something like this:

	$ git rev-list 538c33..d14fc7
	d14fc7c847ab946ec39590d87783c69b031bdfb7
	9f585da4401b0a3999e84113824d15245c13f0be
	234071a1be950e2a8d078e6141f5cd20c1e61ad3
	dfa04c9ef3d5197182f13fb5b9b1fb7717d2222a
	17716ec0f1ff5c77eff40b7fe912f9f6cfd0e475

Now we can take that output, loop through each of those commit SHAs, grab the message for it and test that message against a regular expression that looks for a pattern.  

We have to figure out how to get the commit message from each of these commits to test.  To get the raw commit data, we can use another plumbing command called `git cat-file`.  We'll go over all these plumbing commands in detail in Chapter 9, but for now, let's see what that command gives us:

	$ git cat-file commit ca82a6
	tree cfda3bf379e4f8dba8717dee55aab78aef7f4daf
	parent 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	author Scott Chacon <schacon@gmail.com> 1205815931 -0700
	committer Scott Chacon <schacon@gmail.com> 1240030591 -0700

	changed the verison number

A simple way to just get the commit message from a commit when you have the SHA-1 value is to go to the first blank line and just take everything after that.  We can do that with the `sed` command on Unix systems:

	$ git cat-file commit ca82a6 | sed '1,/^$/d'
	changed the verison number

Now we can use that incantation to grab the commit message from each commit that is trying to be pushed and exit if we see anything that doesn't match. To exit the script and reject the push, we simply have to exit non-zero, so our whole method will look like this:

	$regex = /\[ref: (\d+)\]/

	# enforced custom commit message format
	def check_message_format
	  missed_refs = `git rev-list #{$oldrev}..#{$newrev}`.split("\n")
	  missed_refs.each do |ref|
	    message = `git cat-file commit #{ref} | sed '1,/^$/d'`
	    if !$regex.match(message)
	      puts "[POLICY] Your message is not formatted correctly"
	      exit 1
	    end
	  end
	end
	
	check_message_format

So, putting that in our `update` script will reject updates that contain commits that have messages that do not adhere to our rule.

### Enforcing a User-based ACL System ###

Now let's say that we want to add a mechanism where we have an access control list (ACL) that specifies which users are allowed to push changes to which parts of our projects.  Some people have full access, while others only have access to push changes to certain subdirectories or specific files.  To enforce this, we're going to write those rules to a file named 'acl' that lives in our bare Git repository on the server, and we'll have the `update` hook look at that, see what files are being introduced for all the commits being pushed and see if the user doing the push has access to update all those files.

The first thing we'll do is write our access control list.  Here I'm going to use a format very much like the CVS ACL mechanism, where we have a series of lines where the first field is 'avail' or 'unavail', the next field is a comma delimited list of the users that the rule applies to and the last field is the path that the rule applies to - blank being open access.  All of these fields are delimited by a pipe (`|`) character.

So, if we had a couple of administrators, some documentation writers with access to the 'doc' directory and one developer that only had access to the 'lib' and 'tests' directories, we would have an ACL file that looked like this:

	avail|nickh,pjhyett,defunkt,tpw
	avail|usinclair,cdickens,ebronte|doc
	avail|schacon|lib
	avail|schacon|tests

So, the first thing we want to do is read this data into a structure that we can use.  In this case, I'm only going to enforce the 'avail' directives to keep it simple.  Here is a method that will give me an associative array where the key is the user name and the value is an array of paths they have access to write to.
	
	def get_acl_access_data(acl_file)
	  # read in ACL data
	  acl_file = File.read(acl_file).split("\n").reject { |line| line == '' }
	  access = {}
	  acl_file.each do |line|
	    avail, users, path = line.split('|')
	    next unless avail == 'avail'
	    users.split(',').each do |user|
	      access[user] ||= []
	      access[user] << path
	    end
	  end
	  access
	end

On the ACL file we looked at above, this `get_acl_access_data` method will return a data structure that looks like this:

	{"defunkt"=>[nil],
	 "tpw"=>[nil],
	 "nickh"=>[nil],
	 "pjhyett"=>[nil],
	 "schacon"=>["lib", "tests"],
	 "cdickens"=>["doc"],
	 "usinclair"=>["doc"],
	 "ebronte"=>["doc"]}

Now we have the permissions sorted out, so we need to figure out what paths the commits being pushed have modified to make sure the user pushing has access to all of them.
	
We can pretty easily see what files have been modified in a single commit with the `--name-only` option to the `git log` command (which we mentioned briefly in Chapter 2).

	$ git log -1 --name-only --pretty=format:'' 9f585d
	
	README
	lib/test.rb

If we use the ACL structure returned from the `get_acl_access_data` method and check it against the listed files in each of the commits, we can determine if the user in fact has access to push all of their commits or not.

	# only allows certain users to modify certain subdirectories in a project
	def check_directory_perms
	  access = get_acl_access_data('acl')

	  # see if anyone is trying to push something they can't
	  missed_refs = `git rev-list #{$oldrev}..#{$newrev}`.split("\n")
	  missed_refs.each do |ref|
	    files_modified = `git log -1 --name-only --pretty=format:'' #{ref}`.split("\n")
	    files_modified.each do |path|
	      next if path.size == 0
	      has_file_access = false
	      access[$user].each do |access_path|
			if !access_path  # user has access to everything
				|| path.include?(access_path) # user has access to this path
				has_file_access = true 
			end
	      end
	      if !has_file_access
	        puts "[POLICY] You do not have access to push to #{path}"
	        exit 1
	      end
	    end
	  end  
	end

	check_directory_perms

TODO: /include?/starts\_with/ ?

DWP: Now I'm not a Ruby user, but I am a programmer, so I'm probably not a bad person to test this code on... I can understand everything that is going on there except the path.include?(access_path) bit. I would assume that would just check that access_path occurs as a substring of path, but not where it occurs, and I'm assuming we want it as a prefix. {: class=note}

Now our users can't push any commits with badly formed messages or with modified files outside of their designated paths.  

#### Enforcing fast-forward only pushes ####

The only thing left is to enforce fast-forward only pushes.  Now, in Git versions 1.6 or newer, you can  set the `receive.denyDeletes` and `receive.denyNonFastForwards` settings, but in enforcing this with a hook it will work in older versions of Git and we could modify it to only enforce this for certain users and whatever else we come up with later.

The logic for checking this is simply to see if there are any commits that are reachable from the older revision that are not reachable from the newer one.  If there are none, then it was a fast forward push, otherwise we'll deny it.

	# enforces fast-forward only pushes 
	def check_fast_forward
	  missed_refs = `git rev-list #{$newrev}..#{$oldrev}`
	  missed_ref_count = missed_refs.split("\n").size
	  if missed_ref_count > 0
	    puts "[POLICY] Cannot push a non fast-forward reference"
	    exit 1
	  end
	end

	check_fast_forward

So now everything is all setup.  If you run `chmod u+x [gitdir]/hooks/update`, which is the file you should have put all of this code into, then if you try to push a non fast-forwarded reference, you'll get something like this:

	$ git push -f origin master
	Counting objects: 5, done.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 323 bytes, done.
	Total 3 (delta 1), reused 0 (delta 0)
	Unpacking objects: 100% (3/3), done.
	Enforcing Policies... 
	(refs/heads/master) (8338c5) (c5b616)
	[POLICY] Cannot push a non-fast-forward reference
	error: hooks/update exited with error code 1
	error: hook declined to update refs/heads/master
	To git@gitserver:project.git
	 ! [remote rejected] master -> master (hook declined)
	error: failed to push some refs to 'git@gitserver:project.git'

There are a couple of interesting things here.  First of all, you will see this at the beginning of where the hook started running.

	Enforcing Policies... 
	(refs/heads/master) (fb8c72) (c56860)

Notice that we printed that out to stdout at the very beginning of our `update` script.  It is important to note that anything your script prints to stdout will be transferred to the client.

The next thing you will notice is the error message.

	[POLICY] Cannot push a non fast-forward reference
	error: hooks/update exited with error code 1
	error: hook declined to update refs/heads/master

The first line was printed out by us, the other two were Git telling you that the `update` script exited non-zero and that is what is declining your push.  Lastly, we have this:

	To git@gitserver:project.git
	 ! [remote rejected] master -> master (hook declined)
	error: failed to push some refs to 'git@gitserver:project.git'

You will see a 'remote rejected' message for each reference that your hook declined, and it tells you that it was declined specifically because of a hook failure.

Furthermore, if the ref marker is not there in any of your commits, you'll see the error message we're printing out for that.

	[POLICY] Your message is not formatted correctly

Or if someone tries to edit a file they don't have access to and push a commit containing it, they will see something similar.  For instance, if a documentation author tries to push a commit modifying something in the 'lib' directory, they will see:

	[POLICY] You do not have access to push to lib/test.rb

That's all.  From now on, as long as that `update` script is there and executable, your repository will never be rewound, will never have a commit message without your pattern in it and your users will be sandboxed.

### Client Side Hooks ###

The downside to this is the whining that will inevitably result from your users having their commit pushes rejected.  Having work carefully crafted and then rejected at the last minute can be extremely frustrating and confusing, and furthermore they will have to edit their history to correct it, which is not always for the feint of heart.

The answer to this dilemma is to have some client side hooks that they can setup to be notified when they are doing something that is likely to be rejected from the server.  That way they can correct it before committing and before it becomes more difficult to fix.  Since hooks are not transferred with a clone of a project, you'll have to distribute these scripts some other way and then have your users copy them to their `.git/hooks` directory and make them executable. You can distribute these hooks within the project itself or in a seperate project altogether, but there is no way to set them up automatically.

So, first off, we want to check our commit message right before the commit gets recorded so that we know the server won't reject our changes for badly formatted commit messages.  To do this, we can add the `commit-msg` hook.  If we simply have it read the message from the file passed as the first argument and compare that to the pattern, we can force Git to abort the commit if there is no match.

	#!/usr/bin/env ruby
	message_file = ARGV[0]
	message = File.read(message_file)
	
	$regex = /\[ref: (\d+)\]/
	
	if !$regex.match(message)
	  puts "[POLICY] Your message is not formatted correctly"
	  exit 1
	end

DWP: It would be good to always say where these things need to be. {: class=note}
	
If that is in place and executable, then if you committed with a message that is not properly formatted, you'll see this:

	$ git commit -am 'test'
	[POLICY] Your message is not formatted correctly

No commit was completed in that instance. However, if your message contains the proper pattern, then Git will allow you to commit.

	$ git commit -am 'test [ref: 132]'
	[master e05c914] test [ref: 132]
	 1 files changed, 1 insertions(+), 0 deletions(-)

Next we want to make sure that we're not modifying files that are outside of our ACL scope.  If we have a copy of the ACL file we used previously in our projects `.git` directory, then the following `pre-commit` script will enforce those constraints for us.

	#!/usr/bin/env ruby

	$user    = ENV['USER']

	# [ insert acl_access_data method from above ]
	
	# only allows certain users to modify certain subdirectories in a project
	def check_directory_perms
	  access = get_acl_access_data('.git/acl')

	  files_modified = `git diff-index --cached --name-only HEAD`.split("\n")
	  files_modified.each do |path|
	    next if path.size == 0
	    has_file_access = false
	    access[$user].each do |access_path|
	      has_file_access = true if !access_path || path.include?(access_path)
	    end
	    if !has_file_access
	      puts "[POLICY] You do not have access to push to #{path}"
	      exit 1
	    end
	  end
	end

	check_directory_perms

This is roughly the same script as the server side part of this, but with two important differences. The first is that the ACL file is in a different place, since this script runs from your working directory, not from your Git directory, you have to change the path to the ACL file from this

	access = get_acl_access_data('acl')

to this
	
	access = get_acl_access_data('.git/acl')

The other important difference is the way you get a listing of the files that have been changed.  Since the server side method is looking at the log of commits and at this point the commit has not been recorded yet, we have to get our file listing from the staging area instead.  So instead of 

	files_modified = `git log -1 --name-only --pretty=format:'' #{ref}`

we are going to have to use

	files_modified = `git diff-index --cached --name-only HEAD`

But those are really the only two differences - otherwise the script works exactly the same way.  One caveat is that it expects you to be running locally as the same user you push as to the remote machine.  If that is different, you'll have to set the `$user` variable manually.

The last thing we'll have to do is to check that you're not trying to push non fast-forwarded references, but that is a bit less common.  In order to get a reference that is not a fast forward, you either have to rebase past a commit you've already pushed up, or you have to try pushing a different local branch up to the same remote branch.  

Since the server will tell you that you can't push a non-fast-forward anyways and the hook is really only preventing forced pushes, the only accidental thing we can try to catch is to not let you rebase any commits that have already been pushed.

Here is an example `pre-rebase` script that will check for exactly that.  It will get a list of all the commits you're about to rewrite and checks to see if they exist in any of your remote references.  If it sees one that is reachable from one of your remote references, it aborts the rebase.

	#!/usr/bin/env ruby

	base_branch = ARGV[0]
	if ARGV[1]
	  topic_branch = "refs/heads/" + ARGV[1]
	else
	  topic_branch = `git symbolic-ref HEAD`.chomp
	end

	target_shas = `git rev-list #{base_branch}..#{topic_branch}`.split("\n")
	remote_refs = `git branch -r`.split("\n").map { |r| r.strip }

	target_shas.each do |sha|
	  remote_refs.each do |remote_ref|
	    already_pushed = (`git rev-list ^#{sha}^@ refs/remotes/#{remote_ref}`.chomp == sha)
	    if already_pushed
	      puts "[POLICY] Commit #{sha} has already been pushed to #{remote_ref}"
	      exit 1
	    end
	  end
	end

sop: I don't recall explaining ^@ syntax, indeed I had to really dig for that one myself, its not very common.  A note here, maybe a comment in the code, would be really helpful.  {: class=note}

The main drawback to this approach is that it can be very slow and is often unneccesary, since if you don't try to force the push with a `-f`, the server will warn you and not accept the push anyhow.  However, it is an interesting exercise and would in theory help you not do a rebase that you might later have to go back and fix.

### Summary ###

Now we've covered most of the major ways that you can customize your Git client and server to best fit your workflow and projects.  We have covered all sorts of configuration settings, file based attributes and event hooks and built an example policy enforcing server.  You should now be able to make Git fit nearly any workflow you can dream up.
