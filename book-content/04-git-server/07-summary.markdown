## Summary ##

There are several options to get a remote Git repository up and running so that you can collaborate with others or share your work.  

Running your own server gives you a lot of control and allows you to run the server within your own firewall, but it is generally pretty difficult and possibly expensive to setup and maintain.  

dwp: I don't think you should say it's difficult and expensive to maintain a git server! (Though I agree with you that the easiest way is to use a hosted one...) It's certainly more hassle to roll your own. {: class=note}

Running a server on AWS can be a quick way to get started, but it's also a bit expensive to keep running all the time and you'll have to back it up or move the data to Amazon block storage, or else any time the server goes down you'll lose your whole setup.

Finally, putting your data on a hosted server is easy to setup and cheap to maintain, even for private data, however you have to be able to keep your code on someone else's servers, and some organizations do not allow that.

dwp: It also gives you an off-site backup, of course. {: class=note}

Hopefully it should be fairly straightforward which solution or combination of solutions is appropriate for you and your organization.