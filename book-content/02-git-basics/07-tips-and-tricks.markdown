## Tips and Tricks ##

Before we finish up this chapter on Basic Git, there are a few little tips and tricks that might make your Git experience a bit simpler, easier or more familiar.  Many people use Git without using any of these and I won’t refer to them or assume you have them done later in the book, but you should probably know how to do them.

### Auto-completion

If you use the Bash shell, Git comes with a nice auto-completion script you can enable. If you download the Git source code, look into the “contrib/completion” directory and there should be a file called “git-completion.bash”. You can copy this to your home directory and add this to your .bashrc file:

	source ~/.git-completion.bash

If you want to setup Git to automatically have Bash shell completion for all users, you can copy this script to the `/opt/local/etc/bash\_completion.d` directory on Mac systems or to the `/etc/bash\_completion.d/` directory on Linux systems.  This is a directory of scripts that Bash will automatically load to provide shell completions.

If you’re using windows with Git Bash, which is the default when installing Git on Windows with MSysGit, auto-completion should be pre-configured.

Now just type the tab key when writing a Git command and it should return a set of suggestions for you to pick from:

	$ git co<tab><tab>
	commit config

In this case typing “git co” and then hitting the tab key twice will suggest “commit” and “config”.  Adding “m<tab>” will complete “git commit” automatically.

This also works with options, which is probably more useful.  For instance, if you are running a ‘git log’ command and can’t remember one of the options, you can start typing it and hit ‘tab’ to see what matches.

	$ git log –s<tab>
	--shortstat  --since=  --src-prefix=  --stat   --summary

That’s a pretty nice trick and may save you some time and documentation reading.

### Git Aliases

Git does not infer your command if you type it in partially.  If you don’t want to type out the entire text of each of the Git commands, you can easily setup an alias for each command using ‘git config’.  Here are a couple of examples that you might want to setup.

	$ git config --global alias.co checkout
	$ git config --global alias.br branch
	$ git config --global alias.ci commit
	$ git config --global alias.st status

What this means is that, for example, instead of typing “git commit”, you will just need to type “git ci”. As you go on using git, you will probably use other commands very frequently as well; in this case, don’t hesitate to create a new alias.

This can also be very useful in creating commands that you think should be there.  For example, to correct the usability problem we encountered with unstaging a file, we can simply add our own ‘unstage’ alias to git:

	$ git config --global alias.unstage ‘reset HEAD’

This makes the following two commands equivalent:

	$ git unstage fileA
	$ git reset HEAD fileA

Which seems a bit clearer. Another common one is to add a ‘last’ command like this:

	$ git config --global alias.last 'log -1 HEAD'

This way you can see the last commit easily:

	$ git last
	commit 66938dae3329c7aebe598c2246a8e6af90d04646
	Author: Josh Goebel <dreamer3@gmail.com>
	Date:   Tue Aug 26 19:48:51 2008 +0800

	    test for current head
    
	    Signed-off-by: Scott Chacon <schacon@gmail.com>

As you can tell, Git will simply replace the new command with whatever you alias it for.  However, maybe you want to run an external command, rather than a Git subcommand.  In that case, you simply start the command with a ‘!’ character.  This is very useful if you write your own tools that work with a git repository, but we can demonstrate it with something like aliasing ‘git visual’ to run gitk.

	$ git config --global alias.visual "!gitk"

## Summary

Now you can do all of the basic local Git operations – creating or cloning a repository, making changes, staging and committing those changes and viewing the history of all of the changes that the repository has been through.  Next we will cover how the real killer feature of Git – it’s branching model.
