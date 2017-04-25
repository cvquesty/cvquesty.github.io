---
layout: post
title: "Scaling Puppet Enterprise - Part V - GitLab"
date: 2017-04-24 21:48:29 -0700
comments: true
categories: puppet,geekstuff
---

If you've been following for the past 5 installments, we're nearing the end! Note that each of the prior articles required other things to have been completed before reading/performing the contained steps, but this article is a bit different. In all truth, you could do this process at any point, but I placed it here for one reason alone. _"Why do this manually when I could get Puppet to do it for me?"_

The importance of this particular step is that we need a place to hold our "control repo" _(more on this later)_ and if you don't already have Git installed in your environment, you'll need it. So, before finishing up the installation and configuration of Code Manager, utilizing Puppet to install GitLab is a good test that everything is installed and configured properly, and all the components are communicating as expected. 

Without further delay, let's continue.

---

###Create a Machine to Serve as the GitLab Server

Provision a new node according to our earlier chart to serve as your GitLab server. While I list specifications, you may find more mileage by scaling the Git server larger. If you will be expanding your Puppet team and will have dozens to hundreds of people developing for Puppet, scaling will be a consideration. Also, while outside the scope of this article, you will want to configure offsite backup and/or replication to a geographically separte location for your GitLab server. This is of paramount importance. If you lose this server, all configuration for all systems managed in all environments across your organization would be lost. This isn't the end of the world in terms of business continuity, but trying to recreate all that code from the ground up would be prohibitive. 

Yes, people will have recent copies of the repo on their local machines. Yes, with some nonzero level of effort, you should be able to get the repos back.  No, it's not fun, and you'll have a bad time. Just back up your server, and if possible...replicate it elsewhere in your organization.

My intial suggested specifications on this server are:

![GitLab Specs](http://cvquesty.github.io/images/gitlab_specs.png)

I don't specify disk for /opt and /var here, as each of these images carries ample disk with it. If you believe you will need additional storage for your Git instance, feel free to scale this as you see fit. 

Once the server is installed, go ahead and install the Puppet Agent on it, pointing to the compiler VIP like so:

```
curl -k https://compile.example.com:8140/packages/current/install.bash | bash
```

Once the agent installation is complete, in the Puppet Enterprise Console, navigate to **Nodes | Unsigned Certificates** and accept the new cert request for the GitLab server. Once that is complete, SSH to the GitLab server, and run **puppet agent -t** to complete the initial configuration of the node.

###Create a Profile to Manage the GitLab Installation

On the Puppet Enterprise Master, install the **vshn-gitlab** module.

```
puppet module install vshn-gitlab
```

**NOTE: You will need to perform this on ALL catalog compilers in your infrastructure. If the GitLab serer checks in and doesn't find either the vshn-gitlab module or the profile you're creating below on the master the load balancer refers it to, the catalog run will fail.**

On the Puppet Enterprise Master _(eg. master.example.com)_ create a new profile in **$codedir/environments/production/modules/profiles/manifests/gitlab.pp**.

_(Puppet Enterprise has an internal variable for $codedir now. If you have made no modifications to this in the puppet.conf, the default location is /etc/puppetlabs/code.)_

The profile you create should look like the following:

```
# Configure GitLab Server
class profiles::gitlab {

  class { 'gitlab':
    external_url => 'http://git.example.com',
  }
  
}
```

Save this as gitlab.pp.

In the Puppet Enterprise Console, create a new classification group.

* Navigate to **Nodes | Classification**
* Create a group called '**GitLab**' with a parent of '**All Nodes**' in the Production Environment
* Pin the **git.example.com** node into the newly created **GitLab** group.
* Choose the '**Classes**' tab and click the '**Refresh**' icon to pick up your newly created profile.
* Add the **profiles::gitlab** class to the classification group.
* Commit the changes.

###Caveats

Since we're mid-setup and have multiple compilers but do **not** have code sync enabled, we have to manually copy the new profile to all your compilers in the same location. This allows the agent on the GitLab server to pick up the profile regardless of where the load balancer sends the agent request.

Once the profile is in place, run **puppet agent -t** on your GitLab server, and Puppet will then install the GitLab software onto the server. At this point, after a short delay, you should be able to retrieve your GitLab server in a browser _(e.g. http://git.example.com)_ and login with the default credentials.

In our example, **git.example.com** is the server and the login would be automatically set to **admin@example.com** with a password of **5iveL!fe**.  These are defaults set by the GitLab installer.

Your GitLab server should now be up, running, and ready for action in your Puppet Environment.  Look for the final installment to bring everything together and finish the installation.
