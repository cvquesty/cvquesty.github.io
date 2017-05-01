---
layout: post
title: "Scaling Puppet Enterprise - Part VI - Code Manager"
date: 2017-04-28 11:48:13 -0700
comments: true
categories: puppet, geekstuff
---
###Recap

Let's see where we are.

1. We have a Puppet Enterprise Split Installation consisting of a Puppet Master, PuppetDB, and Puppet Enterprise Console.
2. We have a Load Balancer with two compiler nodes behind it.
3. We have an ActiveMQ Hub and an ActiveMQ Spoke and have removed ActiveMQ responsibilites from the Enterprise Master (MoM).
4. We have built a GitLab instance to host our Control Repo and other items necessary to operation of our Puppet environment.

---

The final remaining piece is like a "glue" step where we pull all the various pieces together, generate keys for SSH and deployment tokens. We also associate the ctalog compilers to the Enterprise master and coordinate the deployment of code across the masters.  Needless to say, you will need to have already performed all the preceding steps, and have made everything ready to go for the following procedure. Failure to have done so will have unpredictable results. So, if you're ready, let's proceed.

###Setup A Control Repository

The first and foremost piece is to have a repo whose job is to "control" the processig of modules and custom code, and giving the "map" for deployment into your Enterprise master and catalog compilers. Puppet Labs has a suggested sample one [here](https://github.com/puppetlabs/control-repo) which has quite a number of nice features. However, when I first wrote this tutorial, it was considerably overkill for what I was needing to do, so I opted to create my own **very simple** version of a control repository. Quite a few iterations have occurred since writing these instructions, so I will continue with my instructions.

###The Control Repo

The control repo came about as a collaboration at Puppet Labs between employees, consultants, users, etc. It was originally named something else which escapes me at the moment, but eventually came to be named the "control repo" by virtue of the service it performs. In short, it contains the "map" between what you have in Git or at the Puppet Forge, and the deployment directories on your Puppet masters. The "map" itself is known as the "Puppetfile". This file contains a listing of all the modules you want deployed to the server. The bonus is that for each Git branch you have within the repo, this specifies an "environment" to Puppet.

I won't get into all the conversation around whether you should have 1:1 mapping between Git branches and Puppet Environments and then from Puppet Environments to application tiers... I'll leave that to the Puppet folks. I have always mapped everthing identically all the way through, and will cover that process here.

_NOTE: The understood way of doing this these days is that if you have a new "thing" you want to create, you fork a feature branch, apply it to a few nodes for testing, then merge back into your master or production branch and deploy everywhere. I'd recommend learning this. In my own needs, I had a lot of governed environments (PCI, SOX, ITIL, etc) that needed absolute code separation, and the ability to demonstrate that no code in one environment had a chance of deploying to another.  (principle of environment separation)  As a result, I always opted for 1:1 correlation._

I have created a sample control_repo you can clone from here:  [https://github.com/cvquesty/control_repo.git](https://github.com/cvquesty/control_repo.git)

This is a bare repo with a collection of Forge modules populated into the Puppetfile with a "development" and a "production" branch. This will trigger Puppet to create directories called "development" and "production" in /etc/puppetlabs/code/environments, and will contain the items you instruct it to deploy there from the Puppetfile.

First, clone the repo:

```
git clone https://github.com/cvquesty/control_repo.git
```

to a working directory of your choice on your local node.  Next, change to the directory and view the remote:

```
cd control_repo
git remote -v
```

This should present you with the GitHub location of my sample repo:

> origin	https://github.com/cvquesty/control_repo.git (fetch)
> origin	https://github.com/cvquesty/control_repo.git (push)

This will present you an issue in that you can't push to my repo. What I always tell people to do is to move it to their own Git repo. This is well documented elsewhere, but I'll give you an example process.

While in the control_repo directory, perform the following:

```
git checkout development
git remote rm origin 
```

This now makes sure you're in the development branch, and that the repo is unattached to my GitHub account.  Next, you'll need to create a control_repo in _your_ Git server, and set it as your own remote.  First login to your Git server and create an empty repo to hold the code. Next, in the repo you forked from mine, run the commands to switch repos like so:

```
git remote add origin https://<YOUR_GIT_SERVER>/<YOUR_ID>/control_repo.git

git add .

git commit -a -m 'Initial Commit'

git push origin development

git checkout production

git add .

git commit -a -m 'Initial Commit'

git push origin production
```

Now, you have the repo local to you and pointing to your own Git repository so you can edit and update the control_repo at will.  _(you might note extra steps there.  This is for those unfamiliar with Git.  They may not be necessary, but it gives a good pattern for how to work with repos, and I want to establish good habits early.)_

###Generate SSH Keys

On the Enterprise Master, you will need to create locations and/or set permissions on files and directories used by Code Manager:

```
chown -R pe-puppet:pe-puppet /etc/puppetlabs/code
chown -R pe-puppet:pe-puppet /etc/puppetlabs/code-staging
mkdir -p /etc/puppetlabs/puppetserver/ssh
```

Next, you will need to generate a secret key to use with Code Manager setup.  To create the secret key, perform the following on the Enterprise Master:

```
ssh-keygen -t rsa -b 4096 -C "SSH Deploy Keys"
```

When ssh-keygen asks you what to name the key, I usually give it a name I can remember, name it after the customer I am doing work for, or just name it for the Control Repo itself. In this case, let's answer with the latter:

```
/etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa
```

Now, when configuring Code Manager in the Puppet Enterprise Console, you will use that key.

next, ensure all associated files are owned by the PE user:

```
chown -R pe-puppet:pe-puppet /etc/puppetlabs/puppetserver/ssh
```

###Create RBAC Code Manager User

Next, you'll need a Code Manager user in the Enterprise Console. use the following process for that:

1. Create a new role named "Deploy Environments"
2. Assign this role the following permissions:
	* Add the "**Puppet Environment**" type.
	* Set **Permissions** for this type to "**Deploy Code**"
	* Set the **Object** for this type to **All**.

3. Add the **Tokens** Type
	* Set **Permissions** for this type to **Override Default Expiry.**
4. Create a local user to manage code deployments.
	* Click "**Access Control | users**"
	* On the **Users** page, in the **Full Name** field, type the User's Full Name: (e.g. **CM Admin**)
	* In the **Login** field, type the name **cmadmin**.
	* Click **Add Local User**.
5. Set the User's password to "puppetlabs" (or whatever you'd like to use)
	* Select the user from the list.
	* Click "**Generate password reset**"
	* Retrieve the link in a browser, and set the password to "**puppetlabs**".
6. Finally, add the user to the "**Deploy Code**" role.
	* Click the "**Deploy Environments**" role.
	* Click the "**Member Users**" tab.
	* Fromt the dropdown list in the **User Name** field, select the **CM Admin** user and click **Add User**.

###Code Manager

Under the covers, Puppet Labs now uses r10k with the control repository to manage the deployment of code.nUnder this scenario, a few items are very important to remember:

* Under no circumstances should you be manually editing code in /etc/puppetlabs/code any more. Any attempt to do so will be overwritten by the code manager. ALL deployments to the system must come through your editing the control_repo and pointing to either Forge modules or custom modules you have written to be deployed to your Enterprise Master (and sync'ed to the catalog compilers).
* You **must** have a control repo branch for each environment you wish to represent in your Masters (production, testing, etc.)
* You cannot shorten or live without the fully named "production" environment. Puppet hard-coded this environment name in the product, and shortening the name to "prd", "prod", etc. will not work.
* Code Manager operates with a synchronization subdirectory that lives in /etc/puppetlabs/code-staging. When you're pushing coe via your control_repo, it goes here first, then Code Manager and Code Sync take over, and publish the code to all compile masters at once. Once all masters have the code in code-staging, it gets copied to /etc/puppetlabs/code.

More information on this process can be found at [Puppet's Documentation site](https://docs.puppet.com/pe/2015.3/code_mgr.html).

###Configuring the Git Server

You should have a custom deployment user explicitly for pushing code into your master. I have settled on using "cmadmin" as a deploy user on the Git Server. This allows you to have a generic user on the GitLab server you created earlier that you can work with, configure web hooks for, and then leave the credentials for that user with your customer or place it into IDM for your company.

To setup the new user:

1. Create a user in the admin area of the GitLab server named "**cmadmin**".  Next, select "**Edit**" in the upper right hand corner of the screen and set the password as you see fit. _(I'll use "**puppetlabs**")_
2. Select "Impersonate" from the upper right hand section of the page to assume the identity of the "**cmadmin**" user.
3. Select "**New Project**".
4. On the resulting page, create a new repo called "**control_repo**" and make it a piblic project.
5. Click "**Create Project**".
6. Push the control repo from the previous section to this repo in the **cmadmin** space.
7. Seeing as we are using GitLab, you are unable to use a full authenticated deploy token because GitLab server's input buffer is too short to handle a full authentication token. _NOTE: This has changed in later versions of GitLab. You may find success in just creating the token._
8. Configure the Webhook:

* Connect to your Git server _(e.g. http://git.example.com)_ and choose the "**settings gear**" from the bottom left hand side of the page.
* Once in the settings for the **cmadmin** user, there is a small icon on the left frame tht looks like two links of chain and is labelled "**Webhooks**".
* Next, add the _**https://master.example.com:8170/code-manager/v1/webhook?type=gitlab**_ formatted webhook into the "**URL**" box.

_The "**prefix**" section points to the name od the user based on the way GitLab uses namespaces in the URL._

* Also select items you need from the list of options. I recommend selecting all items **except** "**Build Events**" and **DE**-select "**Enable SSL Verification**".
* Click "**Add Webhook**".

###Configuring the SSH Key

Finally, add the **PUBLIC** SSH key created on the Enterprise master located at /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa.pub to the SSH keys section for the **CM Admin** user in the GitLab Server.

1. While still impersonating the "**cmadmin**" user in the GitLab GUI Interface, choose the "**cmadmin**" icon in the lower left of the browser.  Next, choose "**Profile Settings**" in the left hand bar.
2. Under the profile's Settings, choose "**SSH Keys**" from the left hand bar.
3. Paste in the **PUBLIC KEY** to the "**Key**" text box. The **Title** text box should populate automatically. _(or, you can name it yourself.)_
4. Click "**Add Key**".

###Installing Code Manager

This process assumes you have follwed this entire series from start to here in order. The final steps are to install and configure the Code Manager itself. This is that process.

1. At the **Puppet Enterprise Console**, navigate to **Nodes | Classification | PE Master | Classes Tab | puppet_enterprise::profile::master**.
2. In the **puppet_enterprise::profile::master** class, you need to set the following parameters:

* **r10k_remote** => 'the _git_ FQDN and path to the namespace/control_repo of this node.'

```
e.g. **git@git.example.com:cmadmin/control_repo.git**
```

* **r10k_private_key** => 'the full path to your deploy key on your Puppet enterprise master'

```
e.g. **/etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa**
```

* **file_sync_enabled** => true
* **code_manager_auto_configure** => true

At this point, you also want to make sure your control_repo has a hieradata value set. If you cloned your repo from mine, you already have that value set in the common.yaml in the hieradata directory. That setting would be:

```
puppet_enterprise::master::code_manager::authenticate_webhook: false
```
_NOTE: Recall that GitLab has change dramatically since the original writing of this tutorial. Later versions allow you to authenticate the webhook. WHen I wrote this, I was working around technological limitations that are now gone. Feel free to complete this as needed, but I just wanted to disclaim the reasoning for these previous configuration steps._

Next, ensure the hiera.yaml lives in $confdir as needed for Code manager:

* Edit the /etc/puppetlabs/puppet/puppet.conf file to ensure there is a line in the "**[Main]**" section: **hiera_config = $confdir/hiera.yaml**.

Finally, run the puppet agent to apply all the above configuration changes:

```
puppet agent -t
```

Test the hiera value on the command line to ensure Hiera has picked up your value:

```
hiera -c /etc/puppetlabs/puppet/hiera.yaml puppet_enterprise::master::code_manager::authenticate_webhook environment=production
```

You should get a return of "false".

_NOTE: Later versions of Hiera respond to the "lookup" command. The older "hiera" command line utility has intermittent proper functioning at this time, and it has been recommended on the Puppet Community Slack that "Lookup" is the way to go at this time._

###Generate Authentication Token

On the Puppet Enterprise Master, you must now generate an authentication token for the CM Admin deployment user to be authorized to push code.  First, request the token:

```
/opt/puppetlabs/bin/puppet-access login --service-url https://console.example.com:4433/rbac-api --lifetime 180d
```

It will request a username and password. Use the credentials you created in the RBAC console _(In my example, **cmadmin::puppetlabs**)_ and the system will write the token to **/root/.puppetlabs/token**.

###Time to restart!!

Run the puppet agent on all compile masters in no particular order:

```
**puppet agent -t**
```

###Now, lets' Test!!

Prior to PE 2016.x.x, you could only fire the tests with curl commands against the API. Those would be as follows:


**Deploy a Single Environment**

```
/usr/bin/curl -k -X POST -H 'Content-Type: application/json' "https://localhost:8170/code-manager/v1/deploys? token=\`cat ~/.puppetlabs/token\`" -d '{"environments": ["ENVIRONMENTNAME"], "wait": true}'
```

**Deploy All Code Manager Managed Environments**

```
/usr/bin/curl -k -X POST -H 'Content-Type: application/json' "https://localhost:8170/code-manager/v1/deploys?token=\`cat ~/.puppetlabs/token\`" -d '{"deploy-all": true}'
```

On PE Versions 2016.x.x and later, a new tool known as puppet-code was created to ease the testing and firing of the deploys.

**Deploy a Single Environment**

```
/opt/puppetlabs/bin/puppet-code deploy {environmentname}
```

**Deploy All Code Manager Managed Environments**

```
/opt/puppetlabs/bin/puppet-code deploy --all
```

###Conclusion

At this point, you should see your code beginning to populate the **/etc/puppetlabs/code-staging** directory and then eventually the **/etc/puppetlabs/code** directory.  Your final tests will include pushing code to the control_repo to test that the hook is working properly.

If all goes well, you should have code automatically deploy to the $codedir after a few seconds to a minute depending on a variety of factors.

####Other Stuff

I wrote these as tutorials as I mentioned in the first article to help coworkers complete the same process I was doing. I had to sanitize out a lot of internal info, and I had to change hostnames on the fly to make sure "all the things" were secret that needed to be, so the names in question have not been specifically tested end-to-end, but the principles are the same.

I worked on both 2015.x.x and 2016.x.x with this process, but newer versions of PE may have different features or setup options not covered here.  As with any "Open" documentation, "Your mileage may vary" and "Use at your own risk."

I hope this helps someone out there get Code Manager setup and fuctioning in a Large Environment Installation scenario, and you scale as large as you need to as a result of the footwork I've done here. Feel free to email me for errors you find, and I'll fix 'em up right away!
