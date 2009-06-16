## Hosted Git ##

Finally, if you don't want to go through all of that, there are several options for hosting your Git projects on an external dedicated hosting site.  There are a number of advantages to this - it is generally very quick to setup, easy to start projects on and there is no server maintainance or monitoring involved.  Even if you do setup and run your own server internally, you may still want to use a public hosting site for your open source code - it's generally easier for the open source community to find and help you with.  

These days you actually have a huge number of options for hosting to choose from, each with different advantages and disadvantages.  To see an up to date list of them, check out the GitHosting page on the main Git wiki.

	http://git.or.cz/gitwiki/GitHosting

As I can't cover all of them and since I happen to work at one of them, I will use this section to walk through setting up an account and creating a new project at GitHub, so you get an idea of what is involved.  I will show some basic GitHub usage both because I work there and so am most familiar with it, and because it is by far the largest open source Git hosting site.  It is also one of the very few that offers both public and private hosting options so you can keep your open source and private commercial code in the same place. In fact, we used GitHub to privately collaborate on this very book.

### GitHub ###

GitHub is slightly different than most code hosting sites in the way that it namespaces projects.  Instead of being primarily based on the project, GitHub is user centric.  That means when I host my 'grit' project on GitHub, you will not find it at 'github.com/grit', but instead at 'github.com/schacon/grit'.  There is no canonical version of any project, which allows for a project to move from one user to another seamlessly if the first author abandons their project.  

GitHub is also a commercial company that charges for accounts that maintain private repositories, but anyone can quickly get a free account to host as many open source projects as you want.  We'll quickly go over how that is done.

#### Setting up a user Account ####

The first thing you'll need to do is setup a free user account.  If you visit the "Pricing and Signup" page at `http://github.com/plans` and click on the "Sign Up" button on the Free account, you'll be taken to the signup page (`https://github.com/signup/free`).

![Plans Page](/images/github/4.1.plans.sm.png)

Here we will have to choose a username that is not taken in the system yet, an email address that will be associated with the account and a password.

![Signup Form](/images/github/4.2.signupform.sm.png)

If you have it available, this is a good time to add your public SSH key as well.  We covered how to generate a new key in the 'Small Setups' section earlier. Simply take the contents of the public key of that pair and paste it into the textbox here.  Clicking on the 'explain ssh keys' link there will take you to detailed instructions on how to do so on all major operating systems.

Once you hit the 'signup' button, you'll be taken to your new user dashboard.

![New User](/images/github/4.3.newuser.sm.png)

Now we have our user all setup and can create a new repository.  We'll start by clicking the 'create a new one' link next to 'Your Repositories'.

#### Creating a new Project ####

Once you've hit that link, you'll be taken to the 'New Repository' form.

![New Repository](/images/github/4.4.newrepo.sm.png)

All we really have to do is provide a project name, but we can also add a description.  When that is done, we can hit the 'Create Repository' button.  Now we have a new repository on GitHub.

![New Repo Header](/images/github/4.5.header.sm.png)

Before we push new content up, GitHub will simply show us instructions for how to get a repository there.  There are instructions specific to a brand new project, pushing an existing Git project, or importing from Subversion.

![New Repo Instructions](/images/github/4.6.instructions.sm.png)

These instructions are similar to what we've already gone over, basically to initialize a project if it is not already a Git project with:

	$ git init
	$ git add .
	$ git commit -m 'initial commit'
	
Then once you have a Git repository locally, add GitHub as a remote and push your master branch up.

	$ git remote add origin git@github.com:testinguser/iphone_project.git
	$ git push origin master

Now your project is hosted on GitHub and you can give the URL to anyone you want to share your project with, in this case `http://github.com/testinguser/iphone_project`.  We can also see from the header on each of our projects pages that we have two Git URLs.

![Project Header](/images/github/4.7.headershort.sm.png)

The 'Public Clone URL' is a public, read only Git URL that anyone can clone the project over.  Feel free to give that out and post it on your website or what have you.

The 'Your Clone URL' is a read/write SSH based URL that you can only read or write over if you connect with the SSH private key that is associated with the public key that you uploaded for your user.  When other users visit this project page, they will not see that URL, only the public one.

#### Importing from Subversion ####

If you have an existing public Subversion project that you want to import into Git, GitHub can often do that for you.  At the bottom of the instructions page there is a link to a Subversion import.  If you click on that, you'll see a form with some information on the import process and a textbox where you can paste in the URL of your public Subversion project.

![SVN Import](/images/github/4.8.svnimport.sm.png)

If your project is very large, nonstandard or private, this process will likely not work for you.  In Chapter 7 of this book, we'll learn how to do more complicated manual project imports.  

#### Adding Collaborators ####

Now let's add the rest of the team.  If John, Josie and Jessica all sign up for accounts on GitHub and we want to give them all push access to our repository, we can add them to our project as 'Collaborators'.  This will allow pushes from their public keys to work as well.

If you click on the 'edit' button in the project header, or the 'Admin' tab at the top of the project, you'll get to the 'Admin' page of your GitHub project which looks something like this:

![Admin Page](/images/github/4.9.collab.sm.png)

To give another user write access to your project, click the 'Add another collaborator' link.  You will get a new textbox that you can type a username into.  As you type, a helper will pop up showing you possible username matches.  When you find the correct user, click the 'Add' button to add that user as a collaborator on your project.

![Collaborator Add](/images/github/4.10.collabadd.png)

When you are finished adding collaborators, you should be able to see a list of them in the 'Repository Collaborators' box. 

![Collaborators](/images/github/4.11.collabs.png)

dwp: Would that image look a bit better with more than one? {: class=note}

If you need to revoke access to individuals, you can simply click the 'revoke' link and their push access will be removed. For future projects, you can also copy collaborator groups by copying the permissions of an existing project.

#### Your Project ####

Once you push your project up, or have it imported from Subversion, then you will have a main project page that looks something like this:

![Main Page](/images/github/4.12.main.sm.png)

When people visit your project, they will see this page.  It contains tabs to different aspects of your projects.  The 'Commits' tab will show you a list of commits in reverse chronological order similar to the output of the `git log` command.  The 'Network' tab will show all the people that have forked your project and contributed back.  The 'Downloads' tab will allow you to upload binaries of your project and have links to tarballs and zipped versions of any of your tagged points in your project.  The 'Wiki' tab provides your project with a wiki for you to write documentation or other information about your project on.  The 'Graphs' tab has some contribution visualizations and statistics about your project.  The main 'Source' tab that you land on will show your projects main directory listing and will automatically render the README file below it if you have one.  It will also show a box above that with the latest commit information.

#### Forking Projects ####

If you want to contribute to an existing project that you do not have push access to, GitHub encourages 'forking' the project.  If you land on someone's project page that looks interesting and you feel that you want to hack on it a bit, you can click the 'fork' button in the project header to have GitHub copy that project to your user so you can push to it.

This way projects don't have to worry about adding users as collaborators to give them push access.  People can fork a project, push to it and the main project maintainer can pull those changes in by adding them as a remote and merging their work in.

To fork a project, simply visit the project page (in this case, 'mojombo/chronic') and click the 'fork' button in the header.

![Forking a Project](/images/github/4.13.fork.sm.png)

After a few seconds, you will be taken to your new project page, which will display that this project is a fork of another one.

![My Fork](/images/github/4.14.myfork.png)

#### GitHub Summary ####

So that's all we'll cover about GitHub, but it's important to note how quickly this all can get done.  You can create an account, add a new project and push to it in a matter of minutes.  If your project is open source, you will also get a huge community of developers that now have visibility into your project and may well fork it and help contribute to it.  At the very least, it may be a way to get up and running and trying out Git quickly.
