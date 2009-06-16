# Git Internals #

You may have skipped to this chapter from a previous chapter or you may have gotten here after reading the rest of the book - in either case, this is where we will go over the inner workings and implementation of Git.  I found learning this information as being fundamentally important to really understanding how useful and powerful Git is, but others have argued to me that it can be confusing and unneccesarily complex for beginners.  Thus, I have placed it as the last chapter in this book so that you can read it early or later in your learning process.  I leave it up to you to decide.

Now that you are here however, let's get started.  First of all, if it is not yet clear, Git is fundamentally a content addressable filesystem which then had a version control system user interface written on top of it. We'll cover what this means in depth in a bit.

In the early days of Git (mostly pre 1.5), the user interface was much more complex because it emphasized more of this filesystem rather than a polished VCS.  In the last few years the UI has become polished and refined to be as clean and easy to use as any other system out there, but often the stereotype of the Git UI in it's early days being difficult to learn and complex still propogates.

However, this content addressable filesystem layer is pretty amazingly cool as well, so we'll cover that first in this chapter, then we'll cover the transport mechanisms and lastly the repository maintainance tasks that you may eventually have to deal with.

## Plumbing and Porcelain ##

DWP: When you say 'will cover' below, that seems a bit odd for the last chapter! {: class=note}

In this book, we will cover how to use Git with about 40 or so verbs, such as 'checkout', 'branch', 'remote' and so on.  However, Git was actually initially a toolkit for an SCM rather than a full user friendly SCM, so it has a bunch of verbs that do real low-level work and were designed to be chained together UNIX style or called from scripts.  These commands are generally referred to as 'plumbing' commands, while the more user friendly commands are called 'porcelain' commands.

DWP: I have forgotten what SCM stands for. I guess I might not be alone! {: class=note}

In the rest of this book, we deal almost exclusively with 'porcelain' commands.  However, in this chapter we will be dealing mostly with the lower level 'plumbing' commands as they give us access to the underlying inner workings of Git and can help demonstrate how and why Git does what it does.  These commands are not meant to be used manually on the command line, but to be used as building blocks for new tools and custom scripts.

## The Git Directory ##

First of all, when we run `git init` in a new or existing directory, Git will create the `.git` directory, which is where almost everything that Git stores and manipulates is located.  Simply copying this single directory elsewhere will give you nearly everything you need.  So, this whole chapter will basically be dealing with the stuff in this directory.  Let's take a quick look at it.

DWP: Copying the directory elsewhere will give you everything you need for what? {: class=note}

	$ ls 
	HEAD
	branches/
	config
	description
	hooks/
	index
	info/
	objects/
	refs/

You may see some other files in there, but this is a fresh `git init`ed repository - it is what you will see by default. First of all, the 'branches' directory is not used by newer Git versions and the 'description' file is only used by the GitWeb program, so don't worry about those.  The 'config' file is where our project specific configuration options are and the 'info' directory keeps a global 'exclude' file for ignored patterns that you don't want to track in a `.gitignore` file.  The 'hooks' directory contains your client or server side hook scripts.

DWP: Perhaps point to the place where you discuss hooks. {: class=note}

This leaves 4 important entries - the 'HEAD' and 'index' files and the 'objects' and 'refs' directories.  These four things are the core parts of Git.  The 'objects' directory stores all the content for your database, the 'refs' directory stores pointers into commit objects in that data ('branches'), the HEAD file points to the branch that you currently have checked out and finally the 'index' file is where Git stores your staging area information.  We're now going to look at each of these sections in detail to see how Git actually operates.








