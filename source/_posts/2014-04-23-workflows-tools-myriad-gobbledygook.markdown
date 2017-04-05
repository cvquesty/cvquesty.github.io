---
author: admin
comments: true
date: 2014-04-23 18:17:41+00:00
layout: post
slug: workflows-tools-myriad-gobbledygook
title: Workflows, Tools, and a Myriad of Gobbledygook...
wordpress_id: 660
categories:
- General
- Puppet
---

Ok, so first let's cover the gobbledygook.

I've had a lot of feedback on various parts of the blog here, and I thought I'd address a few of them here.

**Q:  Why didn't you just point at the appropriate Yum repos for installing Puppet?**

A:  Easy.  I can do "yum install foo" and never know what's going on behind the scenes just like anyone.  My goal here was to give a point-by-point installation guide so those who are interested could know what all the moving parts were and how they fit together.

**Q:  Why are you using the dashboard and not Foreman?**

A:  Also easy... at least at the time of this writing, The Foreman has been indicated to be the next front end to be the "de facto" standard, but as for right now, the Enterprise Console is essentially a turbocharged version of the Dashboard.  As such, when I begin to talk about extending the Enterprise Console, Dashboard would be the analogue by which you can most obtain the same experience (short of installing Puppet Enterprise itself).

**Q:  Why OSS and not PE?**

A:  Well, to be frank... PE is about as simple as you can get.  You can learn a bit about what portions to install and how to connect them all together across machines (something I plan to cover), and you can learn about constructing an answer file (or several for a large, complex installation),  but the vast majority of your PE installations will be a Q/A interaction with the installer.



**Workflows and Tools**

One of the biggest changes in how I deal with Puppet has been my adoption of and implementation of workflows as well as using some modern tools I've been made aware of by the Puppet Labs folks.  Among these are Vagrant, r10k, GitHub, and many others that work together and are tightly integrated and require a lot of configuration and setup to "make happen".  I intend to cover those here.

So, what are all these things?

**Vagrant**

Vagrant is a tool that allows you to pre-configure a small virtualized environment on your host consisting of a Puppet Master and any number of agents for use in a dynamic, iterative fashion.  By bringing up a Vagrant environment, I have a miniature development environment from which I can actually test Puppet code I write in a full PE environment and work out kinks you don't normally encounter when working independently on a single workstation.  From this environment, I can puppet-lint check all my code and then push all that code out of the Vagrant environment up to GitHub, then pull it down to my production instance as needed.  Further, it allows me to simulate all environments I have in my corporate setup (DEV, PROD, TEST, etc.) and commit those differences to the appropriate branches in GitHub.

**r10k**

The method by which I iterate, push/pull code and deploy to various environments both in my Vagrant instance and in my Production site is via r10k.  A leap forward past "puppet librarian", r10k is the glue that ties between GitHub and Vagrant as well as your GitHub instance and your Corporate site.  This can tie to public GitHub as well as Corporate (private) GitHub.

**GitHub**

GitHub is considerably more well known, but it's important to the whole process as you can read above.  GitHub is a revision control repository designed for rapid deployment, iteration, and isolated developer work with periodic pushes back to GitHub. (distributed development).  I intend to do a simple tutorial on GitHub as well.

Look forward to piece-by-piece coverage of each of these and my thoughts as I prepare for and take the Puppet Labs Certification test.
