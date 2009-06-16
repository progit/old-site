## Gitosis ##

Keeping all the users public keys in the `authorized_keys` file for access will only work well for a while.  Once you have hundreds of users, it's going to be much more of a pain to manage that process.  You have to shell onto the server each time and there is no access control at all - everyone in that file will have read and write access to every project.

### Gitosis ###

At this point, you may want to turn to a widely used software project called 'Gitosis'.  Gitosis is basically just a set of scripts that help you manage that `authorized_keys` file as well as implement some simple access controls.  The really interesting part is that the UI for this tool for adding people and determining access is not a web interface but a special Git repository.  You setup the information in that project and when you push it, Gitosis reconfigures the server based on that, which is pretty cool.

Now, installing Gitosis is not the simplest task ever, but it's not too difficult.  It's probably easiest to use a Linux server for it - for these examples I'll be using a stock Ubuntu 8.10 server.  

Gitosis requires some Python tools, so first we have to install the Python 'setuptools' package, which Ubuntu provides as 'python-setuptools'.

	$ apt-get install python-setuptools

Next we clone and install Gitosis from the 'eagain.net' server.

	$ git clone git://eagain.net/gitosis.git
	$ cd gitosis
	$ sudo python setup.py install

That will install a couple of executables that will be used by Gitosis.  Next, Gitosis wants to put it's repositories under '/home/git', which is fine but we have already setup our repositories in '/opt/git', so instead of reconfiguring everything, we'll just create a symlink.

	$ ln -s /opt/git /home/git/repositories

Now, Gitosis is going to manage our keys for us, so we'll need to remove the current file, re-add the keys later and let Gitosis control the `authorized_keys` file automatically.  For now, we'll move the `authorized_keys` file out of the way.

	$ mv /home/git/.ssh/authorized_keys /home/git/.ssh/ak.bak

Next we need to turn our shell back on for the 'git' user, if we changed it to the 'git-shell' command.  People still won't be able to login, but Gitosis will control that for us.  So, let's change this line in our `/etc/passwd` file:

	git:x:1000:1000::/home/git:/usr/bin/git-shell

back to this:

	git:x:1000:1000::/home/git:/bin/sh

Now it's time to actually initialize Gitosis.  We do this by running the `gitosis-init` command with our personal public key.  If your public key is not on the server, you'll have to copy it up there.
	
	$ sudo -H -u git gitosis-init < /tmp/id_dsa.pub
	Initialized empty Git repository in /opt/git/gitosis-admin.git/
	Reinitialized existing Git repository in /opt/git/gitosis-admin.git/

This is going to let the user with that key modify the main Git repository that controls the Gitosis setup.  Next we have to manually set the execute bit on the `post-update` script for our new control repository.

	$ sudo chmod 755 /opt/git/gitosis-admin.git/hooks/post-update
	
Now we're ready to roll.  If you're setup correctly, you can try to SSH into your server as the user you added the public key for to initialize Gitosis.  You should see something like this:

	$ ssh git@gitserver
	PTY allocation request failed on channel 0
	fatal: unrecognized command 'gitosis-serve schacon@quaternion'
	  Connection to gitserver closed.

That means that Gitosis recognized you, but shut you out, since you're not trying to do any Git commands.  So, let's do an actual Git command - we'll clone the Gitosis control repository.

	# on your local computer
	$ git clone git@gitserver:gitosis-admin.git

Now we have a directory named 'gitosis-admin' that has two major parts to it.
 
	$ cd gitosis-admin
	$ find .
	./gitosis.conf
	./keydir
	./keydir/scott.pub

The `gitosis.conf` file is the control file that we will use to specify users, repositories and permissions.  The 'keydir' directory will be where we store the public keys of all the users who have any sort of access to our repositories - one file per user.  The name of the file in 'keydir' (scott.pub) there will be different for you - Gitosis takes that name from the description at the end of the public key that was imported with the `gitosis-init` script.

If we look at the `gitosis.conf` file, it should only be specifying information about the 'gitosis-admin' project that you just cloned.

	$ cat gitosis.conf 
	[gitosis]

	[group gitosis-admin]
	writable = gitosis-admin
	members = scott

It shows you that the 'scott' user - the user whose public key you initialized Gitosis with - is the only one that has access to the 'gitosis-admin' project. 

Now let's add a new project for us.  We'll add a new section called 'mobile' where we'll list the developers on our 'mobile' team and projects that those developers need access to.  Since 'scott' is the only user in the system right now, we'll add him as the only 'member' and we'll create a new project called 'iphone\_project' for us to start on.

	[group mobile]
	writable = iphone_project
	members = scott
	
Whenever we make changes to the 'gitosis-admin' project, we have to commit the changes and push them back up to the server in order for them to take effect.

	$ git commit -am 'add iphone_project and mobile group'
	[master]: created 8962da8: "changed name"
	 1 files changed, 4 insertions(+), 0 deletions(-)
	$ git push
	Counting objects: 5, done.
	Compressing objects: 100% (2/2), done.
	Writing objects: 100% (3/3), 272 bytes, done.
	Total 3 (delta 1), reused 0 (delta 0)
	To git@gitserver:/opt/git/gitosis-admin.git
	   fb27aec..8962da8  master -> master

Now we can make our first push to the new 'iphone\_project' project simply by adding our server as a remote to our local version of the project and pushing.  You no longer have to manually create a bare repository for new projects on the server - Gitosis will create them automatically when it sees the first push.

	$ git remote add origin git@gitserver:iphone_project.git
	$ git push origin master
	Initialized empty Git repository in /opt/git/iphone_project.git/
	Counting objects: 3, done.
	Writing objects: 100% (3/3), 230 bytes, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To git@gitserver:iphone_project.git
	 * [new branch]      master -> master

Notice that we don't need to specify the path anymore (in fact, it won't work), just a colon and then the name of the project - Gitosis will find it for us.

Now we want to work on this with our friends, so we'll have to re-add their public keys, but instead of appending them manually to the `~/.ssh/authorized_keys` file on our server, we'll just add them, one key per file, into the `keydir` directory.  How you name the keys will determine how you refer to the users in the 'gitosis.conf' file.  So, let's re-add the public keys for John, Josie and Jessica.

	$ cp /tmp/id_rsa.john.pub keydir/john.pub
	$ cp /tmp/id_rsa.josie.pub keydir/josie.pub
	$ cp /tmp/id_rsa.jessica.pub keydir/jessica.pub
	
Now we can add them all to our 'mobile' team so they will have read and write access to our iphone project.

	[group mobile]
	writable = iphone_project
	members = scott john josie jessica

Now once we commit and push that change, all four of those users will be able to read and write to that project.

Gitosis has simple access controls as well. If we wanted John to have only read access to this project, we can do this instead:

	[group mobile]
	writable = iphone_project
	members = scott josie jessica

	[group mobile_ro]
	readable = iphone_project
	members = john

Now John can clone the project and get updates, but Gitosis will not allow him to push back up to the project.  You can create as many of these groups as you want, each with different users and projects in them.  You can also specify another group as one of the 'members', to inherit all of it's members automatically.

If you have any issues, it may be useful to add `loglevel=DEBUG` under the `[gitosis]` section.  If you've messed up and have lost push access by pushing a messed up configuration, you can manually fix the file on the server under `/home/git/.gitosis.conf` - that's the file Gitosis actually reads it's info from, a push to the project simply takes the `gitosis.conf` file you just pushed up and sticks it there. If you edit it manually that file manually it will remain like that until the next successful push to the 'gitosis-admin' project.
	
### Git Daemon ###

Now for public, unauthenticated read access to our projects, we'll want to move past the HTTP protocol and start using the GIT protocol.  The main reason for this is speed.  The Git protocol is far more efficient and thus faster than the HTTP protocol, so using it will save your users time.

Again, this is for unauthenticated read-only access, so if you are running this on a server outside your firewall, it would only be used on projects that are publicly visible to the world. If the server you are running it on is inside your firewall, you might use it for projects that you want a large number of people or computers (continuous integration or build servers) to have read-only access to where you don't want to have to add an SSH key for each.

In any case, it's relatively easy to setup.  Basically, we simply need to run this command in a daemonized manner:

	git daemon --reuseaddr --base-path=/opt/git/ /opt/git/

The `--reuseaddr` allows the server to restart without waiting for old connections to time out, the `--base-path` option allows people to clone projects without specifying the entire path and the path at the end tells `git daemon` where to look for repositories to export. You'll also need to punch a hole in the firewall on the box you're setting this up on at port 9418 if you're running one.

So, there are a number of ways you can daemonize this process, depending on the operating system you're running.  The way you'll do this on an Ubuntu machine is to use an Upstart script.  So, in the following file:

	/etc/event.d/local-git-daemon

We'll put this script:

	start on startup
	stop on shutdown
	exec /usr/bin/git daemon \
		--user=git --group=git \
		--reuseaddr \
		--base-path=/opt/git/ \
		/opt/git/
	respawn

Now when we restart our machine, our Git daemon will start automatically and respawn if it goes down.  To get it running without having to reboot, you can just run this:

	initctl start local-git-daemon

On other systems, you may instead want to use xinetd or a script in your sysvinit system or something else.  Just so long as you get that command daemonized and watched somehow.

Next we'll have to tell our Gitosis server which repositories to allow unauthenticated Git server based access to. If we add a section for each repository, we can specify which we want our Git daemon to allow reading from.  If we wanted to allow Git protocol access for our iphone project, we would add this to the end of the `gitosis.conf` file:

	[repo iphone_project]
	daemon = yes

When that is committed and pushed up, then your running daemon should start serving requests for that project to anyone that has access to port 9418 on your server.  

If you decide not to use Gitosis, but want to setup a Git daemon, you'll have to run this on each project you want the Git daemon to serve:

	$ cd /path/to/project.git
	$ touch git-daemon-export-ok

The presence of that file tells Git that it's OK to serve this project without authentication.

Gitosis can also control which projects GitWeb will show.  First of all, you need to add something like the following to the `/etc/gitweb.conf` file.

	$projects_list = "/home/git/gitosis/projects.list";
	$projectroot = "/home/git/repositories";
	$export_ok = "git-daemon-export-ok";
	@git_base_url_list = ('git://gitserver');

Then you can control which projects GitWeb will let users browse by adding or removing a 'gitweb' setting in the Gitosis configuration file.  For instance, if we want the iphone project to show up on GitWeb, we make the repo setting look like this:

	[repo iphone_project]
	daemon = yes
	gitweb = yes
	
Now if we commit and push the project, GitWeb will automatically start showing our iphone project.
