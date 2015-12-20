---
layout: post
title: Enabling vSphere Web Access
date: 2010-09-15 16:56:49.000000000 +02:00
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- ESX4
- sysadmin
- systems administration
- vSphere
- vSphere client
- vSphere Web Access
author: juan_manuel_rey
image:
  feature: banner-vmware.png
---

If you are in the virtualization business you'll probably know that since the release of vSphere back in 2009 the web access the ESX servers has been disabled by default. I really never minded about this, I still have nightmares with the awful and useless web interface of the ESX3, and to be sincere who needs a web access when you have SSH access, [PowerCLI](http://communities.vmware.com/community/vmtn/vsphere/automationtools/powercli?ie=UTF-8&q=powercli) and the almighty vSphere Client.

But recently I found myself only with a Linux machine and no remote access to the vCenter Server. With a so limited range of resources I decided to try the web access but I had to enable it.

The first step is to log into the host via SSH. Once you are inside the ESX and from a root shell execute the following command to start the service.

[![](/images/vsphere-web-service1.png "vSphere-web-service")]({{site.url}}/images/vsphere-web-service1.png)

Now you can point your web browser to **http://&lt;esx\_ip\_address&gt;/ui** and login as root, you will note that the interface is pretty much the same as in VMware Server 2.0.

After that I wanted to make the change permanent and like in any normal RedHat Linux server I issued the classic `chkconfig` command.

{% highlight text %}
[root@esx41-01 ~]# chkconfig vmware-webAccess on
{% endhighlight %}

I thought  that everything was done, nothing so far from the reality, after a reboot of the server the Web Access was gone.

[![](/images/vsphere-web-service_21.jpg "vSphere-web-service_2")]({{site.url}}/images/vsphere-web-service_21.jpg)

At that point I no longer needed to access the ESX through the web so I did not spent more time with this; but later with one of my ESX servers at home I finally found how to permanently enable the Web Access.

From the vSphere Client go to the configuration tab of the ESX host and edit the Security Profile from the Software area. The pop-up window will show a list of services, look for the web access and check it.

[![](/images/webaccess-vsphereclient1.jpg "WebAccess-vSphereClient")]({{site.url}}/images/webaccess-vsphereclient1.jpg)

If you now change to a SSH session and ask for the status of the service, will see the service started and enabled.

[![](/images/webaccess-ssh.jpg "WebAccess-SSH")]({{site.url}}/images/webaccess-ssh.jpg)

Reboot the server, if you can of course ;-), and you'll see that the changes are permanent.

Juanma.
