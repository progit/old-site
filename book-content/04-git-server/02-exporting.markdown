## Getting Git on a Server ##

In order to initially setup any Git served repository, you have to export an existing one into a 'bare' repository - a repository that does not contain a working directory. This is generally pretty straightforward to do. 

In order to clone your repository to create a new bare repository, just run the clone command with the '--bare' option.  By convention, bare repository directories end in '.git', like so:

	$ git clone --bare my_project my_project.git
	Initialized empty Git repository in /opt/projects/my_project.git/

Now, the output for this command is a little confusing. The only part of the clone process that generates status output in this case is the `git init` part, which creates an empty directory.  The actual object transfer gives no output, but it does happen.  You should now have a copy of the Git directory data in your 'my\_project.git' directory.

This is _roughly_ equivalent to something like

	$ cp -Rf my_project/.git my_project.git
	
There are a couple of minor differences in the configuration file, but for our purposes now, this is pretty close to the same thing.  It's just taking the Git repository by itself, without a working directory, and creating a directory specifically for it alone.

### Putting the Bare Repository on a Server ###

Now that we have a 'bare' copy of our repository, all we need to do is put it on a server and setup our protocols.  Let's say that we've setup a server for this that we have SSH access to called 'gitserver.com', and we want to store all our Git repositories under the '/opt/git' directory.  We can setup our repository by just copying our repository over:

	$ scp -r my_project.git user@gitserver.com:/opt/git

At this point, other users that have SSH access to the same server can clone our repository by running:

	$ git clone user@gitserver.com:/opt/git/my_project.git
	
No problem.  Now you see how easy it is to take a Git repository, create a bare version and place it on a server that you and your collaborators have SSH access to.  Now you're ready to collaborate on the same project. 

It's important to note here that this is literally all you need to do to run a useful Git server that several people have access to - just add SSH-able accounts on a server and stick a bare repository somewhere that all those users have read and write access to.  You are now ready to go - nothing else needed.

In the next few sections, we'll see how to expand on that to more sophisticated setups.  This will include not having to create user accounts for each user, adding public read access to repositories, setting up web UIs, the Gitosis tool and more.  However, keep in mind that to just collaborate with a couple of people on a private project all you need is an SSH server and a bare repository.
