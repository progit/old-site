## Git Hooks ##

Like many other version control systems, Git has a way to fire off custom scripts when certain important actions occur.  There are two groups of these hooks, client side and server side.  The client side hooks are for client operations such as committing and merging.  The server hooks are for Git server operations such as receiving pushed commits.  These hooks can be used for all sorts of reasons, we'll cover a few of them here.

### Installing a Hook ###

The hooks are all stored in the `hooks` subdirectory of the Git directory.  In your project, that would be in `.git/hooks`.  By default, Git populates this directory with a bunch of example scripts, many of which are actually pretty useful by themselves, but they also serve as a documentation of the input values of each script.  All the examples are written as shell scripts with some Perl thrown in, but any executable script named properly will work fine - you can write them in Ruby or Python or what have you.  For post 1.6 versions of Git, these example hook files will end with a '.sample', which you will need to rename.  For pre-1.6 verisons of Git, the example files are named properly, but are simply not executable.

To enable a hook script, put a file in the `hooks` subdirectory of your Git directory that is named appropriately and is executable.  From that point forward it should start to be called.  We will cover most of the major hook filenames here.

### Client Side Hooks ###

There are a lot of client side hooks and we'll split them up into email workflow scripts, committing workflow hooks and then the rest of the client side scripts.

#### Committing Workflow Hooks ####

The rest of the client side scripts can be used in just about any workflow.  They are often used to enforce certain policies, though it's important to note that these scripts are not transferred during a clone.  Though you can enforce policy on the server side to reject pushes of commits that don't conform to some policy, it is entirely up to the developer to use these scripts on the client side.  So, these are scripts simply to help the developer and they will have to be setup and maintained by her, though they can be overridden or modified by her at any time.

The first four hooks have to do with the committing process.

The `pre-commit` hook is run first, before you even type in a commit message.  It is used to inspect the snapshot about to be committed to see if you've forgotten something or make sure that tests run or whatever you need to inspect in the code.  If you exit non-zero from this, it will abort the commit, though you can bypass it with `git commit --no-verify`.  You can do things like check for code style (run `lint` or something equivalent), check for trailing whitespace (the default hook for this does exactly that) or check for approprate documentation on new methods.

The `prepare-commit-msg` hook is run before the commit message editor is fired up but after the default message that is put there is created.  It lets you edit the default message before the author sees it.  This takes a few options, the first being the path to the file that holds the commit message so far, the second is the type of commit it is and the third is the commit SHA-1 if this is an amended commit.  This hook is generally not useful for normal commits, but for commits where the default message is auto-generated, say for templated commit messages, merge commits, squashed commits or amended commits.  You might use this in conjunction with a commit template to programatically insert information.

The `commit-msg` hook takes one parameter which again is the path to a temporary file that has the current commit message.  If this script exits non-zero, Git will abort the commit process, so you can use it to validate your project state or commit message before allowing a commit to go through. We'll demonstrate using this to check that your commit message is conformant to a required pattern in the last section of this chapter.

After the entire commit process is completed, the `post-commit` hook is run, which does not take any parameters, but you can easily get the last commit by running `git log -1 HEAD` or something similar.  Generally, this script is used for notification or something similar.

#### Email Workflow Hooks ####

There are three client side hooks that you can setup for an email based workflow.  These are all invoked by the `git am` command, so if you are not using that command in your workflow, you can safely skip to the next section.  If you are taking patches over email prepared by `git format-patch`, then some of these might be helpful to you.

The first hook that is run is `applypatch-msg`.  This script will take a single argument, the name of the temporary file that contains the proposed commit message, and will abort the patch if this script exits non-zero.  You can use this to make sure that a commit message is properly formatted or even to normalize the message somehow by having the script edit it in place.

The next hook to run when applying patches via `git am` is `pre-applypatch`.  This takes no arguments and is run after the patch is applied, so you can use it to inspect the snapshot before making the commit.  You can run tests or otherwise inspect the working tree with this script.  If something is missing or the tests do not pass, exiting non-zero here will also abort the `git am` script without committing the patch.

The last hook to run during a `git am` operation is `post-applypatch`.  This can be used to notify a group or the author of the patch you pulled in that you have done so.  You cannot stop the patching process with this script.

#### Other Client Hooks ####

The `pre-rebase` script runs before you rebase anything and can halt the process by exiting non-zero. You can use this hook to disallow rebasing any commits that have already been pushed.  The example `pre-rebase` script that Git installs will actually do exactly this, though it assumes that 'next' is the name of the branch that you publish.  You will likely need to change that to whatever your stable published branch is.
  
After you run a successful `git checkout`, the `post-checkout` hook will run, which you can use to setup your working directory properly for your project environment.  This may mean moving large binary files in that you don't want source controlled, auto generating documentation or something along those lines.
  
Lastly we have the `post-merge` script, which runs after a successful merge command.  This can be used to restore data in the working tree that Git itself cannot track such as permissions data or can likewise validate the presence of files external to Git control that you may want copied in when the working tree changes.

### Server Side Hooks ###

In addition to the client side hooks, there are a couple of important server side hooks that you can use as a system administrator to enforce nearly any kind of policy for your project.  These scripts run before and after pushes to the server.  The pre hooks can exit non-zero at any time to reject the push as well as print any sort of error message back to the client, so you can setup as complex of a push policy as you wish.

#### pre-receive and post-receive ####

The first script to run when handling a push from a client is the `pre-receive` hook.  It takes a list of references that are being pushed from stdin and if it exits non-zero, none of them will be accepted.  This can be used to check things like making sure none of the updated references are non fast-forwards, checking that the user pushing has create, delete or push access, or even has access to push updates to all of the files they are modifying with the push.

The `post-receive` hook runs after the entire process is completed and can be used to update other services or notify users.  It takes the same stdin data as the `pre-receive` hook.  Examples of using this would be emailing a list, notifying a continuous intergration server, or updating a ticket tracking system which you can even parse the commit messages to see if any tickets need to be opened, modified or closed.  This script cannot stop the push process, but the client will not disconnect until it has completed, so be careful when trying to do anything that might take a really long time.

#### update ####

The `update` script is very similar to the `pre-receive` script, except that they are run once for each branch that the pusher is trying to update.  If the pusher is trying to push to multiple branches, `pre-receive` will only run once, while `update` will run once per branch they are pushing to.  Instead of reading from stdin, this script takes three arguments - the name of the reference (branch), the SHA-1 that reference pointed to before the push and the SHA-1 the user is trying to push. If the `update` script exits non-zero, only that reference will be rejected, other references can still be updated.
