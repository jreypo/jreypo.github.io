---
layout: post
title: HP-UX security resources
date: 2009-12-10 14:04:04.000000000 +01:00
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

Recently a friend asked me about HP-UX security and where to find useful information. We have to admit it, there are not many resources out there about HP-UX security and the great majority of them are obsolete since they are about HP-UX 10.20 or even 9.x. Let's take a look...

[HP Docs](http://docs.hp.com/) is the first place to look for information, there you will find a lot of docs regarding HP-UX security, [IPFilter](http://docs.hp.com/en/internet.html#IPFilter), [HP-UX Bastille](http://docs.hp.com/en/5992-5099A/index.html) and other products and manuals concerning [security](http://docs.hp.com/en/internet.html#IPFilter). Following is a reference of useful docs that can be found on this site:

-   The most up to date document is [**HP-UX System Administrator's Guide: Security Management**](http://docs.hp.com/en/5992-6416/5992-6416.pdf), this is the main reference for any HP-UX admin. It covers HP-UX 11iv3 and is filled with detailed information on how-to protect your system, how is the security implemented in HP-UX and an appendix with references to other security products of HP that can be used to hardening your systems.
-   [HP-UX 11i Security Containment Administrator's Guide](http://docs.hp.com/en/5991-8678/index.html) HP-UX 11.23
-   [HP9000 Computer Systems: Administering Your HP-UX Trusted System](http://docs.hp.com/en/B2355-90121/index.html). Useful information concerning older systems.
-   [HP-UX System Administration Tasks: HP9000](http://docs.hp.com/en/B2355-90672/index.html). It has full chapter about system security, useful for older systems.
-   [Managing Systems and Workgroups: A Guide for HP-UX System Administrators](http://docs.hp.com/en/B2355-90950/B2355-90950.pdf). HP-UX 11iv1 and 11iv2 information.

Second in our small list is the yet classic but still very useful Kevin Steves' great document "[**Building a Bastion Host Using HP-UX 11**](http://www.windowsecurity.com/whitepapers/Building_a_Bastion_Host_Using_HPUX_11.html)". This is without any doubt (at least for me) the best document about HP-UX hardening ever done. Although it was written seven years ago it still applies to a wide variety of areas.

In the Center fo Information Security you will find the "[**CIS Level 1 Benchmark for HP-UX**](http://www.cisecurity.org/bench_hpux.html)". These benchmarks are a compilation of security configurations, settings and best practices. Current version applies to all three versions of HP-UX 11i so it is worthwhile to read them. It will ask for registration prior to allow you to download the docs.

In the ITRC Forums there is a [HP-UX Security](http://forums11.itrc.hp.com/service/forums/categoryhome.do?categoryId=155) forum, it is not the most active forum in ITRC but if you post a question you will find that the people is willing to help you.

HP Security Bulletins. Through ITRC you can subscribe to several digests and bulletins, including the HP-UX Security and HP-UX 11.x patches.

Security specific websites. There are a lot of sites and portals focused in security, and in all of them you can find papers about Unix security hardening in general and even some HP-UX specific papers, but as I said at the beginning most of them are obsolete. I usually read [Security Focus](http://www.securityfocus.com/) but there are many others just do a search in Google and you will find them.

Security mailing lists. Probably the most known security mailing list is [Bugtraq](http://www.securityfocus.com/archive/1) but there are others, they talk about HP-UX security bugs from time to time.

And this is the end... well not really. These are the resources I use in my everyday work, if any of you know about other resources please comment them.

See you next time.

Juanma.
