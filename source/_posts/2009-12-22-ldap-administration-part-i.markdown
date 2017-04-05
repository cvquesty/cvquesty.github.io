---
author: questy
comments: true
date: 2009-12-22 16:59:45+00:00
excerpt: I plan to tell this story mostly in the context of a RedHat-ish rpm-based
  distro, and the setup of both the server and a Linux client to tell both sides of
  the story.  I will use my own terminology to begin, making it very simple to explain
  precisely the setup we're looking for without getting into a lot of LDAP-ese.  As
  we go through the setup, I will slowly start to relate my description back to LDAP-ese
  so you can, once your setup is working, go back to the documentation and determine
  "just what happened here"
layout: post
slug: ldap-administration-part-i
title: LDAP Administration – Part I
wordpress_id: 104
categories:
- LDAP
- Linux
- Systems Administration
- UNIX
tags:
- LDAP
- Linux
- Systems Administration
- UNIX
---

One of the things I've worked on over the last year is setting up a centralized LDAP server for authentication in a UNIX environment.  My clients range from early RedHat to our current distro of RH5.2 or CentOS5.2.

A huge issue I've run across while doing this work is that I had to piecemeal the information together from all over the Internet because the grand majority of what I could find out there was either for Windows clients or non-POSIX login authorization.

I plan to tell this story mostly in the context of a RedHat-ish rpm-based distro, and the setup of both the server and a Linux client to tell both sides of the story.  I will use my own terminology to begin, making it very simple to explain precisely the setup we're looking for without getting into a lot of LDAP-ese.  As we go through the setup, I will slowly start to relate my description back to LDAP-ese so you can, once your setup is working, go back to the documentation and determine "just what happened here"

At the end of the story, I'm going to have a LOT of keywords so Google gets a good sampling of this to help someone who may be like I was, looking for a very specific UNIX auth implementation.

**Server**

One of the big things I needed first was to have the server packages installed.  When I did the original installation, I went a little overboard and downloaded and installed everything from the patch level I used, but the essentials are:

openldap
openldap-servers
openldap-clients
openldap-devel

After getting these installed (via either yum or rpm -i...whichever is your preference) then you should have a subset of important files and tools lying about around your system.  These will be (in no particular order):

/etc/openldap
/etc/openldap/slapd.conf
/etc/ldap.conf

The /etc/openldap directory houses all the files pertaining to your server piece.  From your certs to the appropriate schemas to an example DB_CONFIG for the LDAP backend (Berkeley DB...more about these later...)  The primary file of interest in /etc/openldap will be slapd.conf.  In /etc/ldap.conf, you have the server configuration file.

In my setup, the purposes and configuration of these particular files seemed to be the biggest hinderance to getting everything running as I would expect.  At one moment, there was a config element in /etc/ldap.conf and in another there was one in /etc/openldap/slapd.conf.  It appears there has been some overlap from version to version, and even now you have to take all files together, viewing it all as a single configuration.  That's where we will start.

**/etc/ldap.conf**

I'd say that my first confusing issue was that this file serves different functions depending on whether you are looking at a server or a client.  For the client, it is a simple, short file specifying the server and the ldap URI to use for searches.  In the server context, it is for the LDAP nameservice switch configuration and the LDAP module for PAM.

**/etc/openldap/ldap.conf **

Let's start with the client config.  I know this is putting the cart before the horse, but it is the simplest part of this procedure.  If we configure it now and utilize it in it's easiest configuration, we know that once resolution and searches are functioning, the server is setup correctly.

First, let's talk about your organization a bit before launching into this configuration.  Recall that our entire purpose at this point is POSIX-compliant user/pass functionality and everything that implies.  I'm not looking for a phonebook (what seems to be a large portion of the tutorials out there on the 'net and in the O'Reilly LDAP book) and I'm not looking to find where employee X sits.  I want user/pass/GECOS, etc. of a POSIX-compliant login.

Most organizations are on the Internet these days.  Let's take my mythical company "Bob.com".  Bob's company, Bob.com has a website and all the standard DNS resolution and email and the like for Bob.com is already working perfectly.  It already has a nifty website driving tons of customers to Bob's company.  As for the infrastructure of UNIX logins and passwords, the ONLY THING that Bob.com and LDAP have in common is that their bits and bytes are traveling over the same wires as Bob.com's Internet traffic.  That's where the connection ends.  When we start talking about the naming in the LDAP space, even though Bob.com is valid in the Internet as well as the LDAP world, they have absolutely nothing to do with one another from a technical perspective.

In LDAP-land, Bob.com is the name of the company we will be serving.  So, think of this as the top of your tree from which all the other branches will grow.  Bob.com is your organization.  It is also the top of your tree.  This is a **_conceptual_** designation.

We will be installing the LDAP server on a piece of server hardware hanging in a rack.  This server will have a DNS hostname, and will be called ldap.bob.com.  This has absolutely nothing to do with LDAP-land.  This is simply the every day server naming and numbering like you would do for a new name/app/web server and the like.

The top of our _conceptual_ tree in our organization is the bob.com **_base_** name.  In LDAP-land, we refer to this as your "**_search base_**".  This is the name root from which all things flow.  Like an OU in active directory or a root filesystem in UNIX, everything happens under this.  So, for our LDAP-land based conceptual idea of our organization, we will have a _base distinguished name_ of Bob.com.  In LDAP-ese, we refer to a base by it's separate parts.  In the case of the base name (for our purposes) these separate parts are known as "domain components".  This, should you choose an LDAP name space to match your Internet name space, is the sole point where these two worlds converge.  Bob.com is an Internet domain name while the LDAP-ese separations of the search base are called "domain components".

Let's see what this looks like:

Internet domain name:                   bob.com
hostname of the server:                   ldap.bob.com
LDAP Search base:                          dc=bob,dc=com

If you remember some of your DNS-fu, you'll recall that the host/domain portions of the Internet name have significance in IP-Address land, but that is outside the scope of our little discussion here.  The important thing for you to know is that bob.com is an Internet-based domain name that can be resolved in your browser of choice to a location on the Internet for your viewing pleasure.  The LDAP name space of dc=bob,dc=com is an LDAP-formatted representation of the search base of your organization conceptually referred to as Bob.com.

I know that was painful, but the distinction is paramount to proper understanding of this process, how it is named, and how it is used.

So, let's take our client configuration and give it a whirl...

**Configuring a Client**

Simply stated, the client only needs to know three things.  First, it needs to know our base conceptual name (or search base), an Internet-formatted [URI](http://en.wikipedia.org/wiki/Uniform_Resource_Identifier) to know how to get to the host that has this storehouse of information, and the OS itself needs to know to look at LDAP for this information rather than local files.  Let's look at a client machine's local file:

File:  /etc/openldap/ldap.conf

#
# LDAP Defaults
#

BASE dc=bob,dc=com
URI ldap://ldap.bob.com
TLS_CACERTDIR /etc/openldap/cacerts

That's really all there is.  The search base and the URI to find the server.  You already know about the file that tells UNIX where to look for information.  It is the file **/etc/nsswitch.conf**

This file contains mappings regarding what the system should look at for which services.  The file is in the format:

service:     location1 location2 location3...

You can have as many locations as you need for UNIX to search, but we're only going to worry about two.  local files and the LDAP store.  In the /etc/nsswitch.conf file, you will have three lines for authentication we are concerned with.  They will look like so:


<blockquote>passwd:     files
shadow:    files
group:        files</blockquote>


This essentially tells the system  that for passwords, password hashes, and groups that you should look at the local files.  Pretty straightforward.  However, to point us at the LDAP server for this information, you need only change this file in the following way:


<blockquote>passwd:     files ldap
shadow:    files ldap
group:        files ldap</blockquote>


What this will do is check the local files for a user and then the LDAP store for that user.  If they do not exist locally, they will resolve from LDAP.  The caveat here is that the user you're trying to login with for testing LDAP should not already exist on the client system.  If it already does, you have a possible issue in that things will work, and you'll never know if you're resolving from local files or LDAP.  I'll show you how to handle this later.

That's it for the client.  Now we have a client configured to shout at ldap.bob.com using the search base of dc=bob,dc=com looking for password, shadow, and group information for POSIX compliant logins on your client host.

Next time, we'll start setting up the server.
