---
layout: post
title: Open Source software and HP-UX
date: 2010-02-05
type: post
published: true
status: publish
categories:
- HP-UX
tags:
- commands
- GNU
- HP-UX
- HP-UX Internet Express
- opensource
author: juan_manuel_rey
comments: true
---

HP-UX is a great OS and the default installation provides a plethora of tools for any Unix sysadmin but of course, like almost every operative system, it can improved with third-party OpenSource software

This is a short list of the packages I can't live without, all of them can be obtained at the [Porting And Archive Centre for HP-UX](http://hpux.connect.org.uk/) or the [HP-UX 11i Internet Express](http://h20338.www2.hp.com/enterprise/w1/en/os/hpux11i-internet-express.html) collection. The packages included in the Internet Express collection are fully supported and tested by HP.

-   **Bash shell**. The best Unix shell, at least for
    me, powerful and highly customizable.
-   **GNU coreutils**. Provides with GNU version of many Unix commands,
    what can I say? I like the GNU *ls* colored output x-)

[![GNU coreutils at work](/images/coreutils1.jpg "GNU coreutils at work")]({{site.url}}/images/coreutils1.jpg)

-   `lsof` How can any sysadmin live without `lsof`?
-   `sudo` Indispensable to allow scripts or users run certain commands as root or any other user.
-   `unzip` It's not that I use it but every time I set up a new server to run Oracle the database people ask for it so I got used to install it.
-   `tcpdump` Don't install it by default on my servers but I like to have the depot around, just in case.
-   `tusc` God bless Chris Bertin!
-   `vim` Not on my production servers but I like to have VI Improved installed on my management node, usually the Ignite-UX server, where I store all my scripts and with ssh (public key authentication) access to all the nodes.

Of course all of them have dependencies that have to be met, however with the exception of vim, which have tons of them, there aren't too much.

And that's all folks, which OpenSource package do you use in your HP-UX boxes? Please comment :-)

Juanma.
