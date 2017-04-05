---
author: questy
comments: true
date: 2011-02-18 17:57:33+00:00
layout: post
slug: ldap-administration-iv-replication
title: LDAP Administration V - Replication
wordpress_id: 258
categories:
- LDAP
- Linux
- Systems Administration
- UNIX
tags:
- Corporate Linux
- LDAP
- Systems Administration
- UNIX
---

Continuing our discussion of LDAP Administration, there's the matter of Replication.

So far we've created an LDAP store, turned up the server, configured a client, and even connected Apache authentication to it.  However, if we're going to use our LDAP server for enterprise authentication, then there's the small matter of "What happens when my authentication server wets the bed?".

As with anything in the enterprise, you have backup systems.  Sometimes they're failover systems, sometimes clusters.  Sometimes they're tandem systems, and sometimes they're load-balanced.  No matter the configuration, you have redundancy, resiliency, and scalability.  I plan to talk about one of the many scenarios available to LDAP administrators today; the idea of a master server and many replicants.

**Layout**

In my configuration, I have a single administrative parent.  This system is where we do all administrative level work.  This includes adding users, adding groups, reporting, and the like.  It is also the "provider" store to all replicants in our environment.  We learned earlier how to turn up a server that is queried directly.  Now let's learn, instead, how to configure this system to replicate itself.

Assume 3 systems total, ldap01.bob.com, ldap02.bob.com, and ldap03.bob.com.  ldap01.bob.com is our master server and our replicants are ldap02 & ldap03.  To tell the system it will be replicating, you will need to configure it to do so.  Shut down LDAP on the primary like so:


_**/sbin/service ldap stop**_


This shuts down all daemons and associated processes.  Next, we need to edit our **/etc/openldap/slapd.conf** to include information regarding where our replicants will be.    You must add a few lines to the master to make this happen.  Like so:


replogfile      /var/log/ldap/slapd.replog


replica uri=ldap://ldap02.bob.com:389
binddn="cn=admin,dc=bob,dc=com"
bindmethod=simple credentials=secret

replica uri=ldap://ldap03.bob.com:389
binddn="cn=admin,dc=bob,dc=com"
bindmethod=simple credentials=secret

This can be added at the end of the file.

Next, we take our fresh two servers, and turn up a similar system to what ldap01 was before adding the above lines.  In these systems, there are only two important lines to tell them they are replicants and not masters.  They are as follows:


updatedn "cn=admin,dc=bob,dc=com"
updateref ldap://ldap01.bob.com


That is literally the entire configuration.

**Populating the Replicants**

To have your schema transferred over, and to be working from the same general starting point, I find it important to copy your whole database over to start with.  This is easily done utilizing standard LDAP tools.

First, start back up your master server:


_**/sbin/service ldap start**_


Once you've done this, the database is up and ready for queries.  We will essentially dump our database for import on each of the replicants.  To do this, we will use the _slapcat _utility, redirecting the output to a file we can use to move around to the replicants.  Run _slapcat_ as follows:


**_slapcat >> master.ldif_**


this will output the contents of your LDAP store to a single LDIF-formatted file, suitable for import into other servers.  Simply copy this file to a generic location (such as your personal home directory) on each of the other servers, and we are set for import.

Once your file is in the new location, you're ready to import.  First, start LDAP as outlined above.  Next, add the LDIF to your store:


_**slapadd -l master.ldif**_


Probably unnecessary, but I usually restart my ldap server after the import, and now I'm ready to go.  Repeat the process on your third LDAP store, and your full environment is running.

**Next Steps**

So let's see where we are.

Master server up and serving.. check.
Two slaves configured as replicants, up and running.. check.

Now that you have your stores up, you have to do some testing.  Primarily, that the master replicates to the slaves.  The way I usually do this is use the [Apache Directory Studio](http://directory.apache.org) I covered in an earlier article.  I simply add a user on the master.  Then, I connect to each of the slaves in turn to see that the user has appeared there.  If so, then we're ready for the next steps:  High Availability.

You have two query hosts that can equally provide query answers from remote clients.  There are several ways you can make these available.  Round-robin DNS, HA IP failover, and load-balancing via a hardware load balancer.  I prefer the latter.  However, to do so, you need a way to tell the load balancer that your LDAP store is up and responding.

I prefer to use a small script on the system that can be served up via HTTP to the load balancer that does a simple operation.  First, it does an LDAP search, looks for information, and then prints out to the web page it creates a simple "UP" or "DOWN" message for the load balancer to key on.  The script looks like the following:

[caption id="attachment_265" align="alignnone" width="150"][![](http://questy.org/wp-content/uploads/2011/02/healthcheck-150x150.png)](http://questy.org/wp-content/uploads/2011/02/healthcheck.png) Healthcheck Sample[/caption]

As you can see, all we do is simply do an _ldapsearch _against our bob.com domain, look for the home directory for the admin user to look like "/home/admin".  If the answer returns, we say "UP", if not, we say "DOWN".

Place this script into your "cgi-bin" directory, make it executable _(chmod 0755 <filename>)_ and simply call it in your browser via the URL:  http://yoursite.com/cgi-bin/<filename>.  If you have Apache properly configured (outside the scope of this document) to serve CGI Executables, you should get the status of the individual system.  Do this for both your replicants.

Finally, ask your network team to configure these two systems in a load-balanced configuration behind a VIP (virtual IP).  Have a sensible DNS name pointed at this IP (ldap.bob.com, for instance) and you're in business.  Now, when you configure your clients to authenticate against LDAP (Article #1 in this series), you just point them at the ldap.bob.com name.  If either of the systems go out, the load balancer will point you to the machine that is up to serve your requests.

**Conclusion**

I hope this gives you a basic direction to go in getting high-availability setup for your system through a combination of replication and load balancing.  There are other methods for HA in the replicants.  Perhaps we will cover that soon.

Next up:  Securing your LDAP installation.
