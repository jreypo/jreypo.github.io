---
layout: post
title: Double check! Always!
date: 2010-02-24
type: post
published: true
status: publish
categories:
- HP-UX
- Sysadmin
tags:
- errors
- HP-UX
- NFS
- NFS Toolkit
- ServiceGuard
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

Today I learned the hard way how important is to check everything at least twice.

I had to perform an OS patching on a two node NFS cluster, a couple of [BL860C](http://h18004.www1.hp.com/products/servers/integrity-bl/c-class/860c/index.html?jumpid=reg_R1002_USEN) running HP-UX 11.31 with three packages, one for development, one for testing and the third for the production environment.

Everything was fine until I tried to run one of the packages on its failover node in order to patch its primary, the package didn't start. The log of the package was populated with one error after another, of course I tried to identify the source of the error... nothing, I read the config files line by line... nothing. At that point I started to sweat and then just when a I was thinking to write my resignation letter I noticed it. Somehow an extra damn *equal* slipped into the `hanfs.sh` file and of course the cluster didn't recognize the sharing options and refused to start the package.

In fact the downtime has not been more than a few minutes and fortunately for me not in the production package but the excuses aren't valid, at least for me. I was so confident on my expertise that I made a newbie mistake, probably during a modification of the package after the creation of a new filesystem.

Have to be more careful in the future.

Juanma.
