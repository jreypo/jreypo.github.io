---
layout: post
title: Hardening dilemma
date: 2009-12-14 14:28:11.000000000 +01:00
type: post
published: true
status: publish
categories:
- HP-UX
- Security
tags:
- HP-UX
- Security
author: juan_manuel_rey
---

When you have to secure a system you probably have come to the dilemma 'Which method is the best? Bastille or manual hardening?' at least I did it.

Bastille is a very good option, it will ease the process and you can even use **Install Time Security** mode during the installation of new systems or use the configuration files in an already running system (the files are in `/etc/opt/sec_mgmt/bastille/configs/defaults`), but some time ago I decide that it didn't suit my needs since I like to maintain the control of the whole process.

If you really want to be sure that every corner in your systems is properly secured and monitored it is worthwhile to spend some time studying your severs and the services running and its dependencies.
After the compilation of all that data you can develop a generic security policy and use it as starting-point to customize the security of every server.

In the end of course this is up to you, you must choose whatever suits better your needs.

Juanma.
