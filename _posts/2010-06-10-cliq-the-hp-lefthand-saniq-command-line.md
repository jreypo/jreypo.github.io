---
title: CLIQ – The HP Lefthand SAN/iQ command-line
date: 2010-06-10
type: post
classes: wide
published: true
status: publish
categories:
- HP
- Storage
tags:
- CLIQ
- HP Lefthand
- iSCSI
- P4000 VSA
- SAN/iQ
- Storage
author: juan_manuel_rey
comments: true
---

It seems that a very popular post, if not the most popular one, is the one about my [first experiences with the P4000 virtual storage appliance]({% post_url 2010-04-09-first-hands-on-with-the-hp-lefthand-p4000-vsa %}) and because of that I decided to go deeper inside the VSA and the p4000 series and write about it.

The first post of this series is going to be about one of the less known features of the P4000 series, its command line known as CLIQ.

Typically any Sysadmin would configure and manage P4000 storage nodes through the HP Lefthand Centralized Management Console graphical interface. But there is another way to do it, the SAN/iQ software has a very powerful command line that allow you to perform almost any task as in the CMC.

The command line can be accessed through two ways, remotely from any windows machine or via SSH.

## SSH access

To access the CLIQ via SSH the connection has to be done to the port `tcp/16022` instead of the SSH standard port and with the management group administration user. The connection can be established to any storage node of the group, the operation will apply to the whole group.

[![CLIQ SSH access](/assets/images/cliq_ssh1.jpg "CLIQ SSH access")]({{site.url}}/assets/images/cliq_ssh1.jpg)

## Remote access

The other way to use the CLIQ shell is from a windows host with the **HP Lefthand CLI shell* installed on it. The software is included in the SAN/iQ Management Software DVD can be obtained along with other tools and documentation for the P4000 series in the following URL: <http://www.hp.com/go/p4000downloads>.

Regarding the use of the CLIQ there is one main difference with the On-Node CLIQ, every command must include the address or DNS name of the storage node where the task is going to be performed and at least the username. The password can also be included but for security reasons is best to don't do it and be prompted. An encrypted key file with the necessary credentials can be used instead if you don't want to use the username and password parameters within the command.

[![](/assets/images/cliq_local.jpg "CLIQ remote shell")]({{site.utl}}/assets/images/cliq_local.jpg)

Of course this kind of access is perfect for scripting and automate some tasks.

Juanma.
