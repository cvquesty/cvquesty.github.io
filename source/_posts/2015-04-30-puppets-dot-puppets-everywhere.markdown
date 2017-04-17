---
layout: post
title: "Puppets.. Puppets Everywhere"
date: 2015-04-30 12:52:27 -0400
comments: true
categories: puppet
---
**3.8 is Here!!!**

That’s right, kids. PE 3.8 has dropped, and it is quite tasty. Some highlights:

**AWS Module Now a Supported Module**

As simple as that sounds, it’s huge. Being able to stand up multiple, tens, even hundreds and thousands of servers into AWS at once with Puppet is a great thing, but to have the module supported by Puppet Labs Support is even better.

*Docker Containers??*

Indeed. The Node manager now “gets” Docker containers and you can provision from bare metal as needed. Once the provisioning is done, it hands directly off to Puppet to execute the configuration portion of your run. Seet, sweet sauce right there.

*Bare Metal*

You’ve always been able to foray into the world of bare metal provisioning, but now it too is supported for you. You can stand up OSes, hypervisors, and then hand those off into the config run using Razor. Razor is now core to PE and also supported by the Puppet Labs Support Team.

*Code Management*

A long time coming, you can also manage code deployment to your Puppet Master using r10k, installed by default. Newly dubbed the “Puppet Code Manager”, r10k remains a command line tool, but I hear rumblings there may be some GUI juice on the horizon for this.

**Deprecations**

As with any release, some Puppet Enterprise features are going the way of the Dodo Bird. Some expected, some surprising, Puppet Enterprise’s landscape is certainly changing.

*Cloud Provisioner*

Long decried as a weak part of the PE infrastructure, the newly announced AWS Supported Module renders it redundant, and as such is removed from the shipping product’s default installation. Of course, if you have a large infrastructure that leverages the Cloud Provisioner, you can continue to use it by installing it into PE separately.

*Live Management*

Live management, a long-standing feature of the Enterprise Console, is now also deprecated. Of course, with the new code management features “baked-in” to Puppet Enterprise through r10k, Live Management is somewhat redundant. However, Puppet Labs notes that they will be releasing improved resource management functionality in future releases. If you need Live Management, then just as you can with the Cloud Provisioner, you can turn it on as well in the 3.8.0 product.

**Compatability**

Finally, some older versions of supported OSes are no longer so, and the list is as follows:

centos-5-i386<br>
centos-5-x86\_64<br>
centos-6-i386<br>
debian-6-i386<br>
debian-6-x86\_64<br>
debian-7-i386<br>
debian-7-x86\_64<br>
oracle-5-i386<br>
oracle-5-x86\_64<br>
oracle-6-i386<br>
redhat-5-i386<br>
redhat-5-x86\_64<br>
redhat-6-i386<br>
scientific-5-i386<br>
scientific-6-i386<br>
sles-11-i386<br>
ubuntu-1004-i386<br>
ubuntu-1004-x86\_64<br>
ubuntu-1204-i386<br>
ubuntu-1404-i386<br>

I’m sure you may have some of these in your infrastructure, but they’re usually the result of a vendor application’s supported platforms. If so, you may wish to communicate back upstream to your various vendors, because when you upgrade PE, these go away for you.

**Try It Out**

As usual, I’ve already created a Vagrant instance to allow you to test and work with the new PE, testing out your existing code on the new platform. Check it out on my GitHub here.

Let me know if you find any issues, and happy Puppeting!