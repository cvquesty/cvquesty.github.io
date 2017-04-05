---
author: questy
comments: true
date: 2010-08-05 21:03:09+00:00
layout: post
slug: ldap-authentication-for-apache
title: LDAP Authentication IV - Apache
wordpress_id: 136
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

Once you've got your handy LDAP server authenticating users for login throughout your environment, inevitably you find yourself getting around to the question of authentication for all your backend tools/services machines.  Can you authenticate from Apache to that same auth store to reduce administration burden across the environment?

Yes you can.

Recall from part I of this series that we have a server that is serving auth for our mythical "Bob.com" environment.  Recall also the important info regarding this environment:


Internet domain name:                   bob.com
hostname of the server:                   ldap.bob.com
LDAP Search base:                          dc=bob,dc=com




File:  /etc/openldap/ldap.conf




#
# LDAP Defaults
#




BASE dc=bob,dc=com
URI ldap://ldap.bob.com
TLS_CACERTDIR /etc/openldap/cacerts


Now, we add to the mix an Apache web server that has an area it wants to secure.  Instead of using the old htpasswd method to create a disk-file containing usernames and password hashes, it would make much more sense to simply point Apache's authentication system at our shiny new LDAP server so we eliminate yet another user/pass system we have to administer.

**Apache**

As many of you are already aware, Apache offers simple authentication to allow/disallow access to specific shared directories over the web.  The Apache Project has detailed documentation to help you get started [here](http://httpd.apache.org/docs/2.2/howto/auth.html). I'll cover the basics here.

Traditionally, in Apache's config file(s) you specify a directory you wish to have secured via the "Directory" directive like so:


<blockquote>

> 
> <Directory "/super/secret/directory">
> 
> 

> 
> AllowOverride None
> 
> 

> 
> Options ExecCGI Includes
> 
> 

> 
> Order allow,deny
> 
> 

> 
> Allow from all
> 
> 

> 
> AuthUserFile /secure/passwd/file/location
> 
> 

> 
> AuthType Basic
> 
> 

> 
> AuthName "Cool Security Name"
> 
> 

> 
> Require valid-user
> 
> 

> 
> </Directory>
> 
> </blockquote>




You then create your passwd file for Apache with the htpasswd utility, make sure the above path points to it, bounce Apache, and you're authenticating locally.  Problem with this is that every time you need to add a new user to your environment, every instance like this has to be touched individually and you have to add the user (whether by copy/paste or otherwise is irrelevant) in each.




Enter LDAP.




Fortunately, the configuration for such a change is quite easy; Simple textual changes to the httpd.conf to repoint from the local disk file to the LDAP store is all you need.  First, make sure that your LDAP modules are properly loaded into Apache:







LoadModule ldap_module modules/mod_ldap.so




LoadModule authnz_ldap_module modules/mod_authnz_ldap.so







This tells the Apache core to load up these at startup so that the directives you supply in the Directory stanza will be understood.  Next, let's add the appropriate info into the Directory section:







<Directory "/super/secret/directory">




AllowOverride None




Options ExecCGI Includes




Order deny,allow




Deny from all




AuthName "Cool Security Name"




AuthType Basic




AuthBasicProvider ldap




AuthzLDAPAuthoritative Off




AuthLDAPURL ldap://cool.ldap.server/dc=bob,dc=com?uid




AuthLDAPBindDN "cn=admin,dc=bob,dc=com"




AuthLDAPGroupAttribute memberUid




AuthLDAPGroupAttributeIsDN off




AuthLDAPBindPassword <your_bind_password>




Satisfy any




</Directory>







This simple change (after an Apache restart) should give your Apache server security over that directory, authenticating against LDAP.  Since your LDAP store is POSIX compliant for login, you have a few other things you can leverage in there as well.  Among those, are the "Group" field.  That's right, instead of going out and adding individuals and/or preventing them, you can specify an entire class of people (say LDAP_Logins for instance) that if the user belongs to that group, suddenly they have access to anything that is authenticating against LDAP.




To add that ability, you can add groups to the stanza above like so:







Require ldap-group cn=mygroup,ou=Group,dc=bob,dc=com




Require ldap-attribute gidNumber=500







_(you place your group number in place of the "500" above)_




Now that you have the magic behind LDAP auth for Apache, you can add this to any Apache web server of the 2.x variety, and do your security centrally.




The ease of addition for new users on your network is now as simple as adding the user to your LDAP store, placing them in the appropriate group.  Once Apache was restarted the very first time to make this work, you never have to restart it for user additions.  It will simply query LDAP, and LDAP will provide the credential response Apache needs to continue forward.




Next time:  Sudoers in LDAP!
