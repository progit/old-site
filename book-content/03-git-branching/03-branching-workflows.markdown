## Branching Workflows ##

DWP: Could you add anything interesting to this workflows section if you had it after the remotes stuff? {: class=note}

Now that we have the basics of branching and merging down, what can or should we do with them?  Here we're going to cover some common workflows that this lightweight branching makes possible so you can decide if you would like to incorporate it into your own development cycle.

### Long Running Branches ###

Since merging in Git is a pretty simple 3 way merge, merging from one branch into another multiple times over a long period is generally easy to do.  This means that I can have several branches that are always open that I use for different stages of my development cycle and merge regularly from some of them into others.

Many Git developers will have a workflow that embraces this, such as having only code that is entirely stable in their 'master' branch - possibly only code that has been or will be released.  Then they will have another parallel branch named 'develop' or 'next' that they use to work off of or to test stability - it is not neccesarily always stable, but whenever it gets to a stable state, it can be merged into 'master'.  It is used to pull in topic branches when they are ready to make sure they pass all the tests and don't introduce bugs.  

sop: What is a topic branch? {: class=note}

Now in reality, we're just talking about pointers moving up the line of commits you are making.  The stable branches are farther down the line and the more bleeding edge branches are farther up the history.

![Figure 3.18](/images/br-topic1.png)

But thinking about them as work silos where sets of commits graduate to a more stable silo when they are fully tested is generally an easier way to think about it.

![Figure 3.19](/images/br-topic2.png)

You can keep doing this for several levels of stability.  Some larger projects will also have a 'proposed' or 'pu' (proposed updates) branch that have integrated branches that are possibly not quite ready to go into the 'next' or 'master' branches.  The idea is that you have branches of various levels of stability and when they reach a more stable level, they are merged into the branch above them.

Again, having multiple long running branches is not neccesary, but is often helpful, especially when dealing with very large or complex projects.

### Topic Branches ###

Topic branches, however, are useful in projects of any size.  A 'topic branch' is what you would call a short lived branch that you create and use for a single particular feature or related work.  This is something that you likely have never done with a version control system before because it was generally too expensive to create and merge branches.  However, in Git it is common to create, work on, merge and delete branches several times a day in parallel.

We saw this in the last section with the 'iss53' and 'hotfix' branches that we created.  We did a few commits on them and deleted them directly after merging them into our main branch.  This allows you to context switch quickly and completely - since your work is totally seperated into silos where all the changes in that branch have to do with that topic, it is easier to see what has happened for code review and such.  You can keep the changes there for minutes, days or months, and  merge them in when they are ready, regardless of the order in which they were created or worked on.

Here is an example of doing some work, branching off for an issue, working on it for a bit, branching off of _that_ for another way of handling the same thing, going back to our main branch and working there for a while, then branching off of there to do some work that we're not sure is a good idea.

![Figure 3.20](/images/br-topic3.png)

Now let's say we decide we like the second solution to our issue better (iss91v2) and the 'dumbidea' branch was shown to our coworkers and turns out it is actually genius.  So now we will just throw away the original 'iss91' branch (losing commits C5 and C6) and merge in the other two.  Our history will then look like this:

![Figure 3.21](/images/br-topic4.png)

It is important to remember when doing all of this that all of these branches are completely local.  When you are branching and merging, all of that is being done only in your Git repository, there is no server communication happening at all.

