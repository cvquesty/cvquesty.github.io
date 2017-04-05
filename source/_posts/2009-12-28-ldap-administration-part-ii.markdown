---
author: questy
comments: true
date: 2009-12-28 11:53:02+00:00
layout: post
slug: ldap-administration-part-ii
title: LDAP Administration – Part II
wordpress_id: 112
categories:
- Geekstuff
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

Ok, so in our previous installment, we got LDAP all configured up from the client perspective.  I'll cover some other client-based niceties such as extra PAM modules and security in our third installment.  For now, back to the show!

**The Server**

The lightweight Directory Access Protocol (LDAP) gives us tons of things to use to help make the system login/logout process easy to manage and maintain.  The LDAP server runs over the Berkeley Database (BDB) as a data backend, and has a [long history](http://en.wikipedia.org/wiki/LDAP) as a protocol/standard.

As we said last time, the server configuration consists of the contents of the /etc/openldap directory with the primary file of interest being the /etc/openldap/slapd.conf file.  The secondary file of interest will be the /etc/ldap.conf file, another server configuration file.

**/etc/openldap/slapd.conf**

First, we will cover the ldap daemon (slapd) configuration file, /etc/openldap/slapd.conf.  This file is responsible for specifying information regarding what schemas to use, loggin parametwes, database specifications and locations, security, replication, and security grants.

Let's start with a very simple slapd.conf file:


<blockquote># OpenLDAP Server Configuration

# Primary Server Schemas
include         /etc/openldap/schema/core.schema
include         /etc/openldap/schema/cosine.schema
include         /etc/openldap/schema/inetorgperson.schema
include         /etc/openldap/schema/nis.schema
include         /etc/openldap/schema/redhat/autofs.schema

# Logging

loglevel 0
pidfile      /var/run/openldap/slapd.pid
argsfile    /var/run/openldap/slapd.args

# bdb database definitions
database             bdb
suffix                   "dc=bob,dc=com"
rootdn                 "cn=admin,dc=bob,dc=com"
rootpw                secret
schemacheck    off

# location of the data files
directory              /var/lib/ldap

# Indexes to help speed up queries
index objectClass,uid,uidNumber,gidNumber,memberUid     eq
index cn,mail,surname,givenname                                                eq,subinitial

# Grant users rights to change their own password
access to attrs=userPassword
by self write
by anonymous auth
by dn.base="cn=admin,dc=bob,dc=com" write
by * noneaccess to *
by self write
by dn.base="cn=admin,dc=bob,dc=com" write
by * read</blockquote>


This is a very basic slapd.conf file without TLS security information, without replication information, and without any frills at all.  Recall that our primary goal here is to get a basic LDAP server running and working without any frills.  We want the ting to work, and then we'll add features and security as time goes on.

**/etc/ldap.conf**

The /etc/ldap.conf file bills itself as the "configuration file for the LDAP nameservice switch library and the LDAP PAM module".  While we just configured the operation of the server, we must now put the "glue" in place so that PAM knows how to ask questions of the LDAP system to know if a user is allowed to login or not.  Again, a very simple /etc/ldap.conf file:


<blockquote># ldap.conf -- simple configuration

# Distinguished name of the search base
base dc=bob,dc=com

# The distinguished name to bind to the server with
binddn cn=admin,dc=bob,dc=com

# Password to bind to the directory with
bindpw secretpassword

# Search TimeLimit
timelimit 120

# bind/connection timelimit
bind_timelimit 120

# idle timelimit
idle_timelimit 3600

# PAM password format
pam_password md5

# Assume no supplemental groups for these named users
nss_initgroups_ignoreusers root,ldap,named,avahi,haldaemon,dbus,radvd,tomcat,radiusd,news,mailman

# Main LDAP configuration piece
uri ldap://ldap.bob.com
ssl no
tls_cacertdir /etc/openldap/cacerts</blockquote>


A couple of notes here.  First, the nss_initgroups_ignoreusers portion can most likely be left out.  This is an artifact (as far as I am aware) of being on a RedHat system as several of these users are defaults on RedHat.  Next:  The timelimits are just sane numbers, nothing special.  Should there be any need for tweaking this, it will be in your own experience across your own environment.

Much like the earlier client configuration file, this serves somewhat the same purpose, but for the PAM system as a client rather than a system itself as a client.

**Crank it Up!**

At this point, you should be able to start up your server to begin configuring it for use.  In RedHat-land, as many of you already know, this is typically a two-part process.  First, you set the service to start at boot and then you start it up.  First, run the command to set the runlevels in which you want ldap to start:


<blockquote>chkconfig --level 2345 ldap on</blockquote>


And then use the RedHat tools to start the service up:


<blockquote>/sbin/service ldap start</blockquote>


At this point, you should have a fully functional LDAP server running with no particular schema or setup, ready to take data (in our case, POSIX compliant login information) and serve it out to your infrastructure.

**Populating the Database**

With the RedHat packages come a myriad of important tools you need to help migrate your current authentication information into the LDAP store.  The location of these helpful perl scripts is: /usr/share/openldap/migration.  Before you use these tools, make sure you edit the **_migrate_common.ph_** file in this location to match your environment like so:


<blockquote>$DEFAULT_MAIL_DOMAIN = "bob.com"
$DEFAULT_BASE = "dc=bob,dc=com"</blockquote>


This file is sourced by all the migration scripts resident in this directory, and will automagically insert these values where necessary.

Next, let's do a trial-run of converting our passwd file in "LDAP-ese".  Since you've already edited the migrate_common.ph, you can directly migrate your /etc/passwd and /etc/group files like so:


<blockquote>_From the /usr/share/openldap/migration directory:_

./migrate_passwd.pl /etc/passwd ./passwd.ldif

_Similarly, convert your groups file as well:_

./migrate_group.pl /etc/group ./group.ldif</blockquote>


What these commands will do is convert your standard /etc/passwd and /etc/group file to [ldif](http://en.wikipedia.org/wiki/LDIF)-formatted files for import into your LDAP store.  Let's look at a record for an example:


<blockquote>

> 
> dn: uid=bin,ou=People,dc=best,dc=turner,dc=com
> 
> 

> 
> uid: bin
> 
> 

> 
> cn: bin
> 
> 

> 
> objectClass: account
> 
> 

> 
> objectClass: posixAccount
> 
> 

> 
> objectClass: top
> 
> 

> 
> objectClass: shadowAccount
> 
> 

> 
> userPassword: {crypt}*
> 
> 

> 
> shadowLastChange: 14251
> 
> 

> 
> shadowMax: 99999
> 
> 

> 
> shadowWarning: 7
> 
> 

> 
> loginShell: /sbin/nologin
> 
> 

> 
> uidNumber: 1
> 
> 

> 
> gidNumber: 1
> 
> 

> 
> homeDirectory: /bin
> 
> 

> 
> gecos: bin
> 
> 
dn: uid=bin,ou=People,dc=best,dc=turner,dc=comuid: bincn: binobjectClass: accountobjectClass: posixAccountobjectClass: topobjectClass: shadowAccountuserPassword: {crypt}*shadowLastChange: 14251shadowMax: 99999shadowWarning: 7loginShell: /sbin/nologinuidNumber: 1gidNumber: 1homeDirectory: /bingecos: bin</blockquote>


The LDIF format has a number of self-explanatory fields that you can see correlate to your /etc/passwd file.  There are other fields here that LDAP needs, but are beyond our concern at this point.  We'll come back to these later.

I picked a system-default user just to illustrate the format of the ldif.  This user generally has no password, and thus you cannot login using his credentials.  In the userPassword line above, for a regular login user, you would normally see {crypt} followed by a rather large SHA encrypted string representing the user's hash.

This brings about the question of how you wish to manage your users.  Generally speaking, I allow the local /etc/passwd file manage root and all system-default users (bin, spool, lp, etc.) and all users I add post-installation get placed into LDAP.  In that way, I manage the user centrally from the outset.  I'll be covering how I limit what users are allowed to access what systems in a later article.

**Importing the LDIF**

Once you have your LDIF, you have some decisions/edits to make.  look through your dumped ldif file.  Remove each stanza relating to the users you _do not _wish to manage via LDAP.  The resulting LDIF file will be the one we import.  Generally speaking, in a RedHat system I delete everything before UID 500 (which is usually me) and import everything else.

Once your LDIF file is how you want it to look, it's time to import.  Import it using the following command, modifying the search base and admin user to suit your configuration:


<blockquote>_ldapadd -W -x -D "cn=admin,dc=bob,dc=com" -W -f passwd.lif_</blockquote>


When you run this command, the LDAP server will churn for a moment (depending on how large your passwd file is) and then will return a clean shell with no errors.

**Time to "Get your GUI On"**

At this point, I usually point people toward the excellent Apache Directory Studio available at the [Apache Directory Project](http://directory.apache.org/).  This is a gui directory administration tool that can browse and edit your entire store.  The Apache Foundation describes it like so:


<blockquote>Apache Directory Studio is a complete directory tooling platform intended to be used with any LDAP server however it is particularly designed for use with the ApacheDS. It is an Eclipse RCP application, composed of several Eclipse (OSGi) plugins, that can be easily upgraded with additional ones. These plugins can even run within Eclipse itself.</blockquote>


Apache Directory Studio is available for Windows, Linux, and Mac OSX and is a freely-available Apache 2.0 licensed product.

Once you download and configure the Apache Directory Studio, you should be able to view, browse, edit, and manage all aspects of your LDAP store.

Next time, we will tie PAM into LDAP, work with security and group-based access as well as cover a few "odds & ends" to help you use your LDAP store in a variety of different ways.
