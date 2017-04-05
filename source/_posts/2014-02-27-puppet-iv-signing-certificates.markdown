---
author: admin
comments: true
date: 2014-02-27 20:32:04+00:00
layout: post
slug: puppet-iv-signing-certificates
title: Puppet IV - Signing Certificates
wordpress_id: 623
categories:
- Linux
- Puppet
- UNIX
tags:
- Linux
- Puppet
- UNIX
---

_**Introduction**_

We've covered a lot of ground already in our tutorials thus far.  In [Tutorial I](http://questy.org/2014/01/puppet-i-installation/) we installed our Puppet server.  In [Tutorial II](http://questy.org/2014/01/puppet-ii-client-installation/), the client, and in [Tutorial III](http://questy.org/2014/02/puppet-iii-initial-configuration/) we configured the server and client to start communicating.

In this 4th installment, we'll cover the signing of SSL certificates between your Puppet Master and Client, and running a simple test to ensure your setup is working and ready to function as a master=>slave pair.

On your server, be sure to start up the Puppet Master, so it can receive connections from outside sources, like so:


_**sudo service puppetmaster start
sudo chkconfig puppetmaster on**_


Next, on your Puppet client, run a test run so the client will start, create an SSL certificate for the client host, and make an initial callout to the server.  It will exit immediately, but the server will now know about the client, and we can sign the certificate over there.


_**puppet agent -t**_


When you run this command, the output should look something like this:

[caption id="attachment_625" align="alignnone" width="720"][![Certificate Signing Request Example](http://questy.org/wp-content/uploads/2014/02/agent_cert.png)](http://questy.org/wp-content/uploads/2014/02/agent_cert.png) Certificate Signing Request Example[/caption]

Now, SSH to your Puppet Master, and run the following command:


_**puppet cert --list**_


If everything has gone well, your server will have now seen your client and will report as much like so:

[caption id="attachment_627" align="alignnone" width="719"][![Server Cert Requests in Waiting](http://questy.org/wp-content/uploads/2014/02/server_cert.png)](http://questy.org/wp-content/uploads/2014/02/server_cert.png) Server Cert Requests in Waiting[/caption]

Finally, so _sign _the certificate so there is an SSL connection between the client and server, and all future Puppet actions between the two pass over that connection, simply sign the certificate request on the master as follows:


_**puppet cert sign puppetclient.example.com**_
_(or whatever your hostname is)_


 The server will sign the certificate and make a tabular listing of it in its database finalizing the connection.  You can see this output by using the "list all" options as follows:


_**puppet cert list --all**_


and see the output which should resemble the following:

[caption id="attachment_635" align="alignnone" width="718"][![Signed Certificate](http://questy.org/wp-content/uploads/2014/02/signed_cert.png)](http://questy.org/wp-content/uploads/2014/02/signed_cert.png) Signed Certificate[/caption]

_**
Conclusion**_

Now we're ready to do work with Puppet.  You can actually stop here and fully manage your site with Puppet, however there are several addons and feature-sets yet you can install to expand Puppet's functionality.  We will look at some of these tools in future installments.
