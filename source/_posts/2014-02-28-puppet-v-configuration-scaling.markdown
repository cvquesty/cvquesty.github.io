---
author: admin
comments: true
date: 2014-02-28 20:12:57+00:00
layout: post
slug: puppet-v-configuration-scaling
title: Puppet V - Configuration and Scaling
wordpress_id: 639
categories:
- Linux
- Puppet
- Systems Administration
tags:
- Linux
- Puppet
- Systems Administration
---

_**Introduction**_

The name of this portion of our tutorial was difficult to determine.  This is another set of configurations, but we will also be scaling Puppet to handle production quality traffic, be an external node classifier (ENC), have a backend database, employ an enterprise class web server, turn up a console...  there's a lot to do.  We are indeed configuring the backend, but also scaling Puppet to handle your environment...hence the name.

This portion assumes you've followed all previous tutorials from I-IV, have your certs signed and are complete and ready to go with Puppet "as-is", you simply have not installed any of the following add-ons.  As mentioned last time, you could begin to write manifests and modules right now, using Puppet "as-is", never utilizing any of the other features.  However, the "out-of-the-box" configuration of Puppet is not ready for enterprise use.  Perfect for a small development environment...perhaps up to 25 hosts or so, the Puppet server as installed includes a small WEBRick server (ruby-based) and is not intended to handle large site traffic profiles.

To make Puppet "enterprise-ready", we need to do a few things.



	
  1. Install the Puppet Dashboard

	
  2. Install the Puppet DB

	
  3. Install passenger + Apache modules

	
  4. Install MySQL


The dashboard gives you a GUI configuration mechanism as well as an external node classifier.  The PuppetDB is a centralized config storage mechanism for all node facts and configurations.  Passenger+Apache is the piece that replaces Puppet's WEBRick server, and MySQL holds the database for the dashboard.

This will be the longest portion of the tutorial series, and all the separate & individual pieces will be interdependent, requiring us to do all the work first with configuration testing at the end.  Let's get started.

_**Packages**_

Early in tutorial I, I had you install a number of packages.  Had you stopped after installment IV, you'd have had no need for a few of them, but I wanted all the packages to be on your system as prerequisites to avoid later installation headaches.  However, we do want to get the EPEL package set onto the system to add one prerequisite (and it's a darned fine repo to have, should you need it for other things)

To install EPEL, run the following command:


_**sudo rpm -ivh http://mirrors.kernel.org/fedora-epel/6/i386/epel-release-6-8.noarch.rpm**_


The "Extra packages for Enterprise Linux" (EPEL) set is an important addition to any server set.  Especially to install the package prerequisites we need.

_**Installing Packages**_

_**Passenger:**_
First, we will be installing Passenger.  The passenger packages are in their own repository, not hosted at Puppet Labs.  First, import the GPG key:


_**sudo rpm --import http://passenger.stealthymonkeys.com/RPM-GPG-KEY-stealthymonkeys.asc**_


And then install the passenger release repo:


_**sudo yum -y install http://passenger.stealthymonkeys.com/rhel/6/passenger-release.noarch.rpm**_


Finally, install the Passenger Apache module to tie everything together:


_**sudo yum -y install mod_passenger**_


Congratulations.  The groundwork for Passenger is now installed.

_**Dashboard
**_Since we have done so much preparatory work, the dashboard install is quite simple:


_**sudo yum -y install puppet-dashboard**_


Simple.

**PuppetDB
**PuppetDB is installed a little differently, using Puppet itself to get and install the package:


_**sudo puppet resource package puppetdb ensure=latest**_


This procedure takes a bit of time, but when complete, the PuppetDB is now installed.

**MySQL
**MySQL is installed via the usual yum repos, but we will also turn it on and have it ready for use as well as create our users and remove unneeded and unsecured accounts for the system.


_**sudo yum -y install mysql-server
****sudo /sbin/chkconfig mysqld on**_
_**sudo /sbin/service mysqld start**_


Let's set the database root user's password:


_******/usr/bin/mysqladmin -u root password '<new-password>'**_
_**/usr/bin/mysqladmin -u root -h <FQDN> password '<new-password>'
mysql -u root -p**_


A few words here, for those of you unfamiliar with MySQL.  We are setting the root user's password to be able to administrate the database.  The simplest way to set up this initial security is using the "mysqladmin" tool provided by MySQL.  Note that when I use <> in these above, this is where your site-specific information comes into play.  For <FQDN>, for my example purposes I would replace this with puppet.example.com.  The password setting & changes, then, would look like so:


_**/usr/bin/mysqladmin -u root password 'puppet'
****/usr/bin/mysqladmin -u root -h puppet.example.com password 'puppet'
/usr/bin/mysql -u root -p**_


I just wanted to clarify this for you in the event my use of <> and ' ' above caused any confusion.

_**Configuring MySQL**_

Once you run the above commands, MySQL will prompt you for the password you just set.  Enter that password, and you will find yourself at a mysql prompt that looks like so:


_**mysql>**_


What this means is you have now logged into the MySQL database, and are ready to set it up for use.  Following I will list out all the commands you need to run in a set.  Note that these commands are each entered on a line and you press "<ENTER>" at the end of the line to enter the next command.  There is no output from MySQL when you enter these, so I'll enumerate them all together here for your convenience.


_**mysql> create database dashboard character set utf8;**_
_**mysql> create user 'dashboard'@'localhost' identified by ‘my_password';**_
_**mysql> create user 'dashboard’@‘<FQDN>' identified by ‘my_password';**_
_**mysql> grant all privileges on dashboard.* to 'dashboard'@'localhost';**_
_**mysql> grant all privileges on dashboard.* to 'dashboard’@‘<FQDN>';**_
_**mysql> drop user ‘’@‘localhost’;**_
_**mysql> drop user ‘’@‘<FQDN>’;**_
_**mysql> drop database test;**_
_**mysql> flush privileges;**_
_**mysql> exit**_


As before, replace <FQDN> with the fully qualified hostname for your server and 'my_password' with the password you wish to set for the dashboard user.  A few notes:



	
  1. First, we created the dashboard database

	
  2. Next, we created the dashboard user for connecting from the localhost name

	
  3. Next, we created the dashboard user for connecting from the server FQDN

	
  4. The next two, we grant the dashboard user rights to the whole database from either location

	
  5. The following two lines delete the user '' from the server (a null user w/o a password)

	
  6. Finally we drop the "test" database, flush all our privilege tables (to take effect immediately) and exit MySQL.


The final steps in getting MySQL configured for production use is to tweak the settings in the database by editing the /etc/my.cnf file and restarting the database.  Open the /etc/my.cnf file and add a new line at the end of the file:


_**max_allowed_packet = 32M**_


Save the file and then run


_**sudo /sbin/service mysqld restart**_


for the changes to take effect.

_**Passenger
**_The final piece is to get the appropriate passenger gems and the Apache module installed to handle Puppet Agent requests to the server.  Luckily, our previous prerequisite installs have made this easy for us.  First:


_**sudo gem install rack passenger**_


When this is done, install the Apache module, following the prompts as follows:


_******sudo passenger-install-apache2-module
Press <ENTER>
Press <ENTER>
At the end of the installation, Press <ENTER>**_


If we've done everything right up until this point, you should not need to supply any extra information, packages, or configuration, and only need to continue to press <ENTER> as listed above to complete the installation.

Now comes the time to configure the various pieces...

_**Configuration**_

_**Passenger
**_First we need a number of directories and files to exist around the system, so let's put those in place by using the following commands:


_**sudo mkdir -p /usr/share/puppet/rack/puppetmasterd
****sudo mkdir /usr/share/puppet/rack/puppetmasterd/public
**_**_sudo mkdir /usr/share/puppet/rack/puppetmasterd/tmp
sudo cp /usr/share/puppet/ext/rack/config.ru /usr/share/puppet/rack/puppetmasterd
sudo chown puppet:puppet /usr/share/puppet/rack/puppetmasterd/config.ru
sudo chown puppet-dashboard:puppet-dashboard /usr/lib/ruby/gems/1.8/gems/passenger-4.0.37/buildout/agents/PassengerWatchdog_**


Next, we need to configure the Puppet Dashboard to connect to its database, and setup the tablespaces for use:


_**cd /usr/share/puppet-dashboard/config
****edit database.yml
Remove the last stanza of this file that refers to the "test" database we removed above.
For the Production and Development database stanzas, change the "database:" line to read "dashboard" and the password line to contain your dashboard password so that it appears as follows:**_




_**database: dashboard**_
_**username: dashboard**_
_**password: <PASSWORD>**_


Next, prepare the database for use as follows:


_**cd /usr/share/puppet-dashboard
**__**sudo rake gems:refresh_specs
****rake RAILS_ENV=production db:migrate**_


_(Even though we've reference the production and development databases in the database config above, we'll only be working in the production database in this tutorial)  _

At this point, we should be ready to test the Dashboard configuration to ensure we're still on the right track.  Top do so, run the following:


_**cd /usr/share/puppet-dashboard
****sudo ./script/server -e production**_


Now, attempt to connect to the dashboard via web browser by pulling up the server at the following address:  http://<IP ADDRESS or FQDN>:3000.  If the dashboard displays correctly in your browser, we're ready to continue.

Press CTRL-C to exit the server.

_**Configure Puppet for Dashboard**_

While we have already configured the dashboard itself, we have not told Puppet the dashboard exists.  To do so, edit the /etc/puppet/puppet.conf file and add the following.

In the [master] section of the puppet.conf, add the following lines:


_******# Reporting**_
_**reports = store,http**_
_**reporturl = http://<FQDN>:3000/reports/upload**_




_**# Node Classification (Using as an ENC)**_
_**node_terminus = exec**_
_**external_nodes = /usr/bin/env PUPPET_DASHBOARD_URL=http://localhost:3000 /usr/share/puppet-dashboard/bin/external_node**_


Exit the puppet.conf file, saving your changes and set permissions for following files like so:


_******sudo chown -R puppet-dashboard:puppet-dashboard /usr/share/puppet-dashboard**_
_**sudo /sbin/chkconfig puppet-dashboard-workers on**_
_**sudo /sbin/service puppet-dashboard-workers start**_


_**Apache**_

Next, we need to configure the Apache web server to process requests being made by Puppet agents in your environment and hand them off to the Puppet server.  To do so, we need to create two files in the /etc/httpd/conf.d location.  The passenger installation will have already created a passenger.conf there.  Just remove it before creating the following two files.


_******/etc/httpd/conf.d/dashboard.conf**_




[snippet id="30"]


And like it:


_**/etc/httpd/conf.d/passenger.conf**_




[snippet id="31"]


 _**Starting Everything Up for Testing & Operation**_

Once these configuration files are in place, it's time to test Apache's handoff to Puppet and to make a special SELinux module to allow Passenger handoffs to the various needed places in the filesystem.

First, make sure the puppetmaster process has been stopped:


_**/sbin/service puppetmaster stop
****/sbin/chckonfig puppetmaster off**_


This assumes you've run the procedures in the previous tutorials, including (especially) the certificate signing and exchange between master and agent.  If you've done this, Passenger now has all the certs it needs to handle requests on behalf of Puppet, and no longer needs the Puppet server running.

Next, test the configuration, that you've made no typos:


_**sudo /sbin/service httpd configtest**_


If no errors are displayed, then _at least _the syntax of your Apache configs are correct.  Now, to generate SELinux entries in the audit log to build a custom Passenger SELinux module, you need to start Apache:


_**sudo /sbin/service httpd start**_


Turn off SELinux temporarily:


_******sudo setenforce 0**_


Restart Apache to generate the log entries:


_**sudo /sbin/service httpd restart**_


Test the Puppet dashboard in a browser by going to:


_**http://<FQDN>:3000**_


What you have done is put output into the audit log that can be used as input to the _audit2allow _tool to generate a policy file for import into SELinux to allow Passenger to do it's job.  To create that module, run:


_**grep httpd /var/log/audit/audit.log | audit2allow -M passenger**_


This creates a new policy file for SELinux in your current working directory called "passenger.pp" which you can now import into SELinux.  TO do so, simply import the module:


_**sudo semodule -i passenger.pp**_


...from the directory where the file resides (presumably your current working directory if you have not moved).

Finally, you re-enable SELinux, and begin to test your environment.


_**sudo setenforce 1**_


_**Conclusion**_

I know that's a lot for a single entry, but I wanted to make sure and get all the additional pieces installed and configured before starting to cover how each of the pieces work and interact.  I thought of making each piece its own article, but in every scenario I thought about, you ended up with a not-completey-configured (i.e., "broken") system.  I opted instead to do the complete configuration.

At this point, however, you should be all set and ready to go with Dashboard/Passenger configured and being run by Apache.  You should have MySQL and PuppetDB installed and configured, handling their own individual tasks, and you should have Puppet Master and Agent installed (with one agent node) all configured to perform their duties as a Puppet Infrastructure.


