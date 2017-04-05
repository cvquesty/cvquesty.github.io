---
author: admin
comments: true
date: 2014-05-20 13:15:14+00:00
layout: post
slug: dos-donts-puppet-environment
title: '"Do''s and Dont''s" for your Puppet Environment'
wordpress_id: 709
categories:
- Linux
- Puppet
- Systems Administration
- UNIX
---

IT Automation, like the features and functions offered by Puppet, is riddled with a number of pitfalls. Nothing dangerous or site-threatening in the near term, however evolving a _bad _plan can lead you down a painful path to re-trek when you ultimately need to demolish what you've done and re-tool, re-work, or even re-start from scratch.  Some simple suggestions can help smooth your integration, and also provide tools and methodologies that make changes in philosophy easy to test and implement as well as make the long road back from a disaster easy(-ier?) to navigate.

Here are some simple guidelines that can provide that foundation and framework:

**DO _Always_ Use Revision Control**

It would seem this would be a foregone conclusion in this day and age, but you would be surprised just how many shops don't have revision control of any kind in place.  A series of manifests or configurations might be tarred up and sent to the backup system, but aside from dated backups, there's no real versioning...just monolithic archives to weed through in a time of disaster.

Revision control puts you one command away from restoring those configurations and manifests (and even your data vis-a-vis "Hiera") to their original locations in the most recent state.

**DO Rethink Your Environments**

If you automate a bad workflow, you still have a bad workflow.  (albeit an automated one!)

Rethink how you do things and why.  Why do you promote code the way you do, and is there a better way to do it?  Why do you still have a manual portion to your procedure, and is it entirely necessary, or can this be remanded to Puppet to do for you as well.  What things are you doing well?  How can they improve?

Try to think through all your procedures.  There are more than you think, and they're often less optimized than they can be.  If you're going to implement Puppet automation, it's time to retool.

**DO Implement Slowly and Methodically**

Another pitfall a lot of shops wander into is they try to do too much all at once, and do none of it well.  Either they implement too quickly and migrate a huge environment it took years to build (sometimes as much as a decade!) through a single professional services engagement or at an unrealistic pace.  Automation is complex, but if you take the time to implement correctly, piece-by-piece and hand-in-hand with your rethinking of your environment referred to above, you can revolutionize the way you work and make the environment considerably more powerful, considerably easier to work with, and ultimately release yourself to work on much more interesting problems in your environment.  Take your time to build the environment you want.

**DO Engage the Community**

By using Puppet, you are the beneficiary of the greatest software development paradigm in history -- the Open Source movement.  People all over the world have taken part in crafting the powerful tool you have before you.  If you are able to help in like manner, by all means contribute your code to the community. (With your data in Hiera, this is easier than ever!)  Join a Puppet Users Group.  Share your clever solutions to unique problems with the community via GitHub, the Puppet Forge, your website... give back.  The more you pour in, the more you get out, and something you solve may end up baked into the final product one day in the future.

**DON'T Pit Teams Against Each Other**

DON'T make this a DEV vs OPS paradigm.  This is a marriage of the best tools of both worlds.  Depending on how your culture breaks down, this could be an OPS-aware way of doing development, or a DEV-informed way of doing operations.  You need to remember one thing in all of it.  The marriage of these worlds is a _teamwork_ effort.

I was averse to the term DEVOPS when it first started being used, as it was a tool of the development world I was engaged with to cede root level access to developers.  In a properly managed, secure environment, this is always a no-no.  Development personnel are not trained systems people and rarely are.  By the same token, never ask your systems people to delve into core development, or to troubleshoot your developers' code.  They are not tooled for that work.

This does not say that one is better than the other, nor does it say they do not share a certain amount of core skills at the basest levels. Much like the differences between civil and mechanical engineers, each has a base level of knowledge that ties them together, but each is highly specialized.  You don't want your civil engineer building machine tools just as much as you don't want your mechanical engineer building bridges.  Each discipline is highly specialized and carries with it nuance and knowledge you only gain through experience...experience _on _the job.

Instead, find a culture and a paradigm that joins the forces of these two disciplines to build something unique and special rather than wasting time with dissension and argument.

**DON'T Expect Automation to Solve _Everything_**

I know, that sounds like a sacrilege at this point, but its true.  No matter how automated your site becomes, how detailed your configuration elements are, or how much you've detailed your entire workflow, you still can never replace the element of human consideration and decision-making.

Automation, as I've said before, automates away the mundane to make time for you, DEVOPS person, to work on really interesting and curious work.  You can now write that entire new whiz bang gadget you've been conceptualizing for the last several years, but have never quite gotten there because you were too busy "putting out fires".  Puppet automation is definitely a watershed in modern administration and development, but people are still needed.

Another "intangible" you may not readily think about when considering a DEVOPS infrastructure is one of _culture.  _The best places to work are always the best cultures brought about by the right collection of people, ideas, personalities, and management styles.  When you find that right mix of people and ideas, the workplace becomes a, forgive me, _magical_ place to be.  Automation can never make that happen.

**DON'T Starve Your Automation Environment**

Automation solves a lot of things, but one thing it cannot do is feed itself.  This particular animal has a ton of needs over time.  From appropriate hardware to personnel, the environment needs time, attention, and consideration.  Remember that this is the "machine tool" of your whole company.  It is the thing that builds and maintains other things.  As such, its priority rises above that of the next web server or DNS system.

Always allocate enough resources (read: money, personnel, and time) to your environment.  If that means engineer time to work on a specially project and to do the job _right_, that's what it means.  And, yes, it's more important than meeting an arbitrarily assigned "live date" to your new widget or site or application.  The environment comes first, and all else follows.  If you give the resources and time to your automation initiatives it deserves, a number of years down the road you will look back and be amazed at the sheer amount of work your team was able to accomplish just by keeping this simple precept.

**DON'T Stop Evolving**

Never stop learning.  Never stop bettering yourself or your environment.  Always keep refactoring your code.  (i.e., if you wrote that Apache module 4 years ago, chances are good that what you've learned in the interim can go back into making that module even better.)  Always keep your people trained and engaged on the latest developments in Puppet and all the associated tools.  Never stop striving to be better and never stop reaching.  I may sound lil your coach from high school in this, but those principles he was trying to impart hold true.  If you continue to drive forward and reinvent yourself as a regular part of your forward pursuits, the endpoint of that evolution will benefit you personally, your team both vocationally and culturally, your company's efficiency, and your environment's impact on your bottom line.

**Conclusion**

If we keep a stronger eye on our environment and tools that rises above the simple concept of "that software I bought" and "fit it in between all the other things you have to do" and give Puppet its proper place in our company, it can truly revolutionize your workflow.  However, when properly placed culturally and from a design, implementation, and workflow perspective, it can transform any shop on levels not readily observable when looking at the price tag or the resource requirements list.  DO let Puppet transform your environment and workflow and DON'T be afraid to take the plunge.  It's exciting, challenging, and can easily take your company to the "next level".
