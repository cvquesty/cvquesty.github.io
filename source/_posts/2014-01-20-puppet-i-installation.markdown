---
author: admin
comments: false
date: 2014-01-20 19:23:10+00:00
layout: post
slug: puppet-i-installation
title: Puppet I - Server Installation
wordpress_id: 540
categories:
- Puppet
tags:
- Linux
- Puppet
- Systems Administration
- UNIX
---

Ok, so you're interested in this whole Puppet thing, and you want to get a full install in your test environment, or on a set of VMs to work and play with the product.  PuppetLabs has a ton of really cool documentation, and you should certainly avail yourself of it, but I wanted to give you a quick "cookbook" style set of instructions to get you rolling.

**Puppet Master**

There is a lot of documentation surrounding the Puppet Master installation at Puppet Labs, but I find a bit of obfuscation when trying to parse through the huge pile of meat that is there, when all we're looking for is the best of it... you know, the bacon.  :)

**_Puppet Code Repository & Package Source_**

Puppet Labs provides a myriad of repositories and methods by which to install Puppet.  I tend to stick with the main PuppetLabs repository whenever possible.  I also tend to like RedHat derivatives, so I'll be talking about RHEL here, but the same applies to CentOS, Scientific Linux, and other RHEL "children" in the ecosystem.  In short, if you're using something else other than RHEL, you may find yourself hunting packages and repos that contain packages you'll need that the repositories I list here have by default.  As with any web-based documentation, YMMV.

_**Installing the Puppet Master**_

First, you'll need a functioning Installation of RHEL upon which to install and configure Puppet.  I like to include only the "Base Install" and take the "Defaults" when doing a "special purpose" server, as it doesn't muddy the waters and it gives you the cleanest slate possible upon which to build your Puppet platform.

_**Enabling the RHEL Yum Repo**_

As I'm sure you're aware, to use RedHat's repositories, you have to register and have active a subscription with RedHat.  To do a simple connection/attachment to the RedHat repositories, I've found a mixture of success with the Subscription Manager.  However, by performing the following commands (some are redundant), I've been able to get the configuration working with 100% reliability:


_**sudo subscription-manager register**_
_** sudo subscription-manager auto-attach**_
_** sudo subscription-manager attach --auto**_
_** sudo subscription-manager refresh**_


Obviously, this is a RedHat topic, and not a Puppet one, so if you're having issues with this process, contact Redhat for help.  The crux of this, however, is to just register your instance.

_**Installing the Puppet Labs Software Repository**_

Since I'm working with the current 6.10 release of Puppet, the package you need to install can be added by running:


_**sudo rpm -ivh https://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-10.noarch.rpm**_


This will install the current 6.10 release of the Puppet repo from which we can continue with the Puppet Master installation.

_**Enabling Additional Packages**_

Next, we need to have the "optional" set of packages from RedHat and the development packages from Puppet Labs.  There are two ways to do this.  First, you can edit each repo file in /etc/yum.repos.d (red hat.repo and puppet labs.repo respectively) and change the "enable" flag on each repository from "0" to "1", or you can simply run the provided RedHat Yum configuration manager tool to do it for you as follows.


_**sudo yum -y install yum-utils
sudo yum-config-manager --enable rhel-6-server-optional-rpms**_
_** sudo yum-config-manager --enable puppetlabs-devel**_


_** Get Current!**_

Bring your current host completely up to date by querying the repositories and running the yum updater:


**_sudo yum -y update_**


As I'm sure you're aware, depending on the updates released since the current major version of your repositories, the update time can vary from very short to quite some time.  Just be patient and get up to date before proceeding.

_**Install the Puppet Prerequisite Packages**_

This step name is a *bit* of a misnomer in that I am including a ton of packages that aren't necessary for a "base" install of Puppet Server, but they do support later installations you may wish to do (like PuppetDB, Dashboard, Passenger, etc.).  I just wanted to give a complete list so there's no after-installation required just to get moving with the next component.  Simply install the following via Yum:


_**sudo yum -y install gcc libcurl-devel gcc-c++ openssl openssl-devel ruby-devel httpd httpd-devel  mod_ssl policycoreutils-python apr-devel apr-utils-devel**_


_**Prep Your Firewall**_

You may or may not be using the IPTables Firewall, but you'll want the following rule in place to support the Puppet Master port.  Simply insert the following line into your _/etc/sysconfig/iptables_ file.


_**-A INPUT -m state --state NEW -m tcp -p tcp --dport 8140 -j ACCEPT**_


Note that if you intend to install the Puppet Dashboard as well, you can add that port here too.


**_-A INPUT -m state --state NEW -m tcp -p tcp --dport 3000 -j ACCEPT_**


Make sure you place your firewall rules in the appropriate place in the file.  While outside the scope of this article, IPTables has an order and a format you should follow.  You can find out more [here](http://www.netfilter.org).

Lastly, restart your firewall to pick up the new rules:


_**sudo service iptables reload**_


_**Install Puppet Server**_

After all the above work, if you've had no repository or package errors, your system should be ready to install Puppet Server.  Simply run the following:


_**yum -y install puppet-server**_


_**Conclusion**_

And that should be it.  All packages installed, base RedHat updated with all the latest Ruby & RedHat goodness, and the bad Puppet Server installed and ready for action.

Next time:  Installing the Puppet Client.
