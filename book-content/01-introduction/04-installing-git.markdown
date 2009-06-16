## Installing Git 

So now let's get into using some Git.  First things first – we're going to have to install it.  There are a number of ways we can get it – the two major ways are to install it from source or to install an existing package for your platform.

### Installing from Source

If you can, it's generally useful to install Git from source, as you'll get the most recent version.  Each version of Git tends to include pretty useful UI enhancements, so getting the latest version will often be the best route if you feel comfortable compiling software from source.  It is also the case that many Linux distributions contain very old packages, so unless you are on a very up to date distro or are using backports, installing from source may be the best bet.

In order to install Git, you'll need to have the following libraries that Git depends on : curl, zlib, openssl, expat and libiconv.  For example, if you were on a system that has 'yum' or 'apt-get', you can one of these:

	$ yum install curl-devel expat-devel gettext-devel \
	  openssl-devel zlib-devel

	$ apt-get install curl-devel expat-devel gettext-devel \
	  openssl-devel zlib-devel
	
Once you have all the necessary dependencies, you can go ahead and grab the latest snapshot from the Git website: 

	http://git-scm.com/download

Once you have those, simply compile and install :

	$ tar -zxf git-1.6.0.5.tar.gz
	$ cd git-1.6.0.5
	$ make prefix=/usr/local all
	$ sudo make prefix=/usr/local install

Once this is done, you can also get Git via Git itself for updates:

	$ git clone git://git.kernel.org/pub/scm/git/git.git

### Installing on Linux

If you want to install Git on Linux via a binary installer, you can generally do so through the basic package management tool that comes with your distribution.  If you are on Fedora, you can use 'yum':

	$ yum install git-core

Or if you are on a Debian based distribution like Ubuntu, try 'apt-get':

	$ apt-get install git-core

### Installing on Mac

There are two easy ways to install Git on a Mac.  The easiest is to use the graphical Git installer that you can download from the Google Code page.

	http://code.google.com/p/git-osx-installer

![Figure 1.7](/images/fig17.png)

The other major way to install Git is via MacPorts (http://www.macports.org). If you have MacPorts installed, simply install Git via:

	$ sudo port install git-core +svn +doc +bash_completion +gitweb

You don't have to add all the extras, but you'll probably want to have the '+svn' there in case you ever have to use Git with Subversion repositories.

### Installing on Windows

Installing Git on Windows is very easy.  The msysGit project has one of the easier installation procedures.  Simply download the installer exe from the Google Code page and run it:

	http://code.google.com/p/msysgit

Once it is installed, you will have both a command line version (including an ssh version that will come in handy later), as well as the standard GUI.
