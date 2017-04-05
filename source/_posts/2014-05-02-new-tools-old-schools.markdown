---
author: admin
comments: true
date: 2014-05-02 18:24:37+00:00
layout: post
slug: new-tools-old-schools
title: New Tools and Old Schools
wordpress_id: 664
categories:
- Linux
- Puppet
- Systems Administration
---

**New Tools**

Once you're treading water in this whole DEVOPS thing, a lot of terms get thrown around and a lot of "newest bestest" gets offered up as the cure for everything including the kitchen sink... oh, and cancer.  However, I think when the hubbub is the loudest, that's when I really take a step back and ask myself what we're trying to do, and what's the simplest, most repeatable and safest way to make it happen.

As with any sufficiently new technology, there's a learning curve that accompanies such a shift, and new tools start to be speckle your horizon such that you wonder where to start, what is imperative and what is optional for you to occupy your time with.

Thus it is with Puppet-y stuff.

Given that the whole DEVOPS thing includes within it a heavy lean toward DEV, the regular rank-and-file sysadmin may find himself thrust into a world of development terms he may have only heard in passing in some random meeting or other.  The tools of the developer are as varied and arrayed as those for the operations guru and every bit as arcane (in some cases) as the esoteric shell command only installable from some guy in Russia's repo, with 30 or more command line switches to get a single piece of data upon which to work your evil schemes.

Well... that's why this blog.

I'm about to embark upon a coverage of a set of tools.  However, this set of tools I will not observe in a vacuum, but in light of a powerful workflow engine with which to empower you to become considerably more efficient in writing Puppet code.  The tools in particular I'd like to go over and, much in the same way I covered Puppet Open Source, instruct you step-by-step on how to install, configure, and use said tools are as follows:

Vagrant
r10k
GitHub
Vim (yes, Vim)
puppet-lint
Gepetto

This is by no means and exhaustive list, but what it does is collect together the best tools to assemble a workflow that will speed your work and not leave you spending all your time working on tools, but working on _Puppet Code, _which is our main focus and goal.

**Old Schools**

At the end of the day, the really important things are conceptual.  Whether you use a new whizbang tool to do the heavy lifting, several tools working together to achieve this goal, or you heavy lift all on your own, the process and the rules remain the same.

DO keep all your code in revision control.
DO syntax highlight and check your code validity
DO build repeatable, consistent environments in a timely fashion
DO have a method to share your work environments with others
DO document heavily both in and out of code segments
DO have a solid end-to-end workflow that enables rapid iteration

What of this is new?  Whether we're talking about keeping all your shell scripts in CVS, deploying your script repo with SVN, or deploying versioned code caches with ${PACKAGE_MANAGEMENT_SYSTEM}, we're talking about the same general good practices.  All Puppet and supporting tools does is make it something that is tightly integrated with management consoles.  To organize and institutionalize your _workflow _gives the steady underpinning to the cool DEVOPS tool to make everything repeatable, sharable, and collaborative.  This is where the power comes from.  THIS is where DEVOPS makes sense.

When your workflow is solid, your DEVOPS tools are strong, and your culture has bought in to both, mundane work becomes an afterthought and you get to work on the really interesting things that you've been letting slide while putting out fires that could've been best managed via configuration management anyhow.

My hope is to build the concept and context from the ground up to show you one tight, functional way this can be accomplished.  Let's start next time with Vim.
