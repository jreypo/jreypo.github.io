---
layout: post
title: VMware Lightwave multi-node domain setup
date: 2016-01-10
type: post
published: true
status: publish
categories:
- Cloud-Native
- DevOps
- VMware
tags:
- Cloud
- cloud-native
- containers
- Photon
- Lightwave
author: juan_manuel_rey
comments: true
---

[VMware Lightwave](https://vmware.github.io/lightwave/) is an identity and management access service for Cloud-Native apps. It was released to the community last year and the source code can be accessed in VMware's Github. Since I'm revamping my homelab to become more cloud-native aware Lightwave was the natural choice to provide authentication services, I dedcide to setup a multi-node domain to be able to test different Lightwave scenarios.

## Lightwave installation

First of all we will need two VMware Photon virtual machines with static IP addresses and unique hostnames, these will be our domain controllers. Then on each of them list the available repositories for `tdnf`.

{% highlight text %}
root@lightwave01 [ ~ ]# tdnf repolist
repo id             repo name                               status
photon-updates      VMware Photon Linux 1.0(x86_64)Updates  enabled
photon              VMware Photon Linux 1.0(x86_64)         enabled
lightwave           VMware Lightwave 1.0(x86_64)            enabled
root@lightwave01 [ ~ ]#
{% endhighlight %}

If the `lightwave` is not there we need to add it. Go to `/etc/yum.repos.d/` and create the file `lightwave.repo`, edit it and add the following content.

{% highlight text %}
[lightwave]
name=VMware Lightwave 1.0(x86_64)
baseurl=https://dl.bintray.com/vmware/lightwave
gpgkey=file:///etc/pki/rpm-gpg/VMWARE-RPM-GPG-KEY
gpgcheck=1
enabled=1
skip_if_unavailable=True
{% endhighlight %}

Then check that if the repository photon-extras is present and if not repeat the same process but with the below content.

{% highlight text %}
[photon-extras]
name=VMware Photon Extras 1.0(x86_64)
baseurl=https://dl.bintray.com/vmware/photon_extras
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY
gpgcheck=0
enabled=1
skip_if_unavailable=True
{% endhighlight %}

Install `vmware-lightwave-server` package.

{% highlight text %}
tdnf install vmware-lightwave-server
{% endhighlight %}

## Configure first domain controller

With Lightwave Server installed in both nodes promote the first one to domain controller. Provide the domain, in my case `lightwave.local` and the administration password.

{% highlight text %}
root@lightwave01 [ ~ ]# /opt/vmware/bin/ic-promote --domain lightwave.local --password VMware1!
20160110184918:INFO:Setting up system as Infrastructure standalone node
20160110184918:INFO:Starting service [dcerpc]
20160110184918:INFO:Starting service [vmafd]
20160110184918:INFO:Starting service [vmdir]
20160110184918:INFO:Starting service [vmca]
20160110184918:INFO:Setting various configuration values
20160110184918:INFO:Promoting directory service to be domain controller
20160110184920:INFO:Setting up the logical deployment unit
20160110184920:INFO:Setting up VMware Certificate Authority
20160110184921:INFO:Adding VMCA's root certificate to VMware endpoint certificate store
20160110184921:INFO:Generating Machine SSL cert
20160110184921:INFO:Setting Machine SSL certificate
20160110184921:INFO:Publishing Machine SSL certificate for directory service
20160110184921:INFO:Restarting service [vmdir]
Domain Controller setup was successful
root@lightwave01 [ ~ ]#
{% endhighlight %}

Now that our fist domain controller is ready lets create our first user in the domain. For this task we will use `dir-cli` command, also in `/opt/vmware/bin` path.

{% highlight text %}
root@lightwave01 [ ~ ]# /opt/vmware/bin/dir-cli user create --account jreypo --first-name "Juan Manuel" --last-name Rey --user-password "VMware1!"
Enter password for administrator@lightwave.local:
User account [jreypo] created successfully
root@lightwave01 [ ~ ]#
{% endhighlight %}

## Configure second domain controller

To configure the second domain controller we will use the same `ic-promote` command but with the `--partner` option to indicate the primary domain controller.

{% highlight text %}
rootlightwave02 [ ~ ]# /opt/vmware/bin/ic-promote --partner lightwave01.jreypo.io --domain lightwave.local
Password (administrator@lightwave.local):
20160111083335:INFO:Setting up system as Infrastructure partner node
20160111083335:INFO:Validating credentials to partner [lightwave01.jreypo.io] at domain [lightwave.local]
20160111083335:INFO:Starting service [dcerpc]
20160111083335:INFO:Starting service [vmafd]
20160111083335:INFO:Starting service [vmdir]
20160111083335:INFO:Starting service [vmca]
20160111083335:INFO:Setting various configuration values
20160111083335:INFO:Promoting directory service to be domain controller
20160111083347:INFO:Setting up the logical deployment unit
20160111083347:INFO:Setting up VMware Certificate Authority
20160111083347:INFO:Adding VMCA's root certificate to VMware endpoint certificate store
20160111083348:INFO:Generating Machine SSL cert
20160111083348:INFO:Setting Machine SSL certificate
20160111083348:INFO:Publishing Machine SSL certificate for directory service
20160111083348:INFO:Restarting service [vmdir]
Domain Controller setup was successful
rootlightwave02 [ ~ ]#
{% endhighlight %}

## Join the domain

With both domain controllers configured now we are going to join one Photon Docker host to the domain in order to verify the setup. First we need to install the client tools, configure the same `lightwave` and `photon-extras` repositories and install `vmware-lightwave-clients` package.

{% highlight text %}
tdnf install vmware-lightwave-clients
{% endhighlight %}

With the client tools installed use the command `ic-join` to join the domain.

{% highlight text %}
root [ ~ ]# /opt/vmware/bin/ic-join --domain lightwave.local --domain-controller lightwave01.jreypo.io --password VMware1!
20160111133217:INFO:Setting up system as client to Infrastructure node at [lightwave01.jreypo.io]
20160111133217:INFO:Validating credentials to partner [lightwave01.jreypo.io] at domain [lightwave.local]
20160111133217:INFO:Starting service [dcerpc]
20160111133217:INFO:Starting service [vmafd]
20160111133217:INFO:Setting various configuration values
20160111133217:INFO:Joining system to directory service at [lightwave01.jreypo.io]
20160111133218:INFO:Get root certificate from VMware Certificate Authority
20160111133218:INFO:Adding VMCA's root certificate to VMware endpoint certificate store
20160111133218:INFO:Generating Machine SSL cert
20160111133219:INFO:Setting Machine SSL certificate
Domain Join was successful
root [ ~ ]#
{% endhighlight %}

And we are done, in a future post I'll show you how to enable SSH access against Lightwave.

-- Juanma
