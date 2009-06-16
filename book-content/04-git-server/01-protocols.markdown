# Git on the Server #

At this point you should be able to do most of the day to day tasks you will be using Git for.  However, in order to do any collaboration in Git, you'll need to have a remote Git repository.  Though you technically _can_ push and pull changes to and from individuals repostitories, it is discouraged because you can fairly easily mess up what they are working on if you're not careful.  Furthermore, you want your collaborators to be able to access the repository even if your computer is offline - having a more reliable common repository is often useful. Therefore, the preferred method for collaborating with someone is to setup an intermediate repository that you both have access to and push and pull to and from that. We'll refer to this repository as a 'Git server', but you'll notice that it generally takes a pretty tiny amount of resources to host a Git repo, so it's rare you'll need to use an entire server for it.

Running a git server is pretty simple.  First you'll have to choose which protocols you want your server to communicate with.  The first section of this chapter will cover which protocols are available and what the pros and cons of each are.  The next sections will explain some typical setups using those protocols and how to get your server running with them.  Last, we'll go over a few hosted options, if you don't mind hosting your code on someone elses server and don't want to go through the hassle of setting up and maintaining your own.

If you have no real interest in running your own server, you can skip to the last section to see some options for setting up a hosted account and then move on to the next chapter, where we discuss the various ins and outs of working in a distributed source control environment.

A remote repository is generally going to be what is called a 'bare' repository - this is a Git repository that has no working directory.  Since the repository is only used as a collaboration point - there is no reason to have any snapshot checked out on disk, so it is just the Git data.  In simplest terms, a 'bare' repository is just the contents of the '.git' directory of your project, with nothing else. 

## The Protocols ##

There are four major network protocols that Git can use to transfer data - Local, SSH, Git and HTTP. Here we'll discuss what they are and in what basic circumstances you would want to (or not want to) use them.

It's important to note that with the exception of the HTTP protocols, all of these require Git itself to be installed and working on the server.

### Local Protocol ###

The most basic is the local protocol, in which the remote repository is simply in another directory on disk. This is often used if everyone on your team has access to a shared filesystem such as an NFS mount, or in the less likely case that everyone actually logs into the exact same computer.  The latter would not be ideal, since all your code repository instances would reside on the same computer, making a catastrophic loss much more likely.

If you do have a shared mounted filesystem, then you can clone, push and pull from a 'local' file based repository.  To clone a repo like this or to add one as a remote to an existing project, simply use the path to the repository as the URI.  For example, to clone a local repository, you can run something like this:

	$ git clone /opt/git/project.git

Or you can also do this:

	$ git clone file:///opt/git/project.git

Git will operate slightly differently if you explicitly specify the 'file://' at the beginning of the URL.  If you just specify the path, Git will try to use hardlinks or directly copy the files it needs.  If you specify the 'file://' Git will fire up the processes that it normally uses to transfer data over a network and is generally a lot less efficient.  The main reason you would specify the 'file://' prefix is if you want a clean copy of the repository with extraneous references or objects left out - generally after an import from another version control system or something similar (see Chapter 9 for maintanence tasks).  We'll be using the normal path here, since it's almost always a better idea.

To add a local repository to an existing Git project,  you can run something like this:

	$ git remote add local_proj /opt/git/project.git

Then you can push and pull to that remote just as though it were over a network.

#### The Pros ####

The pros of file based repositories are that they are simple and they make use of existing permissions and network access.  If you already have a shared filesystem that your whole team has access to, setting up a repository is very easy to do.  You simply stick the bare repository copy somewhere everyone has shared access to and set the read/write permissions as you would any other shared directory.  We will discuss how to export a bare repository copy for this in the next section, 'Exporting Git'.

This is also a nice option for quickly grabbing work from someone else's working repository.  If you and a co-worker are working on the same project and they want you to check something out, running a command like `git pull /home/john/project` can often be easier than making them push to a remote server and then have you pull that down.

#### The Cons ####

The cons of this method is that shared access is generally more difficult to setup and have access to from multiple locations than basic network access.  If you want to push from your laptop when you're at home, you have to actually mount the remote disk, which can be difficult and slow compared to a network based access.

It is also important to mention that this is not neccesarily the fastest option if you're using a shared mount of some kind.  A local repository is only fast if you have fast access to the data.  A repository on NFS is often slower than the repository over SSH on the same server, allowing Git to run off local disks on each system.


### The SSH Protocol ###

Probably the most common transport protocol for Git is over SSH.  This is because SSH access to servers is already setup in most places and if not, is pretty easy to do, and also because it's the only network based protocol that you can easily read and write to.  The other two network protocols (http:// and git://) are generally read-only, so even if you have them setup for the unwashed masses, you still need SSH for your own write commands.  It is also an authenticated network protocol and since SSH is pretty ubiquitous, it is generally easy to setup and use.  

To clone a Git repository over SSH, you can specify the 'ssh://' like this:

	$ git clone ssh://user@server:project.git
	
Or you can simply not specify a protocol - Git will assume SSH if you aren't explicit:

	$ git clone user@server:project.git
	
You can also not specify a user, Git will assume the user you are currently logged in as.

#### The Pros ####

The pros to using SSH are many.  For one, you basically have to use it if you want authenticated write access to your repository over a network.  Second, SSH is relatively easy to setup - ssh daemons are commonplace, many network admins have experience with them and many OS distributions are setup with them or have tools to manage them.  Next, access over SSH is secure - all data tranfer is encrypted and authenticated.  Last, like the `git://` and `file://` protocols, `ssh://` is efficient, making the data as compact as possible before transferring it.

#### The Cons ####

The negative aspects of SSH access is that you can't serve anonymous access of your repository over it.  People have to have access to your machine over SSH to access it at all, even in a read only capacity, which does not make SSH access conducive to open source projects.  If you are using it only within your corporate network, perhaps SSH is the only protocol you'll need to deal with.  If you want to allow anonymous read-only access to your projects, you'll have to setup SSH for you to push over, but something else for others to pull over.

### The Git Protocol ###

Next we have the `git://` protocol.  This is a special daemon that comes packaged with Git that will listen on a custom port (9418) that provides a similar service to the SSH protocol, but with absolutely no authentication.  In order for a repository to be served over the `git://` protocol, you have to touch the 'git-export-daemon-ok' file - the daemon will not serve a repository without that file in it - but other than that there is no security.  Either it is available to everyone or it is not. This means that there is generally no pushing over this protocol - you can enable push access, but given the lack of authentication, if you turn on push access, anyone on the internet that finds the URL of your project could push if you enabled it.  Suffice it to say that this is pretty rare.

#### The Pros ####

The advantages to the `git://` protocol is that it is the fastest transfer protocol of them all.  If you are serving a lot of traffic for a public project, or serving a very large project that you don't need user authentication for read access, it's likely that you'll want to setup a Git daemon to serve your project.  It uses the same data transfer mechanism as the SSH protocol, but without the encryption and authentication overhead.

#### The Cons ####

The downside to the `git://` protocol is the lack of authentication.  It's generally undesirable for the Git protocol to be the only access to your project.  Generally you'll pair it with SSH access for the few developers who have push (write) access, and then have everyone else use `git://` for read only access.

It is also probably the most difficult protocol to setup.  It requires running it's own daemon, which is pretty custom - we'll look at setting one up in the 'Large Setups' section of this chapter - it requires xinetd configuration or the like, which is not always a walk in the park.  It also requires firewall access to port 9418, which is not a standard port that corporate firewalls will often allow.  If you are behind a big corporate firewall, it's common that this obscure port will not be let through.

### The HTTP/S Protocol ###

Last we have the HTTP protocol.  The beauty of the HTTP or HTTPS protocols is the simplicity of setting them up.  Basically all you have to do is put the bare Git repository under your HTTP base path, setup a specific post-receive hook and you're done (See Chapter 7 for details on Git hooks). At that point, anyone that can access the web server you put the repo under can also clone your repository. To allow read access to your repository over HTTP, simply do something like this:

	$ cd /var/www/htdocs/
	$ git clone --bare /path/to/git_project gitproject.git
	$ cd gitproject.git
	$ mv hooks/post-update.sample hooks/post-update
	$ chmod a+x hooks/post-update

That's all.  The post-update hook that comes with Git will by default run the appropriate command (`git update-server-info`) to make the http protocol work properly.  This will be run when you push to this repository over SSH, then other people can clone via something like:

	$ git clone http://example.com/gitproject.git

In this particular case I'm using the `/var/www/htdocs` path that is common for Apache setups, but literally any static web server can be used - just put the bare repository in it's path.  The Git data is served as basic static files (See Chapter 9 for more details about exactly how it is served).

It is possible to make Git push over HTTP as well, though that is not as widely used and requires you to setup rather complex WebDAV requirements.  As it is pretty rarely used, we will not be covering it in this book.  If you are interested in using the HTTP-push protocols, you can read about setting up a repository for this at `http://www.kernel.org/pub/software/scm/git/docs/howto/setup-git-server-over-http.txt`.  One nice thing about it is that any WebDAV server can be used, without specific Git features, so it can be used if your web hosting provider supports WebDAV for writing updates to your website.

#### The Pros ####

The upsides to using the HTTP protocol is that it's pretty easy to setup.  Running the handful of commands above once gives you a pretty simple way to give the world read access to your Git repository.  It takes maybe a few minutes to do.  It's also not very resource intensive on your server.  Since it's generally just using a static HTTP server to serve all the data, a normal Apache server can server thousands of files a second on average - it's difficult to overload, even on a small server.

You can also serve your repositories read-only over HTTPS, which means you can encrypt the content transfer, or you can go so far as to make the clients use specific signed SSL certificates.  Generally if you're going to these lengths, it's easier to use SSH public keys, but it may be a better solution in your specific case to have signed SSL certificates or other HTTP based authentication methods for read-only access over HTTPS.

Another nice thing is that HTTP is such a commonly used protocol that corporate firewalls are often already setup to allow traffic through this port.

#### The Cons ####

The downsides to serving your repository over HTTP is that it is relatively inefficient for the client.  It generally takes a lot longer to clone or fetch from and you often have a lot more network overhead and transfer volume over HTTP than any of the other network protocols.  Since it is simply not as intelligent about transferring only the data you need - there is no dynamic work on the part of the server in these transactions - the HTTP protocol is often referred to as a 'dumb' protocol.  For more information on the differences in efficiency of the HTTP protocol versus the other protocols, see Chapter 9.
