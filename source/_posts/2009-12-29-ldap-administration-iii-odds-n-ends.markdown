---
author: questy
comments: true
date: 2009-12-29 11:17:46+00:00
layout: post
slug: ldap-administration-iii-odds-n-ends
title: LDAP Administration III – ‘Odds ‘n Ends’
wordpress_id: 124
categories:
- Geekstuff
- LDAP
- Linux
- Systems Administration
- UNIX
tags:
- LDAP
- Linux
- UNIX
---

By now you should have a functional LDAP server, a properly configured client, and the ability to authenticate against your server.  However, the last couple "cogs in the wheel" are left to manage.

Generally speaking, when you add a user in UNIX-land, part of the process of the adduser command also sets their shell, creates a home directory for them and so forth.  As you may notice, there is no particular method to make this happen in Linux-LDAP.  You just have a store and a tool to manage the store.

Have no fear, it's PAM to the rescue.

As we said in our first installment, the following packages are all that is necessary to have the basic plumbing of LDAP to work:


openldap




openldap-servers




openldap-clients




openldap-devel


While that is true, there are a couple more packages that put the finishing touches on this to make it sing.  Those are:

nss
nss_ldap
nscd

NSS is a glibc mechanism or interface allowing access to the common UNIX databases such as the password or hosts database.  It is most commonly used to provide an interface to both local /etc/passd and /etc/shadow files and usually for the purposes of interfacing with LDAP or NIS.

NSS_LDAP is a set of C library extensions which allows X.500 and LDAP directory servers to be used as a primary source of name service information such as users, hosts, groups, and other such data historically managed via local flat files or a NIS infrastructure.

NSCD is a name services caching daemon that provides a cache for the most common name requests.  While useful in speeding up response to queries, nscd has it's own set of problems.  I'll cover only some of it's usefulness, and you should circumspectly determine whether you will need caching at all based on your organization's size and the size of your serving infrastructure.

**Home Directories**

First and foremost, to make the login process as pleasurable/smooth an experience as possible, there are some things that we generally do in standard administration practices that ease the user's experience.  Namely, we setup a standard set of configuration and environment files to provide a consistent user experience of the shell.  These are usually their home directory files such as:

.bashrc
.bash_profile
.bash_logout
.vimrc
.mozilla

Usually these are handled in the usual way via the useradd command, which also copies one of everything from the /etc/skel directory to your user's home.

As you can see, simply adding a user to the LDAP store would not give occasion for this to happen.  PAM to the rescue!  PAM contains a module that is their "mkhomedir" module.  Depending on your architecture, this lives in the /lib or the /lib64 directories under "security":

/lib/security/pam_mkhomedir.so
/lib64/security/pam_mkhomedir.so

What these modules do when properly invoked is on the first connection of a user that is validated as a proper user/password combination via LDAP, PAM sees that they have no home directory, and _creates the home directory for them just as if you, the administrator had created it with the **adduser **script._

This, by far, is one of the handiest modules provided by PAM.

**Turning on the PAM**

To turn on the mkhomedir facility for your servers and clients, simply do the following:

In /etc/pam.d there are a number of configuration files for handling login defs for varying circumstances.  What we are interested in is _system-auth_.  System-auth is also sourced by the sshd option, so this will work for both console logins and ssh/remote logins.

The last stanza of the system-auth configuration file is the "session" section.  On the second-to-last line between the pam_unix and pam_ldap designations, add the following:

session     required     pam_mkhomedir.so

This is a keyword set that tells Linux to load up and perform the functions offered by the pam_mkhomedir module.  Now, whenever you connect to a system configured in this way, it will automatically create the user's skel information and change ownerships to the proper settings so the user can login.

**Security**

You may note that I also referred to using group-based security.  In LDAP/PAM-land, this is just as simple as the above PAM setup.  Look into the individual system's /etc/security directory for a file called access.conf.  This file allows you to grant/deny access to individual users, groups, and also where they can connect to this machine from.  The syntax is clearly explained in the file itself, but here's an example.

Let's say I only want root, wheel, and the admins group to be able to access a certain system.  let's say they can access this system from anywhere.  Now let's say I also want the backup group to access the same system, but only from one host.  The setup is quite simple.  First, let's allow complete access to the people we wish to be able to connect from everywhere.  At the end of the /etc/security/access.conf file, add the following line:

+: root wheel admins : ALL

This tells the system to add access for root, wheel, and admins from ALL hosts.  Now, let's put in those backup admins, but only allow them to connect from the host we want them to come from:

+: backup : 10.1.10.10

This means the backup group can access this host, but only from the host 10.1.10.10.  There are many other methods of allowing and denying access in this file.  The documentation is contained within the file, so feel free to look through it for more information.

The MOST important line is the last line, however.  After you've setup all the people you WANT to access the system, that does no good if all access is still left on for everyone, everywhere.  The final line in the file should be to deny all access to anyone else not specified above, and would look like this:

- : ALL : ALL

Simply stated, this removes (or subtracts) all access for ALL other users from ALL other hosts.  This line is evaluated last after the preceding lines are, and will take the preceding lines into account when setting up it's rules in memory.

At the very worst, if you mis-order the lines here, you may not be able to login to the system and will need to boot to single-user mode and then use the article here on this site entitled "A Neat Trick" to make /etc read/write so you can change the configuration and then reboot back into your system's default runlevel, but if you're careful, this will be quite easy to configure and manage.

**Closing Thoughts**

I'm sure there's a few things I left out, and I will be revising this series at some point in time to allow for any omissions I've made.  I will also be adding in feature suggestions and will be following this up with a sudo article so you can overlay the "on-box" security set you'd like to see happen in your environment *after* your LDAP infrastructure is in place.

Feel free to drop me a note if you see anything amiss, and I'll be sure to correct it and give you props.
