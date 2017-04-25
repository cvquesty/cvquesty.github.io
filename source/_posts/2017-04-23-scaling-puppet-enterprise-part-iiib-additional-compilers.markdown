---
layout: post
title: "Scaling Puppet Enterprise - Part IIIb - Additional Compilers"
date: 2017-04-23 18:19:42 -0400
comments: true
categories:
---
***As in the previous installment, you need to have already completed a few steps before arriving at this post.  You should have already completed a "split installation" (Documented [here](http://questy.org/blog/2017/04/18/scaling-puppet-enterprise-part-ii-installation/)). Also, your load balancer needs to be configured and running. The procedure for this portion can be found [here](http://questy.org/blog/2017/04/21/scaling-puppet-enterprise-part-iii-additional-compilers-part-1/).  If you've completed all these portions, you are now ready to configure and install the compilers themselves. If this is you, read on!***

---

Once your Load Balancer and split install are in place and functioning, we need to add more compilers to the serving infrastructure. For the purposes of this tutorial, we will install two additional catalog compilers, register them with the currently existing master. Then, we will direct them to look to the "MoM" or the "Master of Masters" as the CA certificate authority.  Further, we will install two agent nodes and connect them to the infrastructure.

You will need to install two compiler nodes and two agent nodes according to the following specifications.

![Compilers and Agents](http://cvquesty.github.io/images/compiler_and_agent_specs.png)

Once these four nodes are in place, we can connect the compilers to the Master of Masters (MoM) and then the agents to the "master" as they see it.  Remember, that for our purposes, these nodes are named:

* compile1.example.com
* compile2.example.com
* agent1.example.com
* agent2.example.com


###Installing the Compilers

SSH to the first compiler master (**compile1.example.com** for this post's purposes) and install the Puppet agent as follows:


```
curl -k https://master.example.com:8140/packages/current/install.bash | bash -s main:dns_alt_names=compile1.example.com,compile.example.com,compile1,compile
```

What this does is simple. When this compiler goes behind the load balancer (**compile.example.com**), traffic may get directed to this node. When the request is made, the agent node will be asking for "**compile.example.com**" but this node's name is "**compile1.example.com**". The additional options at the end of the curl line are to tell the agent that when it installs, it should be aware of both names, and when speaking to the MoM the first time to request its cert, to represent all the comma delimited names listed at the end of the above command.

Next, SSH to your master node and accept the agent cert request as follows to allow for these names on the MoM you just set up in the previous step.

```
ssh master.example.com
puppet cert --allow-dns-alt-names sign compile1.example.com

```

####NOTE THAT YOU CANNOT ACCEPT THIS CERT FROM THE CONSOLE. ALT_DNS IS NOT SUPPORTED FROM THE GUI

Finally, run the puppet agent on the first compiler _(compile1.example.com)_ to configure the node:

```
puppet agent -t
```

Once the agent run is complete, you need to classify the catalog compiler in the console to make it ready for service.

###Classify the Compiler

In the Puppet Enterprise Console:

Choose **Nodes | Classification | PE Master**

Add **compile1.example.com** and pin the node to the classification group and commit the change.

*****BE SURE TO COMPLETE THE NEXT STEPS IN ORDER AS FOLLOWS OR YOU WILL HAVE A BAD TIME*****

First: SSH to **compile1.example.com** and run **_puppet agent -t_**

Second: SSH to **puppetdb.example.com** and run **_puppet agent -t_**

Third: SSH to **console.example.com** and run **_puppet agent -t_**

Fourth: SSH to **master.example.com** and run **_puppet agent -t_**

*****BE SURE TO ALLOW EACH RUN TO COMPLETE _FULLY_ BEFORE MOVING ON TO THE NEXT ONE*****

###For All Subsequent Compile Node Installations

Follow the above instructions completed for **compile1.example.com** for all subsequent compiler installations. This means that if you add compilers six months or a year from now, go back to the previous procedure and duplicate it with the new node name precisely as you did above.  To recap:

1. Install the agent as above with the alt_dns switches
2. Accept the cert on the master with the alt_dns switches
3. Classify the compiler in the console
4. Run the Puppet agent in the above specified order, allowing each one to complete fully before moving on.

###Configure Future Agent Installations to Point to the Load Balancer by Default

In the Puppet Enterprise Console, you must configure the system to point all future agent installations to the load balancer by default so you do not have to continue to make modifications and customizations after each agent install. To do so, perform the following steps:

1. In the Puppet Enterprise Console, choose: **Nodes | Classification | PE Master**
2. Select the "**Classes**" tab.
3. Choose the "**pe_repo**" class.
4. Under the parameters drop-down, choose "master" and set the text box to the name of your load balancer or VIP _(in our case, **"compile.example.com"**)_
5. Commit the changes.

###Point the New Compilers at the Master (MoM) for CA Authority

####Create a new classification group called "PE CA pe_repo Override"

* Go to **Nodes | Classification** in the Puppet Enterprise Console
* Create a New Group
* Name the new group "**PE CA pe_repo Override**"
* From the "Parent Name" drop-down, choose the "**PE Master**" group.
* Click "Add Group".

1. Select your new group and pin **master.example.com** to the new group and click "**Commit One Change**"
2. Select the "**Classes**" tab.
3. Add "**pe_repo**" class.
4. From the parameter drop-down, select "**Master**".
5. Enter the name of the MoM in the text box. _(in this example, **master.example.com**)_
6. Click "Add Parameter" and then "Commit 2 Changes".

####Test New Agents

The two agents you created at the beginning of the article are now able to be tested with this new group of compilers.

1. Make sure **agent1.example.com** and **agent2.example.com** have been installed according to the system requirements covered in this series.
2. Install the Puppet agent on each of these nodes, but this time instead of pointing at the MoM, point to your Load balancer vip like so:

```
curl -k https://compile.example.com:8140/packages/current/install.bash | bash
```

In the PE Console, accept the new certificate request for Agent1.  SSH to **agent1.example.com** and run _**puppet agent -t**_.  Finally, repeat this process for **agent2.example.com**.

If you have completed all the above steps properly, the agents will reach out to the **compile.example.com** VIP and be ferried off to one of the catalog compilers. Regardless of which one, since we accepted all the alternate DNS names when creating the connection between them and the MoM, they will respond for **compile.example.com**, and deliver back to the agent the required information, catalog, etc. as Puppet would do under normal circumstances.

###Conclusion

As you can see, we needed the Load Balancer in place to install the catalog compilers. We also needed all the DNS alt-naming to be in place so the load balancer could send traffic to either catalog compiler as needed, and still have it answer for the VIP name. Finally, we needed to refer requests to their appropriate destinations and also classify the new compilers as such with the MoM, and set up appropriate referral of certificate requests from the compilers back to the CA Master, which is the MoM.

The serving infrstructure is almost done, all we have left to do is to scale MCollective with an ActiveMQ Hub & Spoke, and remove that responsibility from the MoM.  We will also install a GitLab server to hold our Control Repo and associated Roles & Profiles, and we will configure the Code Manager.
