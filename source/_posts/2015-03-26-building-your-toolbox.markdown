---
layout: post
title: "Building Your Toolbox"
date: 2015-03-26 09:15:18 -0400
comments: true
categories: geekstuff,linux,puppet
---
I know it’s been since June we’ve worked on the Puppet Development series, but as work goes, so go I, and as I go, so delays the blog. :)

**Recap**

We started our journey with a reintroduction to Vim. As strange as it sounds, often times we techno-guys take for granted that people coming into the DEVOPS space are well versed in all these things, and overlook remediating the basics. We covered Vim basics and switching between command and insert mode as well as linking you to some good cheat sheets to help you beef up your vim-fu.

Next, we covered Vim plugins for syntax highlighting and just general code visualization so we can see visually when our code editing has issues and needs to be fixed.

The next article centered on revision control in general, but Git in particular. In addition to using Git, we also covered that amazing tool GitHub and how to get registered for it, create your own repos, and how to work with repos from your local command line as well and I linked you some excellent resources on Git to expand your knowledge of the Git world and become proficient and fluent in its use.

Finally, we got around to Vagrant and I gave you a simple tour of Vagrant to be familiar with the tool and what it does for you. We stood up a Vagrant instance and saw how we could destroy and re-provision that precise same instance with only a shell command, and saw the power of automated provisioning at work right on our own node.

**Reasons**

Like the social media world will tell you, I did it because reasons. In short, I wanted you familiar with all these separate tools as we start to coalesce them into an integrated whole we can use as our development toolbox.

So, since we’ve got Vim, Git, and Vagrant as needed components, what else might we need to continue pressing forward?

We need to understand ***Puppet*** itself.

**Puppet - The Product**

Puppet, as I’m sure many of you are aware, is simply “configuration management software” produced by Puppet Labs, Inc. Puppet Labs was founded in 2005 by then Systems Administrator/Engineer Luke Kaines to help automate common repetetive tasks encountered in his regular work duties encoutered on a day-to-day basis.

After a few rounds of venture funding and explosive growth of the market segment known as “configuration management”, Puppet Labs has become a market leader in the space, and continues to develop and improve upon the product at a rather aggressive rate.

**Configuration Management**

If Puppet is “configuration management” software, what is this thing called “configuration management”?

Configuration management as a systems engineering process covers a lot of landscape in its purview. It can mean, speaking generally, a process for maintaining consistency of a product’s performance and can become considerably complex, such as the methodology used to manage miliary weapons systems, IT service management, and other domain models covering civil and industrial engineering.

For the purposes of the technical field of Systems Administration, Engineering, and Automation, however, Configuration Management as a discipline is very well defined. Specifically, the model that covers these areas is Operating System Configuration Management. Certainly, when automating your site you step over into additional disciplines, but at its core, Configuration Management almost always implies that you are working with the primary target of Operating Systems Configuration Management.

**IT Automation**

Often times, configuraiton management as it is expressed in the realm of operating systems more specifically begins to take on automation as the primary characteristic of the work performed. From provisioning to deployment, automation saves the most time and effort through modeling systems design in a modular fashion, whereby allowing you to apply systems configurations against classes of machines tooled to perform specific types of work.

This paradigm allows for many idioms to be used in the description of the destination machines, and is the primary domain within which products like Puppet operate.

**Puppet – the Product**

As you peruse the main website for Puppet Labs, you start to see a much larger emphasis and prevalence of “IT Automation” throughout the website–specifically in the area of data center automation. It is important to note that Puppet prefers to work within this space, although it has abilities that stretch into the entire IT lifecycle workflow.

Puppet is comprised of two distinct products: “Puppet Enterprise” and “Open Source Puppet”. As is common in the Open Source space, the distinction between the two is defined primarily by way of the support options available for each.

**Puppet Enterprise**

The Puppet Labs flagship product is Puppet Enterprise. Puppet enterprise is a fully supported, maintained, and actively developed data-center ready software package designed to enable you to model configurations for your site out-of-the-box. It has an integrated installer, smoothing the installation process, a series of Puppet apps available for use in the Enterprise Console (the Puppet GUI), and for-pay support and licensing options to meet the needs of your enterprise, regardless of size.

**Open Source Puppet**

Open Source Puppet is the core product found within Puppet Enterprise. It is the engine which drives the Puppet product and does the job of configuration modeling against your environment. It has several components, individually installed, and no shipped console. It generally leads the Enterprise version by several revisions, allowing early access to new features and benefits but lacks the additional features afforded by the Enterprise offering.

Make no mistake. Both products are the same software. However, the Enterprise product has much of the legwork done for you in the integrated installer, additional functionality in independently released “Puppet Apps”, and of course, enterprise-level 24x7 support availability. Add to that a vibrant community of development and expansion, the Puppet Forge, an annual conference, and a respected certification program, and Puppet Labs' offerings, while similar, certainly shine on the Enterprise side of things.

**Idempotency**

The main concept upon which Puppet’s operation is founded is that of idempotency. Idempotence is the property of certain operations in mathematics and computer science that can be applied multiple times without changing the result beyond the initial application.

![Idempotency](http://cvquesty.github.io/images/idempotency.png)

As you can see, this characteristic only applies an effect on the target if and only if it has not already been applied. Subsequent applications will have no effect, as the desired state of the target is as it should be.

This is good news in that it allows us to think of our enviromnet in new ways. Instead of thinking of all the changes we’re making or going to make to our environment, we begin to considered the desired state of our site and the maintenance of that target at all times. Then, “events” are no longer misconfigurations, but instances of deviation from the desired state, the “norm”.

**Implications**

Think of the ramifications of the shift in mindset this represents. Audit reports are no longer a series of weeks of review of the site. Your site’s state is described in code, and is a public record within your organization for all to see. Instead, those conversations become “We are ALWAYS compliant. Here is a report of the few times we weren’t compliant in the last year.”

Considerably different conversation to have.

Now, as you model more configuration state in code, your entire way you think of your site evolves. More things hapen automatically, are automatically reported on, and are automatically remediated. Your reporting changes. Your audits change. Your compliance reporting changes. In fact, your entire internal culture chages, and all involved teams need to make adjustments in the way they think about the site and how to mold the way they’ve traditionally done their job (whether administration, audit, or compliance) into this new world of DEVOPS, automation, and the like.

**Conclusion**

Puppet Enterprise and Open Source promise to not only change how you view your systems and site, but how your entire organization functions. From automated configuration expression within the site, to how you originally model your configuration target, the tools provided by Puppet have changed the face of IT. Automation and configuration management bring culture change and ideological evolution to the enterprise, and step you into the next level of efficiency, compliance, and security management.