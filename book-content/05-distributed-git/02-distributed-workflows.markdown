## Distributed Workflows ##

Unlike centralized version control systems, the distributed nature of Git allows us to be far more flexible in how developers collaborate on projects.  In centralized systems, every developer is a node working more or less equally on a central hub.  In Git however, every developer is potentially both a node and a hub - that is, every developer can both contribute code to other repositories and maintain a public repository that others can base their work off of and contribute to.  This opens up a vast range of workflow possibilities for your project and/or your team, so we'll cover a few common paradigms that take advantage of this flexibility.  We will go over the strengths and possible weaknesses of each design, but you can choose a single one to utilize or mix and match features from each.

### Centralized Workflow ###

In centralized systems, there is generally a single collaboration model - the centralized workflow.  There is one central hub, or repository, that can accept code and that everyone synchronizes their work to.  There are then a number of developers that are nodes, consumers of that hub, that synchronize to that one place.

![Centralized Workflow](/images/ch5/workflow-star.png)

This means that if two developers clone from the hub, both make changes, then the first one to push their changes back up will do so with no problems.  The second one will have to merge the first one's work in before they can push changes up so as not to overwrite the first user's changes.  This concept is equally true in Git as in Subversion (or any centralized VCS), and this model works perfectly well in Git.

If you have a small team or are already comfortable with a centralized workflow in your company or team, you can continue using that workflow with Git very easily.  Simply setup a single repository and give everyone on your team push access and Git will not allow users to overwrite each other - if another developer has pushed since the last time you fetched and merged, it will tell the next developer that tries to push that they are not pushing an upstream commit and will deny them the ability to push until they fetch and merge the newest changes.

sop: Perhaps we should clarify the exact error message in the last sentence.  Its awkward to read "not pushing an upstream commit" and to be honest I don't know what this means.   {: class=note}

This workflow is pretty attractive to a lot of people because it is a paradigm that many are familiar and comfortable with.

### Integration Manager Workflow ###

Since Git allows you to have multiple remote repositories, it is possible to have a workflow where each developer has write access to their own public repository and read access to everyone elses.  In this scenario there is often a canonical repository that represents the 'official' project.  To contribute to that project, you create your own public clone of the project and push your changes to it.  Then you can send a request to the maintainer of the main project to pull your changes in.  They can then add your repository as a remote, test your changes locally, merge them into their branch and push back to their repository.

* Project maintainer pushes to their public repository
* A contributor clones that repository and makes changes 
* The contributor pushes to their own public copy
* The contributor sends the maintainer an email asking them to pull changes
* The maintainer adds the contributor's repo as a remote and merges locally
* The maintainer pushes merged changes to the main repository

![Integration Manager Workflow](/images/ch5/workflow-integration.png)

This is a very common workflow with sites like GitHub, where it is very easy to fork a project and push your changes into your fork for everyone to see.  One of the main advantages of this is that you can continue to work and the maintainer of the main repository can pull your changes in at any time. Contributors don't have to wait for the project to incorporate their changes - it allows each party to work at their own pace.


### Dictator and Lieutenants Workflow ###

This is a variant of the multiple repository workflows. It is generally used by huge projects with hundreds of collaborators; one famous example is the Linux kernel. In this case, there are various integration managers in charge of certain parts of the repository, those are called lieutenants and all the lieutenants have one integration manager or "benevolent dictator". In this case, the benevolent dictator’s repository serves as the reference repository all the collaborators needs to pull from. In other words:

* Regular developers work on their topic branch and rebase their work on top of master
* The master branch is the one of the dictator
* Lieutenants merge the topic branches of the developers into their master branch
* The dictator merges the master branch of the lieutenants into his or her master branch
* The dictator pushes his master to the reference repository so that the other developers can rebase on it

![Dictator Workflow](/images/ch5/workflow-dictator.png)

sop: lieutenants also pull from blessed repository, please add these edges.  {: class=note}

This kind of workflow is not very common but can be useful in very big projects or in highly hierarchical environments as it allows the project leader (ie the dictator) to delegate much of the work and collects large subsets of code at multiple points before integrating them.

These are some commonly used workflows that are possible with a distributed system like Git, but you can see how there can be many variations of these to suit your particular real world workflow.  Now that you can hopefully see which particular workflow combination might work for you, we'll cover some more specific examples of how to accomplish the main roles that comprise the different flows. 