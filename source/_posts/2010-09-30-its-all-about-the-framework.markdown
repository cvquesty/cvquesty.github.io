---
author: admin
comments: true
date: 2010-09-30 15:09:26+00:00
layout: post
slug: its-all-about-the-framework
title: It's All About The Framework
wordpress_id: 242
categories:
- Linux
- Systems Administration
- UNIX
tags:
- Linux
- Systems Administration
- UNIX
---

System Administration is a funny thing.

Many people think that it's just adding users, performing requests, and working with project groups to get their "next big thing" out the door.

Certainly, much of your work involves those very things, but what younger admins never quite realize until they've built a beast that cannot be fed is that everything is about frameworks that tie everything together.  From a system load framework to an authentication framework to an asset management framework, if built correctly a framework can save each and every admin countless hours of administration time.  How?

**Automation**

Take distribution.  You can push files around considerably more easily when you've built a framework to do so rather than every admin having an individual way to do it.  In fact, when properly implemented, distribution of a file to "gobs" of similar hosts (that's a technical term there :)) can be as simple as:

**_distrib <object> <class of host> <drop-point>_**

Now, many will say that rsync or scp will do much of this for you, and that is correct.  However, in the context of your individual site, having symbolic abstractions such as "class of host" goes a long way.  Perhaps a certain file needs to be distributed to only your web servers.  Or maybe, only web servers running RedHat 5.3.  If you correctly build a framework for file shove and management, suddenly heavy lifting becomes a light chore.  After all, the more pulleys in the works, the lighter the load becomes.

**Authentication**

As has been covered on this site in the past, LDAP is a wonderful authentication framework that can be tied to positively everything in your environment as well.  From Apache authentication against the store to UNIX authentication, to various types of applications understanding LDAP as a target for authentication sources, much pain of user administration can be solved by having a centralized authentication mechanism.

**Frameworks as Philosophy**

Rather than continue with examples on a case-by-case basis, consider the entire concept of frameworks and unifying ties across systems and networks.  Many places I've worked, organic growth brought about massive numbers of machines that were "siloed" one from another either by project boundaries, function boundaries, or other superimposed logical delineations we as users imposed on them.

Instead of "the DEV servers" or "The PROD servers", we have logically separated them into "the ECommerce DEV servers" versus "the Web DEV servers" and so forth.  Rather than having a framework of systems and their functions in the workplace providing services and features at a certain level to the customer and/or end-user, we cobble together pieces to serve a single purpose rather than modules to expand the greater infrastructure.

**Logical Differentiation and Service-Orientation**

The clear winner over the "as needed" or "for a purpose" way of doing things is the "tiered" or "services-oriented" model of work.  Rather than many groups of things that form a farm of servers, you have the farm of servers that service many different things.  This, I know, sounds something like technological double-speak, but let me explain what I mean.

In a normal environment, we would have a somewhat typical scenario with front-end servers, back-end infrastructure, management, and organization.  The problem we have with this is that if "New Whiz-Bang project #7" comes along, new hardware will need to be procured for each piece of the puzzle.  New app servers, new web servers, new database components, maybe management and authentication considerations... Each project, every time, capital outlay, budgetary justifications, etc.

If, instead, you think in terms of frameworks, your job becomes one of determining total resources needed across an environment rather than resources needed for an individual project, and thus individual project growth concerns, individual project funding concerns, personnel, etc.  "Siloed" growth and expansion may not bite you today or tomorrow, but as I said early on, will become a beast that cannot be fed.

Consider instead, the following example:

Instead of "theses web servers versus those web servers", you instead have "the web layer".  Any and all requests to your company come through "the web layer".  It is scaled as an entity and not as individual projects.  When scaling happens, all environments benefit.

Instead of "these application servers" versus "those application servers", you have "The App Layer".   A single applications framework that serves all application server requests back out the front-end web layer by leveraging container and web server features to do the "effing magic (tm)" on the backend to provide a unified front-facing experience to the user.

Extrapolate these ideas... Instead of the app layer, let's say the app cluster.  Now the power behind this idea becomes clear.  Unlimited scalability with unlimited potential.  How about "the database cluster"?  Regardless of the solution you use, if there is a single database resource (cluster, replication ladder, whatever) that serves back queries you throw at it, how much better is that than "databases for this" and "databases for that"?

Take it a step further.. make an XML services layer that serves out "your data" in a clearly defined API sort of way, and all you do is make XML requests to a services infrastructure rather than directly at your databases.  Or, your cluster is comprised of "write databases" versus "read databases" and you're segmenting the _type_ of traffic you're serving to reading versus writing, making the read operations light-years faster.

Authentication layers, web layers, XML layers, app layers, database layers... All frameworks that grow as an organism rather than series of unrelated growths.

When going through your next design adjustment or your next expansion or data center rollout, consider thinking differently about growth and planning.  I believe that if you think in terms of large organisms with several related parts rather than several growths unrelated except in their location, you'll build into your infrastructure a power and a scalability that will serve you well for many years (and growths!) to come.
