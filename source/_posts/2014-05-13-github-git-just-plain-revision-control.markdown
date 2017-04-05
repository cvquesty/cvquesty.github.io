---
author: admin
comments: true
date: 2014-05-13 21:06:09+00:00
layout: post
slug: github-git-just-plain-revision-control
title: GitHub, Git, and Just Plain Revision Control
wordpress_id: 697
categories:
- Linux
- Puppet
- Systems Administration
tags:
- Git
- Linux
- Puppet
---

One of the "bugaboos" in the sysadmin world for the longest time was the reluctance to use those "stinky developer tools" in our world for any reason.  I'm not sure the impetus behind this, but my wager is on something akin to security or yet another open port or "attack vector" if you will.  But today's competent and conscientious systems admin (not to mention DEVOPS person) will use revision control as their go-to standard for collecting, versioning, backing up, and distributing all manner of things.

I've seen some shops use [CVS](http://www.nongnu.org/cvs/) as their choice, old thought it is, just as a large "bucket" in which to throw things for safekeeping with revisions and rollbacks available in case of some uncertain as yet unencountered event.  [Subversion](http://subversion.apache.org) was the next generation of revision control tools.  Darling of developers and bane of disk space, Subversion still had many more features and performed essentially the same task.

Now, [Git](http://git-scm.com) is the flavor of the month, and not only has gained widespread acceptance as a standard way to "do" revision control, it's the de-facto way to do DEVOPS in  a Puppet world.  Granted, there are those brave souls out there who have tried to stick with the older tools, but the workflow and the "glue" between all the various components therein.  Hence, this post.

**What is Git, really?**

Git was developed by [Linus Torvalds](https://en.wikipedia.org/wiki/Linus_Torvalds) for Linux Kernel collaboration.  He needed a new revision control system akin to the previously used [BitKeeper](https://en.wikipedia.org/wiki/BitKeeper) software that was unencumbered by copyright and able to handle the unique distributed development needs of the Linux project.  So, rather than try and use someone else's project, he collated what was needed and developed the project himself.

Now, Git is used both privately and Publicly throughout the world for many projects.  Git is lightweight and works in a more efficient manner by moving changes via diffs rather than whole repositories, allows developers to maintain and manage an entire repository on their own systems either connected or disconnected from the Internet.  Then, they can "push" all their changes back to the central repository as needed.

**Enter GitHub**

For our purposes, we'll specifically be working in [GitHub](https://en.wikipedia.org/wiki/GitHub).  GitHub is a project offering web-based hosting of your code that you can source from anywhere.  GitHub offers public and private hosting and a spate of other related services to development collaboration on the Internet.  If you do not have a GitHub account, you'll need to surf on over to the site and sign up for one.  It's free and it's fast, and I'll be using and sourcing it heavily as this series continues.

**Basic Git**

Git itself is available on most modern platforms and can easily hook into GitHub for our purposes.  I will be mostly referring to command-line usage of git, but you will find quite a bit in the way of tools, frontends, and "helper" apps for Git that you may or may not wish to leverage as you learn and incorporate Git into your workflow.  In the meantime, stick with me on command-line work.

When you install git on your unix-like platform, it will drop a few binaries.  The one we're most interested in is the git binary itself.  It's very simply designed and has a very straightforward set of options you can get from the command line by simply typing "git" with no options, or "git help".  The output is below:

```
usage: git [--version] [--help] [-C <path>] [-c name=value]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]

These are common Git commands used in various situations:

start a working area (see also: git help tutorial)
   clone      Clone a repository into a new directory
   init       Create an empty Git repository or reinitialize an existing one

work on the current change (see also: git help everyday)
   add        Add file contents to the index
   mv         Move or rename a file, a directory, or a symlink
   reset      Reset current HEAD to the specified state
   rm         Remove files from the working tree and from the index

examine the history and state (see also: git help revisions)
   bisect     Use binary search to find the commit that introduced a bug
   grep       Print lines matching a pattern
   log        Show commit logs
   show       Show various types of objects
   status     Show the working tree status

grow, mark and tweak your common history
   branch     List, create, or delete branches
   checkout   Switch branches or restore working tree files
   commit     Record changes to the repository
   diff       Show changes between commits, commit and working tree, etc
   merge      Join two or more development histories together
   rebase     Reapply commits on top of another base tip
   tag        Create, list, delete or verify a tag object signed with GPG

collaborate (see also: git help workflows)
   fetch      Download objects and refs from another repository
   pull       Fetch from and integrate with another repository or a local branch
   push       Update remote refs along with associated objects

'git help -a' and 'git help -g' list available subcommands and some
concept guides. See 'git help <command>' or 'git help <concept>'
to read about a specific subcommand or concept.
```


We're most interested in a small subset of commands for our purposes here.  They are **add, commit, pull, push, branch, checkout, and clone.**




I will be referencing one particular way to "do" git which works for me, but as with anything [TMTOWTDI](https://en.wikipedia.org/wiki/TMTOWTDI) and [YMMV](https://en.wiktionary.org/wiki/YMMV).




**GitHub Portion**




I am going to assume you've created a GitHub Account.  When you create your account, you'll have a unique URL assigned to you based on your username.  Mine, for instance, is https://github.com/cvquesty/<insert project name here>.  The basic interface to GitHub is rather straightforward and looks like the following:




[![mygithub](http://questy.org/wp-content/uploads/2014/05/mygithub.png)](http://questy.org/wp-content/uploads/2014/05/mygithub.png)The interface keeps track of all projects you're working on, the frequency with which you commit or otherwise use your repository, and (most importantly), a centralized server that is storing those projects you can source from any internet connected system.




**Make a Repository**




In the upper right-hand corner of your screen, you'll notice a "+" symbol.  Let's click that and create us a new repository.  You'll be presented with a dialog to name and describe your new repo.  I'll use the name "sample repo" and the description "Sample Repo for my Tutorial" with no other options other than the defaults.  (we'll go manually through those processes shortly).  After creating the repository by clicking "Create Repository", I'm presented with a page that has step-by-step instructions on what to do next.  I'll include that here for you.




[![myrepo](http://questy.org/wp-content/uploads/2014/05/myrepo.png)](http://questy.org/wp-content/uploads/2014/05/myrepo.png)




As you can see, you have your repo referenced at the top by <userid>/<reponame>.  You have instructions on how to use the repo from both the GitHub desktop client and the command line and some special instructions for if you have it locally on your system, and are just now uploading that content into this repository you've created to hold it.  We're interested in the command line instructions.




**A Place to Git**




On my system (a Mac), I have Git installed by default and I have a directory in my home directory simply called "Projects".  Under there, I have a "Git" directory.  ALL of my work in Git goes here.  This is not a hard/fast rule.  I just chose it as my location to place all my git work so it is centralized and all collected together.




What we're going to do next is to configure Git, create a location for our repo, make a file to commit to the repo and then push that file up to GitHub to see how that workflow works.  Let's get started.




**Configuring Git**




Since Git is personal to you as a user, you need to let Git know a few things about you.  This gives your git server (in our case GitHub) the information it needs when you're pushing code (like your identity, default commit locations, etc).  First, your name and email:




git config --global user.name "John Doe"
git config --global user.email you@yourmail.com




You'll only need to perform this once.  There are quite a few options and you can read up on those [here](http://git-scm.com/book/en/Getting-Started-First-Time-Git-Setup) at your leisure.




Next, create a location for your new repo.  I chose my aforementioned directory and created the location




/Users/jsheets/Projects/Git/samplerepo




for demonstration purposes.  From here, though, we can take up with the instructions on the GitHub page displayed after creating your repo. I'll reproduce that here for reference:




cd /Users/jsheets/Projects/Git/samplerepo
touch README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/cvquesty/samplerepo.git
git push -u origin master




If all has gone well, you have now created an empty README.md file, committed it to your local Git repository and then subsequently pushed it up to GitHub.  You'll note that we added "origin" as the remote and then we pushed to "origin" in a thing called "master".  What's that all about?




GitHub (and git) refer to their repo location as "origin".  This becomes handy when you start pushing between remote repositories and from remote to remote to GitHub, etc.  So, it makes sense to name GitHub _functionally _as well as it's assigned domain name.  By saying "origin", we're making GitHub the de-facto standard center of everything we're doing.




Next, we refer to "master".  What is that?  Simply stated, we're pushing to a "branch" called "master".




**Branching**




Branching is a method by which you can have multiple code "branches" or "threads" in existence simultaneously, and Git is managing them all for you.  For instance, you may wish to have one code collection only for use in production systems while maintaining a separate one for development systems.  In fact, you can create a random branch with a bug name (bug1234, for instance), commit your changes to that, test it, and push it to origin, then pull it down to all your production hosts, solving a big problem in your site or codebase.  Better yet, if it all works great and you're happy with it, you can "merge" that bug back into your main code repository, making it a permanent fixture in your code in whatever branch you like. (or even all of them!)




When you first create your repo, GitHub makes a "main" branch for you automatically, and calls it "master".  So, by utilizing the command above, we're telling Git to push our code (in this case, README.md) to our origin server (GitHub) and put it in the "master" branch.




While we're on the topic, let's create two more branches so we can get the full hang of this branching thing.  (Hint:  It's core to how we integrate this into Puppet).




**Makin' Branches**




As I said before, GitHub creates a default "master" branch for you.  If, from your local repository location, you type "git branch", Git will list a single branch for you,




git branch
* master




This tells us simply, what branch you are currently in.  Now, let's run two commands to create new branches to be tracked by Git.




git branch production
git branch development




Now.  Run "git branch" again:




git branch
development
* master
production




As you can see, your other branches are now visible when running the command.  If you have color, you may notice that the "master" branch is a different color than the others (based on your settings).  If you do not have color, the asterisk denotes what branch is active as well.




**Checkout and Commit, Branch and Merge**




We have our repository and we have our branches.  We have a single README.md in the current directory, and we are ready to roll committing code and pushing it into our repository.  Let's perform a simple experiment to get the "hang" of how the branches work and how to switch between them as needed.  Since we're in "master", let's edit our README.md to reflect that by placing a single word in the file "master". (use vim as discussed in our last tutorial).




Once you're done with your edit, you'll see that the text is in the file.  you can edit it and you can cat the file and see the contents, but if you view the file up at GitHub, that content is not there yet.  Some sort of way, a mechanism must be used to put that data there.  Well, there is such a process, and it is a two part process.




Recall I mentioned that one of the features of Git is that you can have a complete repository local to your machine.  you can work on that repo and make all sorts of changes completely disconnected from your server (in our case GitHub... "origin" as it is named to Git).  Therefore, in reality you are dealing with not one, but two repositories.  The local one on your machine and the remote one at origin.  (remember the "git remote add origin" above?)




So, to finalize your changes locally, you must "commit" them to your local repository as "final".  THEN, you can "push" those changes into your main server (in our case GitHub).   We did as much above with our procedure where we did the commit with a message, and then a push up to origin.  However, now that we've made changes locally, they are not yet reflected at GitHub.  Logic would dictate another commit is in order:




git commit
or
git commit -m 'Some message about your commit'




As you can see, there are two routes you can go.  If you simply supply the "git commit" without any options, you will be brought into the system text editor you (or your OS) has configured in the $EDITOR environment variable.  Most platforms use "vi" or "vim" for this, but I have also seen "pico" used in some distributions like Ubuntu Linux.  In any event, you can edit the file by placing your comments in.  After exiting the file, saving the content, the commit will be complete.  If, however, you do not put anything, git will not commit the changes.  This is to enforce good coding practice by requiring some notes about what a committer is doing before making the changes.  It's a highly recommended workflow to follow.




Once your commit is complete, phase 1 (local commit) is over.  You can commit over and over, as many times as you like.  you are a full, local repository.  In fact, I'd encourage many commits.  Commit when you think about it.  Commit before you walk away from your system.  Commit randomly for no reason in mid-workflow.  The more commits you have, the less likely you are to lose work.




Finally, to get the data up to GitHub, we need to "push" that data off your repository and into your "origin" repository. This is quite simple, and you've done it before:




git push -u origin master




Sometimes you may wish to not keep specifying the location you're pushing to.  If so, you can set a default location for each branch.  Git will tell you just how to do that if you forget the "-u location branch" option.  Let's say I'm in my aster branch and I simply run a "git push".  Git will tell me I did something wrong, but will _also _tell me how to eliminate that problem:




fatal: The current branch master has no upstream branch.
To push the current branch and set the remote as upstream, use




git push --set-upstream origin master


"fatal" seems a _little _melodramatic since Git gives you the answer as to what to do right there.  All you need to do is set the default target once with that last line, and from that point forward, you only need type "git push" when pushing to GitHub.  Hint:  I do this in ALL my branches at create time.  It saves a lot of typing over time, and like any good Sysadmin, I'm lazy.  :)

So, now I've got multiple branches that need this setting, but I'm still stuck in "master".  How do I get to "development" or "production" to perform the same tasks?

Git provides a "checkout" command.  What you're saying with "checkout" is:  "Git, I want to be working on branch "x", and I want you to make that my current branch.  if there are any differences between that branch and the one I'm on, please make those changes on-disk for me so I can exclusively be working in branch "x"".  A little verbose, but you get the point.  So, to move to the next branch and do all the wonderful things we did in "master" above, we perform:


git checkout development
edit README.md to say different text
git commit -a -m 'editing README for development branch'
git push --set-upstream origin development
git push


If all has gone well, your development README.md file is now changed and pushed into GitHub.  What about "master", though?  Well, let's take a look:


git checkout master
cat README.md


If all has gone well, the contents of README.md are back to what was in your "master" branch.  By checking out "development", it'll change back to the new content there.  As a test, checkout the "production" branch, change the README.md file, commit it, set your upstream push target and then push the contents to GitHub.

Now you're cooking with gas.

**Conclusion**

This is a simple tutorial to get you started with Git & GitHub.  There are MANY tutorials and books that can make you into a Git expert, but are way outside the scope of this humble little blog.  let me provide a few of those for you here:

[Git Help](http://git-scm.com/documentation)
[Git Book](http://git-scm.com/book)[
GitHub Help](https://help.github.com)

This documentation should be more than enough to get you moving and well underway with Git ins-and-outs for committing Puppet code and using r10k to interface with and distribute that code around your environment.
