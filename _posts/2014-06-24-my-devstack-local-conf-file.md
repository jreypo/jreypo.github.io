---
title: My DevStack local.conf file
date: 2014-06-24
type: post
classes: wide
published: true
status: publish
categories:
- Cloud
- OpenStack
tags:
- Cloud
- developer
- DevStack
- Fedora
- OpenStack
author: juan_manuel_rey
comments: true
image:
  feature: openstack-banner.jpg
---

One of the pillars of my personal OpenStack ecosystem is [**DevStack**](http://devstack.org/). For those of you new to the OpenStack world DevStack is a tool, basically is a shell script, that allows to deploy a full OpenStack environment, installing every required dependency. It is widely used amongst beginners and the development community.

My DevStack instance is a Fedora 20 virtual machine with 2 vCPUs and 2GB of memory, I use it mostly for testing and development. I have setup a small vSphere environment in Fusion with a vCSA virtual machine and a nested ESXi, both 5.5 version. The DNS and NFS services are provided by my management VM which is another Fedora 20 VM with just 512MB of RAM.

{% gist jreypo/ce4cfa3546dc6ac3ac8d %}

My `local.conf` file is no rocket science as you have seen, but may be it can be of help to anyone wanting to quickly setup a DevStack+vSphere development environment.

Juanma.

 
