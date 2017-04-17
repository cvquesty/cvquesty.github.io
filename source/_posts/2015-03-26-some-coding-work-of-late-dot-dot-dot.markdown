---
layout: post
title: "Some Coding Work of Late..."
date: 2015-03-26 11:26:16 -0400
comments: true
categories: puppet,geekstuff
---
***I AM NOT A CODER***

I know that sounds a little silly with all the Coderwall links over there in the sidebar, but I’m not.

I got into this business as a lowly PC building guy, and worked my way into systems administration through light consulting. A necessary evil of the day was tweaking an autoexec.bat & config.sys to release as much memory to the user as was possible for applications. (This was long before Win95, FYI)

As progress and learning would have it, I landed myself a systems administration job and began to grow. Here, 22 years later, back to consulting (but on a much larger scale), I look back on my professional career and see and realize that there’s a LOT of code behind me. Perl, BASH, ksh, HTML, PHP, light Ruby, old DOS debug scripts, Puppet DSL, Expect scripting… tons of it. All encountered and fleshed out in the context of systems engineering and/or management over the years as situations and needs arose.

Fast forward to today. The juxtapostion of Development, QA, and Operations into one big hairy hard-to-define (but getting clearer) term known as “DEVOPS” is the landscape a new admin comes into, and he or she learns from the very beginning the principles of placing infrastructure definition into code, and working as a developer to enhance and automate the hard infrastructure of the operations world.

I say all that to say this… I’ve got some new releases on my GitHub I’d like to share with you to help you out while navigating in this world of DEVOPS. If you have to call me a coder because of it, I may frown, but it is what it is.

***Vagrant/Virtualbox Fun***

As I’ve been slowly revealing through my series in past months, there’s a lot of tools out there for working with Puppet and there’s a ton of the same to prototype for your company’s environment. One of these is Vagrant, and it has the ability, in a huge way, to help you automate the setup and teardown of sample infrastructures to work with your Puppet code in. I’ve just updated and released a few of these, and I want to tell you about them.

**Vagrant with CentOS 6.5 and PE 3.7.1**

If you look here, you’ll find my current project I use with customers. This is a Vagrant instance that turns up a 4-VM environment including a PE Master, a DEV, TEST, and PROD VM running the PE agent, Enterpise Console, Directory Environments pre-configured, r10k configured, and a simple set of Puppet Modules to get you started.

Most commonly, I share those with customers, coworkers, and community folks to get them started coding right away, and to have a platform with which to teach them how to deploy, merge, and promote code through an environment in a smaller version of wat they might already have in their company. This is the enviroment I spoke about at the Atlanta Puppet User’s Group last year in its current iteration.

**Vagrant with CentOS7 and PE 3.7.2**

Similarly, you can find a CentOS7 + PE 3.7.2 project here. Much like the above, you get the latest of PE with CentOS7 to help your prototyping over a more current OS.

**Vagrant with CentOS7 and Puppet OSS 4**

If you look up the word “experiemental” in the dictionary, this project right here is linked as an example.

I’ve gotten a very rudimentary working setup of a Master and one agent to install completely and autosign, and haven’t even scratched the surface of all the new goodies in Puppet 4. As Puppet 4 is still in Beta, this is not recommended in any way for any reason at any time for you to use for any purpose. :)

My hope here is to prepare myself for the PE4 features long before they’re released. I hope to work on getting directory environments and r10k working for this only to have a base from which to rapidly develop for PE4 when it’s released. **EXPECT THIS ONE TO GO AWAY IN FAVOR OF THE NEW PROJECT.**

**YOU HAVE BEEN WARNED**

I hope these projects assist you in rapidly creating a platform and developing for Puppet. If you have any questions, don’t hesitate to contact me via jsheets@shadow-soft.com, quest@questy.org, or one of the many other social media nexii you have available to you.

As always, these are in active, deep development. If you’ve got some Vagrant chops and/or want to contribute in any way, feel free to do pull requests, and I’ll integrate changes as soon as I’m able between customer engagements and/or other duties I may have here at Shadow-Soft.