# Git Basics #

If you can only read one chapter to get going with Git, this is it.  This chapter will cover every basic command you need to know to do the vast majority of the things you will eventually spend your time doing with Git.  By the end of it, you should be able to configure and initialize a repository, begin and stop tracking files, and to stage and commit changes.  We will also show you how to setup Git to ignore certain files and file patterns, how to undo mistakes quickly and easily, how to browse the history of your project and view changes between commits properly and how to push and pull from remote repositories easily.

## Getting a Git Repository ##

There are two main cases where you would get a Git project.  The first way would be taking an existing project or directory and importing it into Git.  The second is to clone an an existing repository.

### Initializing a repository in an existing directory

If you are starting to track an existing project in Git, you simply need to go to the projects directory and type:

	$ git init

This will create a new subdirectory there named ‘.git’ that will contain all of your necessary repository files - a Git repository skeleton.  At this point nothing in your project is tracked yet.  See Chapter 9 for more information on exactly what files are contained in the Git directory that was just created.

If there are existing files that you want to start version controlling (as opposed to an empty directory), you will probably want to start tracking those files and do an initial commit.  You can accomplish that with a few `git add` commands specifing the files you want to track, followed by a commit.

	$ git add *.c
	$ git add README
	$ git commit –m ‘initial project version’

We will go over what these commands actually do in just a minute, but at this point you have a Git repository with tracked files and an initial commit.

### Cloning an existing repository

If you want to get a copy of an existing Git repository, say a project you would like to contribute to, the command you will need is ‘git clone’.  For those of you familiar with other VCS systems such as Subversion, you will notice that the command is ‘clone’ and not ‘checkout’.  This is an important distinction – Git receieves a copy of nearly every object that the server has.  Every version of every file for the history of the project is pulled down when you run ‘git clone’.  In fact, if your server disk gets corrupted, you can actually use any of the clones on any client to set the server back up to the state it was in when cloned (you might lose some server-side hooks and such, but all the versioned data would be there – see Chapter 4 for more details.)

So, the way you clone a repository is with ‘git clone [url]’.  For example, if we wanted to clone my Ruby Git library called Grit, you could do so like this:

	$ git clone git://github.com/schacon/grit.git

That will create a directory named ‘grit’, initialize a ‘.git’ directory inside of that, pull down all the data for that repository and checkout a working copy of the latest version.  So, if you go into the new ‘grit’ directory, you will see the project files in there, ready to be worked on or used.  If you wanted to clone it into a directory named something other than ‘grit’, you can specify that as the next command line option:

	$ git clone git://github.com/schacon/grit.git mygrit

That will do the same thing, but the target directory will be called ‘mygrit’ instead.

Git has a number of different transfer protocols you can use.  In the above example we used the ‘git://’ protocol, but you may also see ‘http(s)://’, or ‘user@server:/path.git’ which will use the SSH transfer protocol.  Chapter 4 will introduce all of the available options the server can setup to access your Git repository over and the pros and cons of each. 
