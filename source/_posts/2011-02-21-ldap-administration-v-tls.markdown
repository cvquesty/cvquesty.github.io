---
author: questy
comments: true
date: 2011-02-21 20:35:34+00:00
layout: post
slug: ldap-administration-v-tls
title: LDAP Administration VI - TLS/SSL
wordpress_id: 269
categories:
- Geekstuff
- LDAP
- Linux
- Systems Administration
- UNIX
tags:
- Corporate Linux
- Geekstuff
- LDAP
- Systems Administration
- UNIX
---

###### _This article goes hand in hand with "LDAP Administration - Part I" in regards to configuring the client. _


So, let's see where we are.  We have a master server you will be doing all administration work on.  This master server replicates to two hosts in the environment that serve LDAP queries to your clients.  These servers are replicants and are load-balanced under a VIP that is pointed to by the name you choose.  (in our case, ldap.bob.com).  You can change passwords at the client level, and have it pushed back up to master and replicated out to the environment immediately.

Finally, we need to talk about security.  There's a number of ways to do security, but RedHat has done a lot of the footwork for you.  Unfortunately, it's very poorly documented, and they really Really REALLY want you to use RedHat Directory Server for everything, so I don't guess it's a priority.

Essentially, we want to secure all queries floating around the network with TLS.  In a RedHat world, you simply need to make a couple changes at the server, restart LDAP, and then connect from TLS-enabled clients and all works just as it did before, except now it runs over an encrypted channel.

**First Steps**

RedHat has tried to ease the pain of generating certificates by placing all you need in a Makefile on-box.  navigate to **/etc/pki/tls/certs **and see that there is a makefile there.  Next, run:


**make slapd.pem**


to generate the needed files.  If it has already been done for you by the system, you will get the answer:


**make: `slapd.pem' is up to date.**


If you get this message, you're halfway there.

Next, edit the **_/etc/openldap/slapd._conf** file.  You will need to refer to the appropriate files to allow for secure operation.  Insert the following lines into that file:


**# TLS Security
TLSCACertificateFile /etc/pki/tls/certs/ca-bundle.crt
TLSCertificateFile /etc/pki/tls/certs/slapd.pem
TLSCertificateKeyFile /etc/pki/tls/certs/slapd.pem**


Next, edit the file _**/etc/sysconfig/ldap**_.  Make the following lines:


**SLAPD_LDAP=yes
SLAPD_LDAPS=no
SLAPD_LDAPI=no**


look like:


**SLAPD_LDAP=no
SLAPD_LDAPS=yes
SLAPD_LDAPI=no
**


Then, restart LDAP:  _**/sbin/service ldap restart. **_This does two things.  First, it tells the client where to look for the certificates, and then tells the system to only serve from the secure port 636.  (recall that we are on the replicants which are, in turn, servers themselves.  We have handled connecting to the master as well as setting the replicant up to receive queries)

Finally, we connect a client.

**Connecting the Client**

To allow a client to connect, you need the appropriate key on the client (public server key) to be able to exchange identities with the server, and establish the secure session.  To do this, you have to distribute this key you just made out to each client you wish to connect back to the server.

The key you will be distributing lives in **/etc/pki/tls/certs** and is named **ca-bundle.crt**.  Simply move this cert to your client (I use **scp** for such an operation) and place it into your openldap cacerts directory like so:


_**scp -rp ca-bundle.crt host.bob.com:/etc/openldap/cacerts**_


If you don't have rights to copy straight into the destination, send it to your home directory, then just move the cert there using "sudo".

Finally, you need to tell the system about the cert.  This is done in **/etc/openldap/ldap.conf **via three lines that tell the system how to connect, and where the cert lives:


**TLS_CACERTDIR /etc/openldap/cacerts
TLS_CACERT /etc/openldap/cacerts/ca-bundle.crt
TLS_REQCERT allow**





###### Next, we run the RedHat authentication gui tool (curses) called authconfig-tui. _**(Note: last time I had you hand-edit the /etc/openldap/ldap.conf file, and this is entirely still possible.  I am endeavoring to show you that there is a tool to do this work, should you desire to use it.  If not, simply add the above lines and change the URI to the one below, making sure /etc/nsswitch.conf is configured correctly, and you should be good to go.)
**_


In the left column, select "Use LDAP" and in the right column "Use LDAP Authentication".  Tab down to the "Next" button and press "Enter".

As misleading as "Use TLS" may be, do not select it.  :)  Instead, go down to your server line, and modify it like so:


**ldaps://ldap.bob.com:636**


Your base DN should already be filled out (in our case: dc=bob,dc=com).  Navigate to the "OK" button, and press "Enter".

This should conclude your client configuration.  Now, you should be able to run a query against LDAP, and the whole path be secure:


**id bob
uid=123(bob) gid=123(users) groups=123(users),456(bob)**


**Conclusion**

I'm sure I've missed or glossed over something highly important.  I am in the process of discovery on this particular topic, and this article is serving as my documentation store until I can get the whole thing cleaned up & finalized to push back into my work environment as official documentation.  I'll correct here as I find mistakes and omissions.
