---
layout: post
title: "Scaling Puppet Enterprise - Part III - Additional Compilers"
date: 2017-04-21 09:32:47 -0400
comments: true
categories: puppet, geekstuff
---

###First, Some Philosophy

The Puppet Enterprise documentation circa PE 2015.3.2 had some "issues". Let me actually preface that, though. Puppet Labs' documentation is by far some of the most voluminous and in many respects most complete vendor documentation out there. I don't mean to disparage their work AT ALL. When it comes to the fact they even have documentation at this level, they're the "bees knees". However, I've always written documentation to fit the "grandma rule". If my grandma couldn't read the documentation and follow a step-by-step process to install Puppet successfully, then its just too complex and needs to be simplified.

This causes a problem, of course. There are technologists out there that would become annoyed at repetition, verbosity around "understood" things, and spelling out each and every step along the way... even painfully. However, I feel it is the only _proper_ way to document something. My rules are simple.

	1. Leave nothing to question
	2. Be as verbose and clear as possible
	3. Make sure everything is in order, step-by-step

By following this simple guideline, I feel I'm doing more of a service to the reader than if I presumed on their level of sophistication with Puppet, Linux/UNIX, Windows, research capability, Google-foo or whatever.

So let's dive in, shall we?

---

***You should have completed a split install before beginning this section. You can find the Split Installation documentation at Puppet's Website, or the first installment of this tutorial [here](http://questy.org/blog/2017/04/18/scaling-puppet-enterprise-part-ii-installation/).***

##HAProxy

Seemingly counterintuitive, I want to next install the HAProxy we will use as a Load Balancer on the additional compilers.  By installing this first, we can utilize Puppet to install the HAProxy, and manage them automatically rather than doing a lot of ad-hoc work.

Also, by doing the proxy first,  the prerequisites are satisfied in their proper order, the Load Balancer exists before configuring additional compilers (to be able to utilize the dns_alt_names for the load balancer along with the compilers) and to have the GitLab in place and hosting the control_repo before turning on and configuring Code Manager.

###Hardware

In the initial hardware list, I included a node called "Compile Master".  This node  looked like:

![Compile Master Specs](http://cvquesty.github.io/images/compile_master_specs.png)

This node may seem like overkill, but disk and memory are cheap.  If you are scaling at this level, its better to not have to reinstall your Load balancer later. Keep in mind, you don't have to use HAProxy and can use a corporate Load Balancer here, but its configuration is outside the scope of this tutorial.

Once you've provisioned the load balancer, ssh to the node as the root user, and use the "frictionless installer" to add your Puppet agent.

```
curl -k https://master.example.com:8140/packages/current/install.bash | bash
```

When the client is fully installed, retrieve the Enterprise Console from your browser, and navigate to Nodes | Classification | Unsigned Certificates and select "Accept All".  Finally, ssh to the instance as the root user and run **_puppet agent -t_** to finish the setup.

##Configure the Load Balancer

At this point, the node is provisioned and you have a Puppet agent running on it, but you have as of yet not configured the HAProxy Load Balancer for use in the environment. The load balancer will be necessary to have in place prior to adding compile masters to your existing split installation. The following instructions guide you through setting up the HAProxy load balancer.

1. SSH to the Puppet Master as root.  _(master.example.com in our list)_

2. Install the HAPRoxy Forge Module on the master
```
puppet module install puppetlabs-haproxy
```
<br>
	_leave your root console open while performing steps 3-6_

3. Retrieve the Enterprise Console in your browser

4. Select **Nodes** | **Classification**

5. Create a New Classification Group called "**Load Balancer**"

6. Select the new group from the list and pin the node "**compiler.example.com**" into the new group.

7. In your open SSH session to master.example.com, create the profiles module to hold the configuration for HAProxy

```
cd /etc/puppetlabs/code/environments/production/modules

mkdir -p profiles/manifests

cd profiles/manifests
```
8. Once you have changed to the profiles/manifests directory, create the loadbalancer.pp manifest.

9. Follow the documentation [here](https://forge.puppet.com/puppetlabs/haproxy/readme) to configure HAProxy. When complete, the loadbalancer.pp manifest should resemble the following with IPs corrected for your particular instance:


```
# Load Balancer Profile
class profiles::loadbalancer {

  class { 'haproxy': }

  # Main Proxy Listener
  haproxy::listen { 'compiler.example.com':
    collect_exported => false,
    ipaddress        => $::ipaddress,
    ports            => '8140',
  }

  # First Load balanced Compile Master
  haproxy::balancermember { 'compiler1.example.com':
    listening_service => 'compiler.example.com',
    server_names      => 'compiler1.example.com',
    ipaddress         => '10.0.1.24',
    ports             => '8140',
    options           => 'check',
  }

  # Second Load Balanced Compile Master
  haproxy::balancermember { 'compiler2.example.com':
    listening_service => 'compiler.example.com',
    server_names      => 'compiler2.example.com',
    ipaddress         => '10.0.1.25',
    ports             => '8140',
    options           => 'check',
  }
}
```

Once you have created this profile, retrieve the Puppet Enterprise Console in your browser and navigate to **Nodes | Classification | Load Balancer**.

1. Selet the **Classes** tab.
2. Click the "refresh" button so the console will pick up your new loadbalancer.pp profile to classify your node with.
3. Under the "Add new Class" heading, select **profiles::loadbalancer** from the list that drops down.
4. Click "Add Class".
5. Select "Commit 1 Change" at the bottom right of the page.
6. SSH back into **compiler.example.com** and run **puppet agent -t** to configure the Load Balancer.

Your Load Balancer is now prepared to balance traffic to two catalog compilers (*catalog1.example.com and catalog2.example.com*) as listed in the above configuration.

###Notes
---
I noted when putting together the loadbalancer.pp profile above that I had previously used some REALLY ODD ip addresses in the balancer config.  Why? For the life of me I cannot recall. The original file looked like so:

```
# Load Balancer Profile
class profiles::loadbalancer {

  class { 'haproxy': }

  # Main Proxy Listener
  haproxy::listen { 'compiler.example.com':
    collect_exported => false,
    ipaddress        => $::ipaddress,
    ports            => '8140',
  }

  # First Load balanced Compile Master
  haproxy::balancermember { 'compiler1.example.com':
    listening_service => 'compiler.example.com',
    server_names      => 'compiler1.example.com',
    ipaddress         => '10.0.1.24',
    ports             => '8140',
    options           => 'check',
  }

  # Second Load Balanced Compile Master
  haproxy::balancermember { 'compiler2.example.com':
    listening_service => 'compiler.example.com',
    server_names      => 'compiler2.example.com',
    ipaddress         => '10.0.1.25',
    ports             => '8140',
    options           => 'check',
  }
}
```
In my original implementation I set the ipaddres fields with some odd IP addresses. For info around how to fill those but ,the documentation gives some hints:


> ipaddresses: Optional. Specifies the IP address used to contact the balancermember service. Valid options: a string or an array. If you pass an array, it must contain the same number of elements as the array you pass to the server_names parameter. For each pair of entries in the ipaddresses and server_names arrays, Puppet creates server entries in haproxy.cfg targeting each port specified in the ports parameter. Default: the value of the $::ipaddress fact.


Since I was originally setting these up in Digital Ocean, I used the IP space 159.203.x.x which belongs to Digital Ocean. I am guessing these were the hard IPs on the instances I stood up. Since the documentation above states these are optional, you have two options here.  Either leave those lines out of your config altogether, or manually set them to the IP Address of the instance you're using. Try each and do which works for you.

##Conclusion

Your HAProxy Load balancer is now complete and ready to take traffic to the additional catalog compiler nodes. In installment IV, we'll begin to add in more components along the way to a fully developed LEI of Puppet Enterprise.
