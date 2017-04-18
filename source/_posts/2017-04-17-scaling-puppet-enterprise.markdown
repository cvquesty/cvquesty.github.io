---
layout: post
title: "Scaling Puppet Enterprise"
date: 2017-04-17 16:06:11 -0400
comments: true
categories: puppet, geekstuff, linux
---

In my former life as a consultant, I had to install all manner of configurations of Puppet for clients.  Some were small and some were large, but none were ***VERY*** large.  One of the big things I was finding back then was there just wasn't a lot of publicly available information regarding doing a full install and scaling it large.

So, I took some "research time" on my own and started to build out the configurations according to Puppet Labs' (at the time... now just "Puppet") documentation. The problem I was having was that the docs wouldn't ever lead me to a successful install following a chronolgical set of steps. I had to click into subpages, jump over to sub-sub configurations, and then jump back to the main docs to follow yet another trail down until I reached the end...lather, rinse, repeat.

**Some Caveats..**

**First**, this is probably no longer a good "HOWTO" unless you're installing an older Puppet Enterprise. It was created between 2015.2 and 2016.x, and likely has some amount of artifacting related to those versions.

**Second**, I'm going by docs I've recorded for my own use.  I wrote these as mentioned above through prototyping, tearing it down, starting again, and literally doing the entire install over and over until it worked "as advertised". A lot of this was really just ordering things the right way, and finding documentaiton for various pieces online at Puppet's documentation site as well as blogs, conversations, and plain old trial and error. I certainly can't warrant anything to anybody for any reason. As with most open source/creative commons assets, "it works for me, hope it works for you, and if it doesn't, sorry about that."

**Finally**, I hope to use this as the springboard to start brain-dumping all my old notes, conversations, ideas, and other prototyping I did in my home lab.  There's still a fair amount of documentation I cannot use or touch because they belong to my former employer or Puppet Labs, so some things may be less than clear and usually because I'm dancing around an NDA, noncompete, or just plain being a nice guy. If I inadvertently reveal something I shouldn't, chances are it could disappear without a trace, but I'll still make a note that I removed something, and try and replace whatever it is with published docs.

In short: I want to help the community, but I'm walking a tightrope here, so please be kind.

**Format**

I hope to start easy with a decision making process for installing Puppet, how to choose a method, think about scale, and will likely have quite an opinionated view at times.  Once PE is installed, we'll add compilers, scale postgres, etc. but for starters, I hope to just have the following:

PE Master (MoM)<br>
Puppet DB<br>
PE Console<br>
HA Proxy Node for Compilers<br>
Two Catalog Compilers<br>
One ActiveMQ Hub<br>
Two ActiveMQ Spokes<br>
Two Agent Nodes for testing<br>

I know that's quite a number of nodes to get started with, but this after all a large environment infrastructure, and we want to scale big. 

**Required Nodes**

To put together all the required components for a good large installation, I've settled on the below specs. You can change those as you see fit, but note that some of the disk space requirements and related were due to Puppet's documented requirements _at the time_.  YMMV, of course, but this is what I consider to be a base level installation if you intend on scaling into the multiple tens of thousands of nodes. Be sure that if you're going to size this down that you're still meeting Puppet's needs in regards to memory, cores, and disk.  _(for a current listing of Puppet's reuqirements, you can look [here](https://docs.puppet.com/pe/latest/sys_req_hw.html) for more information.)_

![Puppet_Prerequisites](http://cvquesty.github.io/images/prerequisites.png)

In addition, you'll need to be aware of firewall requirements for such an installation.  Puppet has documentation regarding firewall configurations and needed ports at their website [here](https://docs.puppet.com/pe/latest/sys_req_sysconfig.html#for-large-environment-installations), but I'll insert the image and recount the requirements here.

![Firewall_Ports](http://cvquesty.github.io/images/lei_port_diagram.png)

In short:

| Port | Use |
| ---- | --- |
| 8140 | communication between nodes and Puppet Master of Masters |
| 8142 | Orchestration between nodes and Puppet Master of Masters |
| 61613 | MCollective between nodes and Puppet Master of Masters and ActiveMQ Hub and Spokes |
| 443 | HTTPS to Puppet Console |
| 8081 | Connectivity to PuppetDB from Puppet Console, All Masters, and ActiveMQ Brokers |
| 4433 | Node Classifier API Endpoint from Puppet Master of Masters and Compile Masters |
| 5432 | PostgreSQL DB port From master of Masters and Console |
| 61616 | AtiveMQ TCP Hub & Spoke comms channel |

This is a close approximation to what you need to know.  Detailed charts found in the above links, and a "point-by-point" port and use list is available to review.

In short, I've found it easiest to have all PE components on the same VLAN with no restrictions between them. If you are going to have a local firewall turned up on each node, you'll need to manage all the above communications as you see fit, but for the serving infrastructure (if in a secure environment, of course) you can likely drop host firewalls in favor of corporate ones.  In short, make it as easy on yourself as you see fit while balancing that toward your corporate security policy.

Finally, make sure DNS and NTP are all ready to go.  I can't tell you the number of times I've had major issues trying to get all this working, and NTP was off, or DNS didn't propagate as expected (it's always DNS, right?) or some other similar seemingly unrelated piece was not restarted or some such. Just make sure that all nodes resolve to their respective FQDN from all nodes. Obviously, the easiest way to do this is to simply put them all in DNS. You ***can*** manage the host files manually, but why would you want to do that?

If you're at this point and all ready to go, look to the next entry to get started.
