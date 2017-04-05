---
author: admin
comments: true
date: 2014-01-22 17:51:45+00:00
layout: post
slug: puppet-ii-client-installation
title: Puppet II - Client Installation
wordpress_id: 547
categories:
- Puppet
tags:
- Linux
- Puppet
- Systems Administration
- UNIX
---

In our last installment "[Puppet I - Server Installation](http://questy.org/2014/01/puppet-i-installation/)", we covered all the basic necessities of installing Puppet Open Source server.  Of course, a server in and of itself is no good unless we have a client on another host to connect to it.  This article will similarly lead you step-by-step through the process of installing the client.

_**Puppet Client**_

Installation of our client is not so very much different than the initial server install.  Both come from the same Yum repository, and both need the same support libraries and files.

As expected, a client will need the OS installed (again, we're working on the assumption of RedHat Enterprise Linux, but CentOS, SecurityLinux, etc. should do fine.

Once your operating system is completely installed, just as before we'll need to follow the same steps of getting RedHat registered, the RHEL Yum repo enabled, the Puppet Labs repo configured, the correct support repos added, and finally, the client installed.

_**Enabling the RHEL Yum Repo**_

Just as we did before, we register our RedHat installation via "subscription manager" from RedHat.


_**sudo subscription-manager register**_
_** sudo subscription-manager auto-attach**_
_** sudo subscription-manager attach --auto**_
_** sudo subscription-manager refresh**_


As I mentioned before, RedHat's tool isn't 100% effective 100% of the time, and some level of redundancy in the commands above ensures all the appropriate bits get twiddled.

_**Installing the Puppet Labs Software Repository**_

Again, as before, we simply use RPM to get the repository installed (the below command is all on one line):


**_sudo rpm -ivh https://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-10.noarch.rpm_**


_**Enabling Additional Packages**_

Once again, the "optional" RedHat packages and the "development" PupeptLabs repositories have to be enabled.  You can either use the Yum Config Manager or edit the repo files directly, your choice.  Yum Configuration Manager commands are as follows:


_**sudo yum -y install yum-utils
sudo yum-config-manager --enable rhel-6-server-optional-rpms**_
_** sudo yum-config-manager --enable puppetlabs-devel**_


You should note that the "enable" command above actually has two hyphens in front of it instead of one long one.  WordPress concatenates those.

_**Get Current!**_

Now is the time to get RHEL completely up to date through the usual means:


_**yum -y update**_


_**Install the Client**_


**_yum -y install puppet_**


If all goes well, this will complete the client config on your client host and you are now ready to configure the server, configure the client, and then connect them via SSL as provided by Puppet Labs.

In our next edition, we'll be doing just that.
