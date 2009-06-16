## Git Server on EC2 ##

sop: Do we need to go into detail about using AWS EC2? {: class=note}

dwp: I wonder if this is a candidate for an appendix. It's not really describing git specifically, but it could come in handy for the reader. What do you think? {: class=note}

If you want to test this setup out quickly instead of spending a few hours getting everything setup step by step following the instructions in the previous section, I've setup an Amazon EC2 image for your convenience.  If you're unfamiliar with Amazon's EC2 service, it's a web service that lets you provision a custom server on demand and pay for it by the hour, starting at $0.10 per hour.  

dwp: Pricing correct at time of press, of course... {: class=note}

If you are interested in getting a server with Gitosis, GitWeb and a Git daemon up and running quickly to evaluate, this section will quickly describe how to do so.  You'll need an Amazon AWS account and the EC2 tools installed.  Since getting EC2 itself setup is not the simplest process and this isn't really the right place to go over it in detail, if you don't have an EC2 account yet you can find out all about the service at Amazon's official website here:

	http://aws.amazon.com/ec2/

To setup local tools for an Ubuntu machine, a nice quick start guide is at

	http://tinyurl.com/ec2ubuntu
	
If you use a Mac, the instructions are pretty similar, but there is a Mac specific guide here:

	http://tinyurl.com/ec2mac

### Running a Git Server Instance ###

dwp: What is AMI? {: class=note}

The AMI for the Git server image I built is `ami-eec22587`. Assuming you have the EC2 tools installed properly, all you need to do to spin up your new Git server is to run `ec2-run-instances` with that AMI (I've moved the output fields to new lines to be easier to read).

	$ ec2-run-instances ami-eec22587
	RESERVATION	r-98229cf1	502376316513	default
	INSTANCE	i-41c05628	
			ami-eec22587
			
						
			pending		
			m1.small
			2009-02-23T00:52:14+0000	
			us-east-1a	
			aki-a71cf9ce	
			ari-a51cf9cc
				
Then you'll want to authorize the SSH and GIT ports:

	$ ec2-authorize default -p 22
	$ ec2-authorize default -p 9418

After a couple of seconds, your new server will be running.

	$ ec2-describe-instances 
	RESERVATION	r-c4239dad	502376316513	default
	INSTANCE	i-95c056fc	
			ami-eec22587	
			ec2-75-101-209-100.compute-1.amazonaws.com	
			domU-12-31-39-00-AA-41.compute-1.internal	
			running
			m1.small	
			2009-02-23T01:06:27+0000	
			us-east-1c	
			aki-a71cf9ce	
			ari-a51cf9cc	

You should be able to SSH into your new server now.

	$ ssh -i <your.pem> root@ec2-75-101-209-100.compute-1.amazonaws.com

### Configuring your new Server ###

Now all you have to do is initialize Gitosis with your personal SSH public key and you're ready to go.  First, get your public key on the server.

	# locally
	$ scp ~/.ssh/id_rsa.pub root@ec2-75-101-209-100.compute-1.amazonaws.com:/tmp
	
dwp: Do you need a root password? Or will the reader have that somehow? {: class=note}

Then you can initialize Gitosis.

	# on the server
	$ sudo -H -u git gitosis-init < /tmp/id_rsa.pub
	$ chmod a+x /opt/git/gitosis-admin.git/hooks/post-update
	
Now you can clone the new gitosis-admin repository locally and start your configuring.

	# locally
	$ git clone git@ec2-75-101-209-100.compute-1.amazonaws.com:gitosis-admin.git 
  
There is also some help in the README file that should be in the home directory of the root user when you log in.  You should now have a server running GitWeb and a Git daemon, administered via Gitosis.

### Persistant Data ###

When a server on EC2 goes down for any reason, you lose any data that was added to that disk since it was instantiated. This means that if you set all this up, any reboot or AWS power loss will lose everything you've done.  This image is meant to be used as a learning tool, not a permanent server.  If you really want to keep permanent data on EC2, you'll have to setup an Amazon elastic block storage device, link it to your instance and set it up so you can reboot the server appropriately. You can read about EBS at `http://aws.amazon.com/ebs/`.

	
dwp: It seems a strange idea to pay AWS 10cents an hour to host a server for a bit of a play, don't you think? {: class=note}