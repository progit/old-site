# Introduction #

So you're about to spend several hours of your life reading about Git.  Let's take a minute to explain what we have in store for you.  In this first chapter, we're going to cover Version Control Systems (VCS) and Git basics – no technical stuff, just what Git is, why it came about in a land full of VCSs, what sets it apart and why so many people are using it.  Then we're going to cover how to get it and set it up for the first time if you don't already have it on your system. 

In **Chapter Two** we will go over basic Git usage - how to use Git for the 80% of cases you will most often have to use it for.  After this chapter, you should be able to clone a repository, see what's going on, modify stuff and contribute changes.  If the book spontaneously combusts at this point, you should already be pretty useful wielding Git in the meantime that it takes you to go pick up another copy.

**Chapter Three** is about the branching model in Git, often described as Git's killer feature.  Here you will learn what truly sets Git apart from the pack, and when you are done you may likely feel the need to take a quiet moment and ponder how you lived life before Git branching was a part of it.

**Chapter Four** will cover Git on the server.  This chapter is for those of you who want to setup Git inside your organization or on your own personal server for collaboration.  We will also cover various hosted options if you prefer to let someone else handle that for you.

**Chapter Five** will go over in full detail various Distributed workflows and how to accomplish them with Git.  When you are done with this chapter, you should be able to work expertly with multiple remote repositories, using Git over email and to deftly juggle numerous remote branches and contributed patches.

**Chapter Six** is about advanced Git commands.  Here you will learn things like binary searching to identify bugs, editing history, revision selection in detail and a lot more.  This will round out your knowledge of Git so that you are truly a master.

**Chapter Seven** is about configuring your custom Git environment.  This includes setting up hook scripts to enforce or encourage customized policies and setting environment configuration settings so that you work how you want to.  We will also cover building your own set of scripts to enforce a custom commiting policy.

**Chapter Eight** deals with Git and other VCSs.  This includes using Git in a Subversion world and converting projects from other VCSs to Git.  A lot of organizations still use SVN and are not about to change, but you've learned the incredible power of Git – this chapter shows you how to cope.  Git has top notch SVN tools, where you get all the local power of Git but can still submit to a SVN server.

Now that you know all about Git and can wield it with power and grace, you may want to read **Chapter Nine**, which delves into the murky yet beautiful depths of Git internals.  Here we will show you details about how Git actually stores it's objects, what the object model is, details about the packfile implementation, server protocols and more.  Throughout the book we will refer to sections of this chapter if you feel like diving deep at that point, but if you are like me, you may actually want to read this section first.  We leave that up to you.

## About Version Control ##

So, what is “Version Control” and why do you care?  Version Control is simply a system that records changes to a file or set of files over time so that you can recall specific versions later.  Although in the context of this book we will often be using software source code as the files being version controlled, in reality we can do this with nearly any files on a computer.   

If you are a graphic or web designer and want to keep every version of an image or layout (which you most certainly would want to), a VCS is a very wise thing to use.  It allows you to revert files back to a previous state, revert the entire project back to a previous state, compare changes over time, see who last modified something that might be causing a problem, or who introduced an issue and when, and more.  It also generally means that you can screw things up or lose files and be able to easily recover them.   In addition, you get all of this for very little functional overhead.

### Local Version Control Systems ###

For many, the version control method of choice is simply to copy the files into another directory, maybe a time stamped one if you're clever.  This is very common, because it is so simple, but it is also incredibly error prone.  It is easy to forget which directory you're in and accidentally write to the wrong file or to copy over files you didn't mean to.

To deal with this issue, programmers long ago developed local version control systems, which had a simple database that kept all the changes to files under revision control.

figure:ch1/18333fig0101.png:local version control

One of the more popular was a system called 'rcs', which is still distributed with many computers today.  In fact, even the popular Mac OS X operating system will have 'rcs' on it if you install the Developer Tools.  This tool basically works by keeping patch sets (that is, the differences between files) from one change to another in a special format on disk, so it can recreate what any of the files looked like any point in time by adding up all the patches.

### Centralized Version Control Systems ###

The next major issue that people run into is that they need to collaborate with others, sometimes people that are very far away.  To deal with this problem, Centralized Version Control Systems (CVCS) were developed.  These systems, such as CVS, Subversion or Perforce, have a single server that contains all the versioned files and a number of clients that check files out from that central place.  For many years, this has been the standard for version control.

![Figure 1.2](/images/fig12.png)
_Figure 1.2 - centralized verison control_

There are many advantages to this setup, especially over local VCSs.  For one, everyone knows to a certain degree what everyone else on the project is doing.  Administrators have pretty fine grained control over who can do what and it's far easier to administer than dealing with local databases on every client.  
However, there are some serious downsides to this setup as well.  The most obvious is the single point of failure that the centralized server represents.  If that server goes down for an hour, that is an hour that nobody can collaborate at all, or save versioned changes to anything they're working on.  If the hard disk that the central database was on corrupts and proper backups are not kept, you lose absolutely everything – the entire history of the project except whatever single snapshots people happen to have on their local machines.  Local VCS systems suffer from this same problem – whenever you have the entire history of the project in a single place, you are at risk of losing everything.

### Distributed Version Control Systems ###

This is where Distributed Version Control Systems (DVCS) step in.  In a distributed VCS (such as Git, Mercurial, Bazaar or Darcs), clients do not just check out the latest snapshot of the files, they fully mirror the repository.  That means that if any server dies where these systems were collaborating from, any of the clients repositories can be copied back up to the server to restore it.  Every 'checkout' is really a full backup of all of the data.

![Figure 1.3](/images/fig13.png)
_Figure 1.3 - distributed version control_

Furthermore, many of these systems deal pretty well with having several remote repositories that they can work with, so you can collaborate with different groups of people in different ways within the same project simultaneously.  This allows you to setup several types of workflows that are simply not possible in centralized systems, such as hierarchical models. Git falls into this category of VCS.
