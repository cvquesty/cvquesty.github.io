---
author: questy
comments: true
date: 2009-12-17 12:38:37+00:00
layout: post
slug: a-neat-trick
title: A neat trick…
wordpress_id: 101
categories:
- General
tags:
- Linux
- Systems Administration
- UNIX
---

When you have a large environment that can and does have a number of fingers into things, there is always the possibility that something could go awry on bootup such that you cannot boot fully into the OS.  When you do this, it's common practice to use Grub options or a CD to boot into single-user mode and then do whatever it is you need to do.

Sometimes, however, you get booted into single and go to edit a file, say... /etc/fstab for example, and when you open it shows as "readonly".

Here's a simple trick to do once you're logged in as root at a single-user prompt to do the work you need to do.  run:


**mount / -o remount,rw**




This will remount the root filesystem as read-wwrite during maintenance so you can edit whatever you like in the /etc directory (or anything else residing on the root filesystem.




I've had to use this a number of times for varying reasons, and thought it might help you a bit.
