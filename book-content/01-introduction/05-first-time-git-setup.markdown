## First Time Git Setup 

Now that you have Git on your system, there are a few things you're going to want to do to customize your Git environment.  You should only have to do these things once, and they'll stick around in between upgrades.  You can also change them at any time by just running through the commands again.

Git comes with a tool called git-config that allows the user to get and set configuration variables that can control all aspects of how Git looks and operates. These variables can be stored in three different places.  

The first place Git will look for these values is in an '/etc/gitconfig' file, which contains values for every user on the system and all of their repositories.  If you pass the option '--system' to git-config, it will read and write from this file specifically. The next place it will look is in the '~/.gitconfig' file, which is specific to your user.  You can make Git read and write to this file specifically by passing the '--global' option. Finally, Git will look for configuration values in the 'config' file in the Git Directory (ie: '.git/config') of whatever repository you are currently using.  These values are specific to that single repository. Each level will overwrite values in the previous level, so values in `.git/config` will trump those in `/etc/sysconfig`.

** TODO : What does Windows do? **

### Your identity

The first thing we're going to want to do when we install Git is to set our users name and email address. This is important because each and every git commit will use this information and it will be immutably baked into the commits you pass around.  

	$ git config --global user.name “John Doe”
	$ git config --global user.email johndoe@example.com

Again, you will only need to do this once if you pass the '--global' option, because then it will always use that information in anything that user does with Git.  If you want to override this with a different name or email address for specific projects, than you can just run this without the '--global' option when you are within that project.

### Your editor

Now that your identity is set up, you can configure your default text editor that will be used on commit messages. By default, Git will use your system default editor which is generally Vi or nano. If you want to use a different text editor such as “emacs”, do the following:

	$ git config --global core.editor "emacs"

### Your diff tool

Another useful option you might want to configure is the default diff tool to use in order to resolve merge conflicts. Say you want to use vimdiff:

	$ git config --global merge.tool vimdiff

### Checking your settings ###

Now, if you want to check what all settings you have setup, you can use the `git config --list` command to list out all the settings Git sees right then.

	$ git config --list
	user.name=Scott Chacon
	user.email=schacon@gmail.com
	color.status=auto
	color.branch=auto
	color.interactive=auto
	color.diff=auto
	alias.ci=commit
	alias.lh=!git-lh
	...

You may see keys more than once, that is because it's reading the same key from different files (`/etc/gitconfig` and `~/.gitconfig` for example).  In this case, Git will use the last value for each unique key that it sees.

You can also check what Git thinks a specific keys value is by just typing `git config {key}`.

	$ git config user.name
	Scott Chacon

