---
layout: post
title: "Scaling Puppet Enterprise - Part II - Installation"
date: 2017-04-18 13:03:21 -0400
comments: true
categories: 
---
Installing Puppet Enterprise has been made remarkably easier as time has gone on. The efforts of Puppet Labs (I still can't get used to simply 'Puppet') to make the installation as seamless and powerful as possible with the simplest of interfaces has been highly successful. 

Many changes have occurred over time to include changing from answer files to a [HOCON](https://docs.puppet.com/pe/latest/config_hocon.html) formatted **pe.conf** file containing the various configuration elements you may need to stand up an instance. I somewhat preferred the simple nature of the original answer files, but I can see the sense in moving to HOCON moving forward.

I will try and cover both GUI and text based installation and relate any "gotchas" or idosyncasies I may have encountered in the past.

**Obtain puppet**

Needless to say, you're going to need the Puppet Enterprise package to install from. Unlike Puppet Community, the entire installer is provided as a tarball rather than repo based installations via package management, and requires a little bit of UNIX-y knowhow to get it started, as the Puppet Enterprise Server is only installable on Linux.

When you navigate to the Puppet Download page, you may be required to sign up for a free account if you haven't already. The opening download page is found [here](https://puppet.com/download-puppet-enterprise).  

You will be presented with a launch page that contains a "Download" button.  Click the button, and one of two things will happen.  Either you will be directed to a "Thank You" page or a page to sign up for an account. As you can see, the "Thank You" page means you already have an account and are signed in whereas the signup page is self-explanatory. Sign up for an account, and retry the download link.

Once you've made it to the "Thank You" page, there are three tabs containing "Puppet Enterprise Masters", "Puppet Enterprise Agents", and "Puppet Enterprise Client Tools". As of this writing, the only supported Puppet Master platforms are RedHat 6 & 7, Ubuntu 12.04, 14.04, 1nd 16.04, as well as SLES 11 and 12.

If you had intentions of running the Puppet Master server on any other platform, here is where you realign your expectations.  :)  I have heard that people have hacked the server to run on other platforms, but since we're dealing with Puppet Enterprise, why would you break support and eliminate warranty?  Pick one of the three and download the tarball for your appropriate platform.

***NOTE:  If You need legacy versions of PE, you can download those [here](https://puppetlabs.com/misc/pe-files/previous-releases).***

**Installation**

For the purposes of this scenario, we will be installing the Puppet Infrastructure for fictional super-mega huge company "example.com". I am going to trust you have worked out the DNS/Host file naming structure, and can resolve everything everywhere.  If you cannot, don't comment on the post, as I will make fun of you publicly... you deserve it.

My assumed setup will be:

| Node Name | IP Address |
| --------- | ---------- |
| master.example.com | 10.0.1.20 |
| console.example.com | 10.0.1.21 |
| puppetdb.example.com | 10.0.1.22 |
| compiler.example.com (VIP) | 10.0.1.23 |
| compiler1.example.com | 10.0.1.24 |
| compiler2.example.com | 10.0.1.25 |
| amqhub.example.com | 10.0.1.26 |
| amqspoke.example.com | 10.0.1.27 |
| agent1.example.com | 10.0.1.28 |
| agent2.example.com | 10.0.1.29 |

***Automated***

The Puppet Enterprise Installer is a GUI web-browser based installer. Puppet has gone through the process of giving you a nice frontend to your installation, and making it dead-easy to perform a monolithic as well as split installation.  For our purposes, though, we will be doing a "split" installation.

**Stand up 3 Nodes with the specifications from the first article in the series as follows:**

| Node | Image | IP Address |
| ---- | ----- | ---------- |
| master.example.com | m3 or m4.xlarge | 10.0.1.20 |
| console.example.com | m3 or m4.large | 10.0.1.21 |
| puppetdb.example.com | m3.2xlarge | 10.0.1.22 |

In my experience, I've found it much easier to exchange root keys between all three of the above nodes to allow the installer to do all it needs to do on each node. You can, however, decide to set the root password to something temporary to hand to the installer as well (and many people opt for this) and then return root's password to your site default.  In any event, all the machines should be able to resolve themselves and each other by name and root should be able to freely ssh between them either via shared keys (easiest) or password.

**Transfer the package to the Puppet Master node:**

```
scp -rp puppet-enterprise-2015.3.2-el-7-x86_64.tar.gz root@master.example.com:/root/
```

Once the package is on the destination machine, you should connect to the machine to work with the package on-box:

```
ssh root@master.example.com
```

which places you in the root user's home directory where you copied the package.

**Extract the Package**

```
tar -zxvf puppet-enterprise-2015.3.2-el-7-x86_64.tar.gz 
cd puppet-enterprise-2015.3.2-el-7-x86_64
```

**Run the Installer**

```
./puppet-enterprise-installer
```

You will receive a text prompt that states:

```
??Install packages and guided install [Y,n]
```

Simply press "Y" or the [Enter] key and the GUI portion of the installation will begin.

***GUI Installer***

Once you have started the Installation, the Puppet Enterprise Installer will perform some preparatory steps and then launch an installation interface on your master node on port 3000. To access this interface, you can bring it up in the web browser of your choice at:

```
https://master.example.com:3000
```

Navigate to this interface in your Internet browser. When you first arrive at the GUI installer, simply click the "Let's Get Started" button. On the next page, select "Split" to begin the Split Installation.The Puppet Enterprise Installer will present you with a GUI questionnaire to fill out regarding your environment. The following is that process in order by section.

## Puppet Master Component

1. Choose the "Install on this Server" radio button.2. Enter the name of your Master in the **Puppet Master FQDN** text box. (e.g. master.example.com) 3. Enter all appropriate names for your master in the **Puppet Master DNS Aliases** text box.4. Select the "Enable Application orchestration" check box.## PuppetDB Component

1. Enter the hostname of your PuppetDB Node in the **PuppetDB Hostname** text box. (e.g. puppetdb.example.com) 
2. Change no other selections under the remainder of the items for this section.

## PE Console Component

1. Enter the hostname of your Puppet Console in the Console Hostname text box. (e.g. console.example.com) 
2. Change no other selections under the remainder of the items for this section.

### Database Support

No changes are needed to is section. Simply leave "Install PostgreSQL on the PuppetDB host for me" selected.

### Console 'admin' User

Enter the password you would like to use for the Puppet Enterprise console once your installation is complete in the final text box.

## Final Considerations

After completing the final section, click the "Submit" button, and the Puppet Enterprise Installer will present you with a confirmation page for you to review before commencing the installation based on the configuration elements you just provided to the installer.

If everything is to your satisfaction, click the "Continue" button and the Puppet Enterprise Installation will begin.The Installation progress summary will continue to update you as to the progress of the installation. If you would like to see logging "as it happens", you can click the "log view" button to see that in real time. If you would like to switch back to the summary, simply click the **Summary** button.After what is roughly 10-15 minutes of installation and configuration, the installer will have completed all its work, and you will be presented with a button at the bottom of the progress screen you have been viewing that says: "Start Using Puppet Enterprise".  Click that button, and the installer will redirect you to the PE Console login screen.  Enter the admin credentials you created earlier, and you are ready to begin working with the console as needed.