---
layout: post
title: "Vagrant and Docker Love"
date: 2015-01-15 21:31:15 -0400
comments: true
categories: geekstuff,linux,puppet
---

Not a full-on post, but more a note for myself to both investigate and test this… It would seem that Vagrant has now added provisioner support for Docker here. Be looking for a new post on this in the near future!

I’m working steadily to start off the new year, so my posting may be somewhat sporadic, but I will continue to blog Puppet fundamentals, supporting tools, and related items as I have the chance.

And in that vein, I encountered something this week I wanted to share with you.

It would seem that the new feature whereby you can include facter facts in a module for pluginsync to distribute them, but using the new mechanism of:

/modulename/facts.d/external_fact

is not 100% reliable when distributing those facts. By this, I mean the following observed behavior.

You create your fact and place it in the above directory. Say, a shell script that gives you a value.
You make your fact executable, and running it natively at the shell works perfectly.
You do a puppet agent run, and the fact syncs to the agent machine, but never becomes available in the facter table.
You find the sync'ed location of the fact (in the losgs from the sync) and run it manually, and it works perfectly.
I spoke with some folks at Puppet just in a conversation describing what I was seeing and they suggested the following workaround:

Make the fact a file resource and place the fact in /etc/puppetlabs/facter/facts.d. This makes the fact available to the facter system, and displays correctly in the facter table and responds to the facter -p as expected.

That’s it for now! Look for more to come on workflows and tools in the very near future.