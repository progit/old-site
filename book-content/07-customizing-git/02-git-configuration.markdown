## Git Configuration ##

As we breifly covered in the very first chapter, you can set Git configuration settings with the `git config` command.  One of the first things we did was setup your name and email address with:

	$ git config --global user.name "John Doe"
	$ git config --global user.email johndoe@example.com

Now we'll go over a few of the more interesting options that you can set with Git in this manner to customize your Git usage.

We covered some simple Git configuration details in the first chapter, but I'll go over them again quickly here.  Git uses a series of configuration files to determine non-default behaviour that you might want. The first place Git will look for these values is in an '/etc/gitconfig' file, which contains values for every user on the system and all of their repositories.  If you pass the option '--system' to git-config, it will read and write from this file specifically. The next place it will look is in the '~/.gitconfig' file, which is specific to your user.  You can make Git read and write to this file specifically by passing the '--global' option. Finally, Git will look for configuration values in the 'config' file in the Git Directory (ie: '.git/config') of whatever repository you are currently using.  These values are specific to that single repository. Each level will overwrite values in the previous level, so values in `.git/config` will trump those in `/etc/sysconfig` for instance.  You can also set them by manually editing the file and inserting the correct syntax, but it's generally easier to run the `git config` command.

### Basic Client Configuration ###

The configuration options recognized by Git fall into two categories - client side and server side.  The majority of the options are client side - configuring up your personal working preferences.  Though there are tons of options, we're only going to cover the few that are either commonly used or can significantly affect your workflow somehow.  There are a lot of options that are really only useful in edge cases that we won't go over here.  If you want to see a list of all the options that your version of Git will recognize, you can run:

	$ git config --help
	
The manual page for `git config` lists out all the available options in quite a bit of detail.

#### core.editor ####

By default, Git will use whatever you have set as your default text editor or else fall back to the Vi editor to create and edit your commit and tag messages.  If you want to change that default to something else you can set the `core.editor` setting.

	$ git config --global core.editor emacs

Now no matter what is set as your default shell editor variable, Git will fire up Emacs for editing messages once that is run.

#### commit.template ####

If you set this to the path of a file on your system, it will use this file as the default message when you commit.  For instance, if you create a template file at `$HOME/.gitmessage.txt` that looks like this:

	subject line
	
	what happened
	
	[ticket: X]

Then you can tell Git to use it as the default message filled in your editor when you run `git commit` by setting the `commit.template` configuration value.

	$ git config --global commit.template $HOME/.gitmessage.txt
	$ git commit
	
Then your editor will open to something like this for your placeholder commit message when you commit:

	subject line

	what happened

	[ticket: X]

	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	# modified:   lib/test.rb
	#
	~                                                            
	~                                 
	".git/COMMIT_EDITMSG" 14L, 297C

If you have some sort of commit message policy in place, then putting a template for that policy on your system and configuring Git to use it by default can help increase the chance of that policy being followed regularly.

#### core.pager ####

The `core.pager` setting determines what pager to use when paging long Git output such as `log` and `diff`.  You can either set it to `more` or whatever your favorite pager is (by default it is `less`), or you can turn it off completely by setting it to a blank string.

	$ git config --global core.pager ''

If you run that, then Git will simply output the entire output of all commands, no matter how long they are.

#### user.signingkey ####

If you are making signed annotated tags (as we covered in Chapter 2), setting your GPG signing key as a configuration setting will make things a bit easier.  If you set your key ID like so:

	$ git config --global user.signingkey <gpg-key-id>

Then you can simply sign tags without having to specify it every time with the `git tag` command.

	$ git tag -s <tag-name>

#### core.excludesfile ####

You can put patterns into the .gitignore file in your project to have Git not see them as untracked files or try to stage them when you run `git add` on them, as we discussed in Chapter 2.  However, if you want another file outside of your project to hold those values, or the have more values, you can tell Git where that file is with the `core.excludesfile` setting.  Simply set it to the path of a file that has content similar to what a `.gitignore` file would have.

#### help.autocorrect ####

This option is only available in Git 1.6.1 and later.  If you mistyped a command in Git 1.6, it will show you something like this:

	$ git com
	git: 'com' is not a git-command. See 'git --help'.

	Did you mean this?
		commit

If you set `help.autocorrect` to '1', Git will automatically run the command if it only has one match under this scenario.

### Colors in Git ###

Git can color it's output to your terminal, which can help you visually parse the output quickly and easily.  There are a number of options that help you set the coloring to your preference.

#### color.ui ####

Git will automatically color most of it's output if you ask it to.  You can get very specific in what you want colored and how, but to simply turn on all of the default terminal coloring, you can just set the `color.ui` setting to 'true'.

	$ git config --global color.ui true

When that is set, Git will color it's output if the output is to a terminal.  Other possible settings are 'false' which will never color the output, and 'always' which will set colors all the time, even if you're redirecting Git commands to a file or piping them to another command.  This setting was added in version 1.5.5, so if you have an older version, you'll have to set all the color settings individually.

It is likely pretty rare to actually want `color.ui = always`.  In most scenarios, if you really do want color codes in your redirected output you can instead often pass a `--color` flag to the Git command to force it to use color codes. The `color.ui = true` setting is almost always what you're going to want to use.

#### color.* ####

If you want to be more specific about which commands are colored and how, or have an older version, Git provides verb-specific coloring settings.  Each of these can be set to 'true', 'false' or 'always'.

	color.branch
	color.diff
	color.interactive
	color.status

In addition, each of these has subsettings where you can set specific colors for parts of the output if you want to override them.  For example, to set the meta information in your diff output to blue foreground, black background and bold, you can run:

	$ git config --global color.diff.meta blue black bold

You can set the color to any of 'normal', 'black', 'red', 'green', 'yellow', 'blue', 'magenta', 'cyan' or 'white'.  If you want an attribute like 'bold' in the previous example, you can choose from 'bold', 'dim', 'ul', 'blink' and 'reverse'.

See the `git-config` manpage for all the subsettings that you can configure, if you really want to do that.


### External Merge and Diff Tools ###

Though Git has an internal implementation of diff, which is what we've been using, you can also setup an external tool to do it instead.  You can also setup a graphical merge conflict resolution tool instead of having to do it manually. Here we'll demonstrate setting up Perforce visual merge tool to do our diffs and merge resolutions, since it's a pretty nice graphical tool for this and it is free.

If you want to try this out, Perforce visual merge tool works on all major platforms, so you should be able to do so.  I'll use pathnames in the examples that work on Mac and Linux systems, for Windows you'll have to change `/usr/local/bin` to some executable path in your environment.

You can download Perforce visual merge tool here:

	http://www.perforce.com/perforce/downloads/component.html

Now, to begin, we'll setup external wrapper scripts to run our commands.  I'll use the Mac path for the executable, but in other systems it will simply be where your p4merge binary is installed instead.  We will setup a merge wrapper script named 'extMerge' and will simply call our binary with all the arguments provided.

	$ cat /usr/local/bin/extMerge
	#!/bin/sh
	/Applications/p4merge.app/Contents/MacOS/p4merge $*

The diff wrapper will check to make sure that there are seven arguments provided and will just pass two of them to our merge script.  By default, Git will pass the following arguments to the diff program:

	path old-file old-hex old-mode new-file new-hex new-mode

Since we only want the old-file and new-file arguments, we'll use the wrapper script to only pass the ones we need.

	$ cat /usr/local/bin/extDiff 
	#!/bin/sh
	[ $# -eq 7 ] && /usr/local/bin/extMerge "$2" "$5"

You'll also want to make sure these tools are executable:

	$ sudo chmod +x /usr/local/bin/extMerge 
	$ sudo chmod +x /usr/local/bin/extDiff

Now we can setup our config file to use our custom merge resolution and diff tools.  This will take a number of custom settings, `merge.tool` to tell Git what strategy to use, `mergetool.*.cmd` to specify how to run it, `mergetool.trustExitCode` to tell it if the exit code of that program is indicative of a successful merge resolution or not and `diff.external` to tell it what command to run for diffs.  So, you can either run four config commands:

	$ git config --global merge.tool extMerge
	$ git config --global mergetool.extMerge.cmd 'extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"'
	$ git config --global mergetool.trustExitCode = false
	$ git config --global diff.external extDiff

or you can just edit your `~/.gitconfig` file to add these lines:

	[merge]
	  tool = extMerge
	[mergetool "extMerge"]
	  cmd = extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"
	  trustExitCode = false
	[diff]
	  external = extDiff

After this is all set, if you run diff commands such as:

	$ git diff 32d1776b1^ 32d1776b1
	
Instead of getting the diff output on the command line, Git will fire up Perforce with the diff, which will look something like Figure 7.1

![Figure 7.1](/images/fig0701.png)

Also, if you try to merge two branches and subsequently have merge conflicts, you can run the command `git mergetool` and it will start your Perforce visual tool to let you resolve the conflicts through that GUI tool.

The nice thing about this wrapper setup is that we can change our diff and merge tools pretty easily.  For example, if we want to change our `extDiff` and `extMerge` tools to run the KDiff3 tool instead, all we have to do is edit our `extMerge` file.

	$ cat /usr/local/bin/extMerge
	#!/bin/sh	
	/Applications/kdiff3.app/Contents/MacOS/kdiff3 $*

Now Git will use the KDiff3 tool for diff viewing and merge conflict resolution instead.

Git also comes preset to use a number of other merge resolution tools without having to setup the 'cmd' configuration setting for it.  You can set it to "kdiff3", "opendiff", "tkdiff", "meld", "xxdiff", "emerge", "vimdiff" or "gvimdiff".  If you're not interested in using KDiff3 for diff, just for merge resolution, and the command is in your path then you can just run:

	$ git config --global merge.tool kdiff3

If you just run this instead of setting up the `extMerge` and `extDiff` files, then Git will use kdiff3 for merge resolution and the normal Git diff tool for diffs.

### Formatting and Whitespace ###

Formatting and whitespace issues are some of the more frustrating and subtle issues that many developers encounter when collaborating, especially cross-platform. It is very easy for patches or other collaborated work to introduce subtle whitespace changes because of editors that silently introduce it, or Windows programmers to introduce carriage returns at the end of lines they touch in cross-platform projects.  Git has a few configuration options to help with these issues.

#### core.autocrlf ####

If you're programming on Windows, or using another system but working with people that are programming on Windows, you will probably run into line ending issues at some point.  Git can handle this by auto-converting CRLF line endings into LF when you commit, and vice-versa when it checks code out onto your filesystem.  You can turn on this functionality with the `core.autocrlf` setting. If you are on a Windows machine, set it to 'true' - this will convert LF endings into CRLF when you checkout code.

	$ git config --global core.autocrlf true

If you are on a Linux or Mac system that uses LF line endings then you don't want it to convert when you check files out, however if something accidentally gets introduced then you may want Git to fix it.  You can tell Git to convert CRLF to LF on commit but not the other way around by setting `core.autocrlf` to 'input'.

	$ git config --global core.autocrlf input

This setup should leave you with CRLF endings in the Windows checkouts but LF endings on Mac and Linux systems and in the repository itself.

If you are a Windows programmer doing a Windows only project, then you can turn this functionality off altogether, recording the carriage returns in the repository itself by simply setting the config value to 'false'.

	$ git config --global core.autocrlf false

#### core.whitespace ####

Git comes preset to detect and can even fix some whitespace issues.  There are four main whitespace issues that Git can look for, two are enabled by default and can be turned off while two are not enabled by default but can be activated.

The two that are turned on by default are `trailing-space` which looks for spaces at the end of lines, and `space-before-tab` which looks for spaces before tabs at the beginning of a line.

The two that are disabled by default but can be turned on are `indent-with-non-tab`, which looks for lines that begin with 8 or more spaces instead of tabs, and `cr-at-eol` which tells Git that carriage returns at the end of lines are OK.

You can tell Git which of these you want enabled by setting the `core.whitespace` setting to the values you want on or off, seperated by commas.  You can disable settings by either leaving them out of the setting string or by prepending a `-` in front of the value. For example, if you wanted all but cr-at-eol to be set, you could do this:

	$ git config --global core.whitespace trailing-space,space-before-tab,indent-with-non-tab

Now Git will detect these issues when you run a `git diff` command and try to color them for you to see so you can possibly fix them before you commit.  It will also use these values to help you when you apply patches with `git apply`.  When applying patches, you can ask it to warn you if it's applying patches with the specified whitespace issues:

	$ git apply --whitespace=warn <patch>

Or you can have Git try to automatically fix the issue before applying the patch:

    $ git apply --whitespace=fix <patch>

These options apply to the `git rebase` option as well.  If you've committed whitespace issues but have not pushed upstream yet, you can run a rebase with the `--whitespace=fix` option to have it automatically fix whitespace issues as it's rewriting the patches.

### Server Configuration ###

There are not nearly as many configuration options for the server side of Git stuff, but there are a few interesting ones you may want to take note of.

#### receive.fsckObjects ####

By default, Git will not check all the objects it receives during a push for consistency.  Though Git can check to make sure that each object it has matches it's SHA-1 checksum still and points to valid objects and everything, it does not do that by default on every push.  This is a relatively expensive operation and may add a good amount of time to each push, depending on the size of the repo or the push.  If you want Git to check object consistency on every push, you can force Git to do that by setting the `receive.fsckObjects` setting to 'true'
	
	$ git config --system receive.fsckObjects true

Now Git will check the integrity of your repository before each push is accepted to make sure faulty clients are not introducing corrupt data.

#### receive.denyNonFastForwards ####

If you rebase commits that you have already pushed and then try to push again, or otherwise try to push a commit to a remote branch that does not contain the commit that the remote branch currently points to, you will be denied.  This is generally good policy, but in the case of the rebase you may determine that you know what you're doing and can 'force update' the remote branch with a `-f` flag to your push command.

If you want to disable the ability to force update remote branches to non fast-forward references, you can set the `receive.denyNonFastForward` setting.

	$ git config --system receive.denyNonFastForwards true
	
The other way you can do this is via server side receive hooks, which we'll cover in a bit.  There you can do more complex things like deny non fast-forwards to only a certain subset of users.

#### receive.denyDeletes ####

One of the workarounds to the `denyNonFastForward` policy is for the user to simply delete the branch and then push it back up with the new reference.  In newer versions of Git (it was introduced in 1.6.1), you can set the `receive.denyDeletes` setting to 'true'. 

	$ git config --system receive.denyDeletes true

This will deny branch and tag deletion over a push across the board - no user will be able to do it.  To remove remote branches, you'll have to remove the ref files from the server manually.  There are also more interesting ways to do this on a per-user basis via ACLs that we'll cover at the end of this chapter.

