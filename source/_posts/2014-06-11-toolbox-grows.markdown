---
author: admin
comments: true
date: 2014-06-11 16:57:18+00:00
layout: post
slug: toolbox-grows
title: The Toolbox Grows...
wordpress_id: 724
categories:
- Geekstuff
- Linux
- Puppet
tags:
- Linux
- Puppet
- Systems Administration
---

So far we've gotten our heads around some important things.  First and foremost, vim.  Our editor and companion for creating great code and ways to see our code in action and be able to determine at a glance whether our syntax is correct.  Also, we've looked at revision control.  The single largest "CYA" ohmygodimgladivegotanoldercopytorestoreto sort of paradigm where you can roll yourself back to previously "known good" revisions to save that day...besides that, it's just darned good practice to keep your code externally saved, revision controlled, and accessible.

I've also talked about importance of workflow clarity and quality.  If you implement a poor workflow, you just have an automated _poor _workflow. Key word here is "poor".

Next up on our browse through the "toolbox" is "Vagrant".  What is this Vagrant, you ask?

Virtualization is paramount in today's world in a number of ways and for a number of reasons.  For extending your server farms to handle even more application expression, to expand your own desktop machines to test/try different operating systems, and even just rolling up an ad-hoc VM so you can try something without touching a "real" machine in your environment.

Some may disagree, but I've found virtualization to be one of the most powerful tools added to the toolbox in years.  Not only can you prototype systems or applications, but you can prototype entire environments.  This is where Vagrant shines, and especially in the context of Puppet (master + clients), allows you to create a fully functioning Puppet environment upon which to develop, prototype, and test without ever jeopardizing even the least important system of your infrastructure.  I count that as a "win".  Let's see what this tool can do.

_**What *is* Vagrant?**_

According to its website:


Vagrant provides easy to configure, reproducible, and portable work environments built on top of industry-standard technology and controlled by a single consistent workflow to help maximize the productivity and flexibility of you and your team.


To achieve its magic, Vagrant stands on the shoulders of giants. Machines are provisioned on top of VirtualBox, VMware, AWS, or any other provider. Then, industry-standard provisioning tools such as shell scripts, Chef, or Puppet, can be used to automatically install and configure software on the machine.

There's a lot there, but it's just a fancy way of saying exactly what I said before.  Vagrant is essentially a framework system that wraps your virtualization engine to manage _environments _of VMs.  Here is where Vagrant will hold the power for us.

_**Virtualization**_

If Vagrant is the framework, then Virtualization is the foundation.  Now, I've chosen to use "Virtualbox" for my virtualization technology, but VMWare works every bit as well.  I am doing all my testing over Virtualbox, however, so YMMV.  Virtualbox is freely available from oracle, and you can download the appropriate version from Virtualbox at https://www.virtualbox.org.  I am running the latest version at 4.3.12 (as of this writing) and it serves the Vagrant system extremely well.

_**Vagrant**_

Next, you'll need to install Vagrant on your system.  You can find all the right packages at http://www.vagrantup.com.  I am currently running version 1.6.3 without errors.

[warning]I want to make a disclaimer here since I've had an issue or two with Vagrant on a platform I don't use-Windows.  I am a Mac & Linux user, and have had no issues using the Vagrant/Virtualbox combo on either of these.  However, literally every time I've used Vagrant over Windows, it's just been a mess.  I've known one person (ONE!) who has gotten Vagrant to work over Windows, and it required his getting into the product, editing code, etc.  As such, I wouldn't recommend it for those new to the platform.[/warning]

On the Mac platform, you get a .dmg file and can extract it run the installer.  Linux versions are available as RPM installs and Debian Packages.  Once you're installed, let's mess around a bit with Vagrant to see what we can do.

_**Getting Started**_

Vagrant is a unique tool in that it allows you to manage all these varied VMs, but adds a twist.  The big twist is that you don't have to have the _source _materials for the VMs you're installing.  In fact, the simplicity of turning up a new VM is astounding.  Take the following series of commands:


cd <your favorite directory>
mkdir precise32
cd precise32
vagrant init hashicorp/precise32
vagrant up


If your Vagrant is installed correctly, a number of things start to happen.  First, Vagrant places a file in your cwd called "Vagrantfile".  Your vagrant file (indie) looks like this:


[snippet id="34"]




Note that this is a long file with a lot of explanatory documentation.  In actuality, the most important part of your Vagrantfile can be summed up here:


[snippet id="35"]

These are the lines that are uncommented plus the top two declaratives that tell Vagrant what to do.  It's a very simple file that does some very powerful things.  First, it checks your home directory in the ~/.vagrant.d location to see if you already have the "precise32" Vagrant source "box".  (more on boxes later).  Next, if you do have this, it will simply start up a VM in your virtualization of choice with a randomized name.  For instance, mine is called "precise32_default_1402504453444_30545".  Vagrant takes away the selection of an .iso image, connecting it to the virtual CD/DVD Rom, starting an installer, etc.  It simply sends you a pre-rolled image, places it in your .vagrant.d directory, and provisions the VM to respond to Vagrant commands, and starts it up within Virtualbox.  Precise32 is simply a test scenario, as Vagrant's site has quite a number of varied and specially configured "box" files that you can use to prototype on at their "ready-made" box discovery site: https://vagrantcloud.com/discover/featured.  You can install boxes with too many variations and differentiations to enumerate here, and that's not really the point for our purposes... you may find these of great assistance in your own workplace, but let's continue.

When you run your "vagrant init" command listed above, it places a Vagrantfile, and when you do a "vagrant up", it automatically retrieves your box file, provisions, and starts the VM.  Now, by simply running "vagrant ssh default", you are now logged into this virtual machine!  You also have full sudo to become root and do any sort of damage you may wish to do.  If you logout ("exit" or CTRL-D), and type "vagrant destroy", the VM goes away and you have nothing in Virtualbox.

Were we to just stop here, the power inherent in being able to just have these "Vagrantfiles" (sort of like a "Makefile" for boxes) to spin up and down test scenarios at will is incredible.  But, let's look at this in light of the Vagrantfile, what it can do and how you can customize it.  There is an entire descriptive language surrounding Vagrant PLUS Vagrant has a plugin infrastructure whereby developers can extend Vagrant's capabilities.  We will capitalize on these later.

So, imagine a scenario where you can create a directory, copy a text file into it, run a single command, and it automatically provisions a 4-node Puppet Enterprise infrastructure, fully installed with a master and three agents, MCollective fully installed, PuppetDB installed and in use...  literally a full installation just like you would use for your infrastructure...  Now we get powerful.  NOW we have the ability to do some cool things.

Next time, that's exactly what we're going to do.


