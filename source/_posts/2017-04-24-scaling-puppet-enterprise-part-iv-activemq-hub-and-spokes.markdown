---
layout: post
title: "Scaling Puppet Enterprise - Part IV - ActiveMQ Hub and Spokes"
date: 2017-04-24 21:10:52 -0700
comments: true
categories: 
---
***As in the previous installment, you need to have already completed a few steps before arriving at this post.  You should have already completed a "split installation" (Documented [here](http://questy.org/blog/2017/04/18/scaling-puppet-enterprise-part-ii-installation/)). Also, your load balancer needs to be configured and running. The procedure for this portion can be found [here](http://questy.org/blog/2017/04/21/scaling-puppet-enterprise-part-iii-additional-compilers-part-1/).  Finally, you should have the additional compilers installed and configured along with two example agent nodes as covered [here](http://questy.org/blog/2017/04/21/scaling-puppet-enterprise-part-iiia-additional-compilers/) and [here](http://questy.org/blog/2017/04/21/scaling-puppet-enterprise-part-iiib-additional-compilers/).. If you've completed all these portions, you are now ready to configure ActiveMQ for scaling MCollective.***

Once the preceding items are performed, you may find it necessary to add ActiveMQ hubs and spokes to increase capacity for MCollective and/or the Code Sync and Code Manager functions of Puppet Enterprise. This installment documents how to install these additional components and tie them into the existing infrastructure.

###Create an ActiveMQ Hub

1. Go to the Puppet Enterprise Console in your browser.
2. Select **Nodes | Classification** and create a new group called **"PE ActiveMQ Hub"**
3. Stand up two new nodes for the ActiveMQ Hub and Spoke (in our example, **activemq-hub.example.com** and **activemq-spoke.example.com**) according to the following specifications:

![Hub and Spoke Specs](http://cvquesty.github.io/images/hub_spoke_specs.png)

Once your nodes have been provisioned, install the Puppet Agent on each node, **making sure to point the installer DIRECTLY at the MoM (**master.example.com**) instead of at the compiler VIP.

```
curl -k https://master.example.com:8140/packages/current/install.bash | bash
``` 
and let the agent install complete in its entirety.

Next, from your browser, retrieve the Puppet Enterprise Console and select the "**PE ActiveMQ Hub**" group you created earlier. Pin the **activemq-hub.example.com** node into the **PE ActiveMQ Hub** group.

1. Select the "**Classes**" tab and add a new class entitled: "**puppet_enterprise::profile::amq::hub**"
2. Click "**Add Class**".
3. Under the Parameters drop-down, select "**network\_collector\_spoke\_collect\_tag**" and set its value to "**pe-amq-network-connectors-for-activemq-hub.example.com**"
4. Commit the changes.
5. SSH to the **activemq-hub.example.com** and run **puppet agent -t** to make all your changes effective for the Hub node.

###Create ActiveMQ Spoke (or "broker")

1. In the Puppet Enterprise Console, Select **Nodes | Classification | PE ActiveMQ Broker**
2. Pin your new ActiveMQ broker into the **PE ActiveMQ Broker** group.
3. Select the "**Classes**" tab.
4. Under the **puppet\_enterprise::profile::amp::broker** class, choose the **activemq_hubname** parameter and set it to the FQDN of the hub you just created. In our case, **activemq-hub.example.com**.
5. SSH to the new broker (**activemq-spoke.example.com**) and run **puppet agent -t**.
6. Finally, unpin **master.example.com** from the **PE ActiveMQ Broker** group.

###Conclusion

At this point, you should have:

* Puppet Master of Masters - **master.example.com**
* PuppetDB - **puppetdb.example.com**
* PE Console - **console.example.com**
* HAProxy Node - **compiler.example.com**
* 2 Catalog compilers - **compile1.example.com** and **compile2.example.com**
* An ActiveMQ Hub - **activemq-hub.example.com**
* An ActiveMQ Spoke - **activemq-spoke.example.com**
* Two Agent Nodes - **agent1.example.com** and **agent2.example.com**

with their respective configurations. Your serving infrastructure is complete, and you are now ready to configure it for use.