## Small Setups ##

If you are a small outfit or are just trying out Git in your organization and only have a few developers, things can be pretty simple for you.  One of the most complicated aspects to setting up a Git server is user management.  If you want some repostiories to be read only to certain users and read/write to others, access and permissions can be a bit difficult to setup.

### SSH Access ###

If you already have a server that all of your developers have SSH access to, it's generally easiest to setup your first repository there, since there is almost no work to do (as we covered in the last section).  If you want more complex ACL type permissions controls on your repositories, you can simply handle it with the normal filesystem permissions of the operating system your server runs.

If you want to setup your repositories on a server that does not have accounts for everyone on your team you want to have write access, then you'll have to setup SSH access. I assume that if you have a server to do this with that you already have an SSH server installed and that's how at least _you_ are accessing it.  

There are a few ways you can give access to everyone on your team.  The first is to setup accounts for everybody, which is pretty straightforward but possibly a bit cumbersome. You may not want to run `adduser` and set temporary passwords for every user.

A second method is to create a single 'git' user on the machine, ask every user that is to have write access to send you an SSH public key and to add that key to the `~/.ssh/authorized_keys` file of your new 'git' user.  At that point, everyone will be able access that machine via the 'git' user.  This does not effect the commit data in any way - the SSH user you connect as does not effect the commits you've recorded at all.

Another way to do it is to have your SSH server authenticate off an LDAP server or some other centralized authentication source that you may already have setup.  As long as each user can get shell access on the machine, any SSH authentication mechanism that you can think of should work.

#### Generating your SSH Public Key ####

That being said, many Git servers authenticate using SSH public keys.  In order to provide a public key, each user in your system will have to generate one if they don't already have them.  This process is pretty similar across all operating systems.  

First you should check to make sure you don't already have a key.  By default, a user's SSH keys are stored in that user's `~/.ssh` directory. You can easily check to see if you have a key already by going to that directory and listing the contents.

	$ cd ~/.ssh
	$ ls
	authorized_keys2	id_dsa		known_hosts
	config		id_dsa.pub

You're looking for a pair of files named `something` and `something.pub`, where the 'something' is usually one of 'id\_dsa' or 'id\_rsa'.  The '.pub' file is your public key and the other is your private key.  If you don't have these files (or you don't even have a '.ssh' directory) you can create them by running a program called `ssh-keygen`, which is provided with the SSH package on Linux/Mac systems and comes with the MSysGit package on Windows.

	$ ssh-keygen 
	Generating public/private rsa key pair.
	Enter file in which to save the key (/Users/schacon/.ssh/id_rsa): 
	Enter passphrase (empty for no passphrase): 
	Enter same passphrase again: 
	Your identification has been saved in /Users/schacon/.ssh/id_rsa.
	Your public key has been saved in /Users/schacon/.ssh/id_rsa.pub.
	The key fingerprint is:
	43:c5:5b:5f:b1:f1:50:43:ad:20:a6:92:6a:1f:9a:3a schacon@agadorlaptop.local

First it will confirm where you want to save the key to (.ssh/id\_rsa), then it will ask you for a passphrase twice, which you can leave empty if you don't want to type a password when you use the key.

Now each user that did this will have to send their public key to you, or whomever is administrating the Git server (assuming you're using an SSH server setup that requires public keys).  All they have to do is copy the contents of the '.pub' file and email it.  The public keys will look something like this:

	$ cat ~/.ssh/id_rsa.pub 
	ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
	GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
	Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
	t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
	mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
	NrRFi9wrf+M7Q== schacon@agadorlaptop.local

For a more in depth tutorial on creating an SSH key on multiple operating systems, see the GitHub Guide on SSH keys at `http://github.com/guides/providing-your-ssh-key`.

#### Setting Up the Server ####

So, let's walk through setting this up on the server side.  In this example we'll use the 'authorized\_hosts' method for authenticating our users.  I'll also assume you are running a standard Linux machine like Ubuntu.  First you'll want to create a 'git' user and create a '.ssh' directory for that user.

	$ sudo adduser git
	$ su git
	$ cd
	$ mkdir .ssh

Next we need to add some developer ssh public keys to the `authorized_keys` file for that user.  Let's assume we've saved a few keys from emails sent to us to temporary files.  Again, the public keys will look something like this:

	$ cat /tmp/id_rsa.john.pub
	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4L
	ojG6rs6hPB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4k
	Yjh6541NYsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9Ez
	Sdfd8AcCIicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myiv
	O7TCUSBdLQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPq
	dAv8JggJICUvax2T9va5 gsg-keypair

So we just append them to our `authorized_keys` file.

	$ cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
	$ cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys
	$ cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys

Now we can setup an empty repository for them by running `git init` with the `--bare` option, which will initialize the repository without a working directory.

	$ cd /opt/git
	$ mkdir project.git
	$ cd project.git
	$ git --bare init                   

Then John, Josie or Jessica can push the first version of their project into that repository by adding it as a remote and pushing a branch up.  Note that someone will have to shell onto the machine and create a bare repository every time you want to add a project.  I'll use 'gitserver' as the hostname of the server you've setup your 'git' user and repository on.  If you're running it internally and setup DNS for 'gitserver' to point to that server, then you can use the commands pretty much as-is.
	
	# on Johns computer
	$ cd myproject
	$ git init
	$ git add .
	$ git commit -m 'initial commit'
	$ git remote add origin git@gitserver:/opt/git/project.git
	$ git push origin master

At that point, the others could clone it down and push changes back up just as easily.

	$ git clone git@gitserver:/opt/git/project.git
	$ vim README
	$ git commit -am 'fix for the README file'
	$ git push origin master


With this method, you can get a read/write Git server up and running for a handful of developers pretty quickly.
	
As an extra precaution, you can easily restrict the 'git' user to only doing Git activities with a limited shell tool that comes with git called `git-shell`.  If you set this as your 'git' user's login shell, then the 'git' user will not be able to have normal shell access to your server.  To use this, simply specify `git-shell` instead of `bash` or `csh` for your user's login shell.  To do this, you'll likely just have to edit your '/etc/passwd' file.

	$ sudo vim /etc/passwd

At the bottom, you should find a line that looks something like this:

	git:x:1000:1000::/home/git:/bin/sh

Then just change the '/bin/sh' to '/usr/bin/git-shell' (or run `which git-shell` to see where it is installed).  The line should now look something like this instead:
	
	git:x:1000:1000::/home/git:/usr/bin/git-shell

Now the 'git' user can only use the SSH connection to push and pull git repos, and cannot actually shell onto the machine.  If you try, you'll see a login rejection like this:

	$ ssh git@gitserver
	fatal: What do you think I am? A shell?
	Connection to gitserver closed.

### Public Access ###

What if you want anonymous read access to your project?  Perhaps instead of hosting an internal private project you want to host an open source one.  Or maybe you have a bunch of automated build or continuous integration servers that change a lot and you don't want to have to generate SSH keys all the time, you just want to add simple anonymous read access.

Probably the simplest way for smaller setups is to just run a static web server with it's document root where your Git repositories are, then enable that post-update hook we mentioned in the first section of this chapter.  Let's work from the previous example.  Say we have our repositories in the `/opt/git` directory and there is an Apache server running on our machine.  Again, you can use any web server for this, but just to show an example I'll demonstrate some basic Apache configurations that should give you an idea of what you might need.

First off we'll need to enable the hook:

	$ cd project.git
	$ mv hooks/post-update.sample hooks/post-update
	$ chmod a+x hooks/post-update

If you're using a version of Git earlier than 1.6, the 'mv' command there will not be necessary - they only started naming the hooks examples with the '.sample' postfix recently.  So what does this `post-update` hook do anyhow?  It looks basically like this:

	$ cat .git/hooks/post-update.sample 
	#!/bin/sh
	exec git-update-server-info

Which means that when you push to the server via SSH, Git will run this command to update the files needed for HTTP fetching.

Next we need to add a VirtualHost entry to our Apache configuration with the document root as the root directory of our Git projects.  Here I'm assuming that you have wildcard DNS setup to send *.gitserver to whatever box you're using to run all of this.

	<VirtualHost *:80>
	    ServerName git.gitserver
	    DocumentRoot /opt/git
		<Directory /opt/git/>
			Order allow, deny
			allow from all
		</Directory>
	</VirtualHost>

Then once we restart Apache, we should be able to clone our repostiories under that directory by just specifying the URL for our project.

	$ git clone http://git.gitserver/project.git

This way, we can setup HTTP based read access to any of our projects for a fair number of users in a few minutes.  Another simple option for public unauthenticated access is to start a Git daemon, though that requires us to daemonize it - we'll cover this option in the next section if you prefer that route.

### GitWeb ###

Now that we have basic read-write and read-only access to our project, you may want to setup a simple web-based visualizer for your projects.  Git comes with a CGI script called GitWeb that is commonly used for this.  You can see GitWeb in use at sites like `git.kernel.org`.

![Figure 4.1 - GitWeb](/images/gitweb.sm.png)

If you want to checkout what it would look like for your project, Git comes with a command to fire up a temporary GitWeb instance if you have a lightweight server on your system like `lighttpd` or `webrick`.  On Linux machines `lighttpd` is often installed, so you may be able to get it to run by just typing `git instaweb` in your project directory.  If you're running a Mac, Leopard comes pre-installed with Ruby so `webrick` might be your best bet.  To start `instaweb` with a non-lighttpd handler, you can run it with the `--httpd` option.

	$ git instaweb --httpd=webrick
	[2009-02-21 10:02:21] INFO  WEBrick 1.3.1
	[2009-02-21 10:02:21] INFO  ruby 1.8.6 (2008-03-03) [universal-darwin9.0]

dwp: I agree. We shouldn't assume ruby will be installed. I don't have ruby either, so that's not a great sample of current readers! Given you've already used apache, why not use it again? {: class=note}

sc: Sorry guys, I want to show this because people often want to see a web ui for thier Git install and I can't get the apache2 handler to work on my mac and Leopard comes with Ruby and is Ruby is otherwise pretty easy to install on nearly any system. It's the only handler I think I've ever gotten to work with instaweb. {: class=note}

That will startup an httpd server on port `1234` and then automatically open a web browser that opens on that page.  Pretty easy on your part.  When you are done and want to shut down the server, you can run the same command with a `--stop` option.

	$ git instaweb --httpd=webrick --stop

If you want to run this on a web server all the time for your team or for an open source project you're hosting, you'll need to setup the CGI script to be served by your normal webserver.  Some Linux distributions have a 'gitweb' package that you might be able to install via `apt` or `yum` - so you may want to try that first. We'll walk though installing it manually very quickly.  First you'll need to get the Git source code, which GitWeb comes with, and generate the custom CGI script:

	$ git clone git://git.kernel.org/pub/scm/git/git.git
	$ cd git/
	$ make GITWEB_PROJECTROOT="/opt/git" \
			prefix=/usr gitweb/gitweb.cgi	
	$ sudo cp -Rf gitweb /var/www/  

Notice that we had to tell it where to find our Git repositories with the `GITWEB_PROJECTROOT` variable.  Now we need to make Apache use CGI for that script, which we can add a VirtualHost for.

	<VirtualHost *:80>
	    ServerName gitserver
	    DocumentRoot /var/www/gitweb
	    <Directory /var/www/gitweb>
	        Options ExecCGI +FollowSymLinks +SymLinksIfOwnerMatch
	        AllowOverride All
	        order allow,deny
	        Allow from all

	        AddHandler cgi-script cgi

	        DirectoryIndex gitweb.cgi

	        RewriteEngine On
	        RewriteCond %{REQUEST_FILENAME} !-f
	        RewriteCond %{REQUEST_FILENAME} !-d
	        RewriteRule ^.* /gitweb.cgi/$0 [L,PT]
	    </Directory>
	</VirtualHost>

Again, this can be served with any CGI capable webserver, so if you prefer using something else it should not be difficult to setup.  The rewrite stuff is also not really neccesary, it just makes the URL look a bit prettier - feel free to remove it if you don't have mod\_rewrite on your server.

At this point, we should be able to visit `http://gitserver/` to view our repositories online, and we can use `git.gitserver` to clone and fetch our repositories over HTTP as well.

We'll probably also want to set the 'group' of the `/opt/git` directories to 'www-data' so our GitWeb script can read access the repositories, since the Apache instance running the CGI script will (by default) be running as that user.

	$ chgrp -R www-data /opt/git
