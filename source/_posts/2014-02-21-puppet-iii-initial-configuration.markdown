---
author: admin
comments: true
date: 2014-02-21 03:23:42+00:00
layout: post
slug: puppet-iii-initial-configuration
title: Puppet III - Initial Configuration
wordpress_id: 552
categories:
- Geekstuff
- Puppet
- Systems Administration
tags:
- Linux
- Puppet
- Systems Administration
- UNIX
---

By now you should have 1 Server installation completed and 1 Client installation completed.  The only thing you're missing now is to link the two together.  The following client side configuration will be performed on ALL existing systems you have, to connect them to the server and to pull their configuration catalog down and apply the changes to the local host.

**Certificates**

By default, all communications in Puppet goes over SSL channels.  You can configure Puppet to use its own certificate signing mechanism or to use an external signing authority (outside the scope of this tutorial).  The signing process does two things.  First, it authorizes your client to communicate with the Puppet Master and receive configurations and keeps all that traffic secure from prying eyes.

**Node Lists**

Of course, the master needs a way to enumerate what hosts will be connecting, and how to apply what configuration where.  We'll look at node classification both on and off the host as well.

**Server Configuration**

Here we go.

To get the server and client talking, there is a small amount of configuration that has to happen.  It essentially tells the server who will be calling, and tells the caller the server's "phone number".  Very short and simple.  So, let's do that first.

_**Configs**_

Two files require editing on the server to prepare for your first Puppet run.  They are:


/etc/puppet/manifests/site.pp
/etc/puppet/manifests/nodes.pp


As of this writing, the OSS installation does not place either file, so you will have to create them.  Make their content as follows:


_**site.pp**_




` import ‘nodes.pp’
$puppetserver = ‘--FQDN--’
`


These two lines lay the foundation for the server's self-declaration as authority on configuration.

The first line: "import nodes.pp" tells Puppet to look for the nodes.pp file you are about to create in the next step for node definitions (and thus their individual configurations) and the next line indicates this puppet servers' complete name.  This line is very important, as configurations throughout the Puppet software product and the client node software continually refer back to this variable.  This setting takes the form of the fully-qualified-domain-name of the server.  It is recommended you use "puppet" as the name of the server in your infrastructure.

So if your site's name is "bob.com", this line would be:


`$puppetserver = 'puppet.bob.com'`


While any hostname should do, I might mention one caveat you should be aware of should you decide to use something other than 'puppet.x.x' formatting.  In the event your Puppet server cannot see or read the $puppetserver value, the internals of Puppet cause it to _default _to "puppet".  To avoid any strange outage-based silliness, I would recommend sticking with the puppet name.  While this may cut cross-grain with your particular host naming system, there is no reason why the root name of the system cannot be puppet.domain.com and you CNAME your super-special host naming to that.

Why not the other way around?  I've seen in the wild situations where Puppet (especially as it concerns certificate signing) take the output of gethostbyname() for signing over a designated CNAME, even when you set this variable to the appropriate value.  Rather than deal with the mess, I recommend the former.

Close the site.pp file.

Next, you'll want to configure your nodes.pp file.  Now, keep in mind that while we are specifying our nodes manually in this file for the purposes of this tutorial, we will be doing all manner of really cool things with node designations in the future.  So, for now humor me, and I assure you we'll step into the wonderful world of the ENC, Dashboard and other goodies all too soon.  Right now, we're just trying to get Puppet up, connected, and talking.

**nodes.pp**


 `
node '--FQDN--' {
}
`


Here we are specifying our very first node, letting the master know it's allowed to connect, and eventually putting a number of things in-between the curly braces {} specifying the configuration for (or pointers to the configuration for) our client node.   Just as above, the "FQDN" piece in between the single tics should be replaced with the fully qualified domain name of the client host you wish to manage.  As above, in our fictitious domain of "bob.com", we will simply call this host "puppetclient.bob.com", thus making the content of the nodes.pp file:


 `
node 'puppetclient.bob.com' {
}
`


For now, there's no need to  add configuration.  This is all we need to get started.  Exit from the nodes.pp file, saving it into the aforementioned spot of "/etc/puppet/manifests/nodes.pp"

Here ends the server-side configuration for this portion of our tutorial.  On to the client configuration.

**Client Configuration**

For the client configuration, a base-level file has already been provided for you, and only needs some customization.

The client configuration file is found at /etc/puppet/puppet.conf on the client (as well as the server).  The puppet.conf configuration file is an "ini-style" configuration file with [bracketed] headings and "key = value" line by line variables.

For our initial configuration we will be adding a couple lines.  First of all, at the end of  the [main] section, add the following line:

server = FQDN of the server

or, as above

server = puppet.bob.com

Next, in the  [agent] section, the following three lines (for our fictitious "bob.com" domain)  should be added at the end:

report = true
server = puppet.bob.com
certname = puppetclient.bob.com

The server line appears to me to be somewhat redundant, but I've always placed it in both locations, and cannot comment as to whether it is NOT needed.  The certname, FYI, is the FQDN of the client host you are configuring to connect to the server.

The purpose of the "certname" variable should be explained here a bit.  This is the name you placed in the "nodes.pp" file, and is also the hostname that is passed to the master in the key signing process.  You want all locations in which you're specifying your node name to agree (puppetclient.bob.com in our example), so it's highly important to ensure consistency throughout.

**Ready to Connect**

Ok, let's recap...

We've set our site.pp and nodes.pp files in place on the server, specifying the server name and ensuring there is a FQDN node designation in the nodes.pp.  Next up, we'll make our first connection to the server and test the connection.
