---
layout: post
title: vMA 5 rejects SSH connections
date: 2011-10-19
type: post
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- ssh
- sysadmin
- systems administration
- vMA5
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

If you deploy **vMA 5.0** within your virtual infrastructure will find the first time you try to access it by SSH that the appliance is continuously rejecting your connections.

Try to check if SSH is enabled and running and will se that everything is OK.

[![](/images/image6.png "image")]({{site.url}}/images/image6.png)

So you will be asking yourself, what the hell is going on? BTW restarting the service doesn’t help either.

The key to fix this issue is in the `hosts.deny` and `hosts.allow` files. In Unix and Linux systems this files are used by the TCP Wrapper daemon `tcpd` to decide whether or not to accept an incoming connection.

If you look into `host.deny` will see that everything is denied by default.

[![](/images/image7.png "image")]({{site.url}}/images/image7.png)

Add the following line to the `host.allow` file.

{% highlight text %}
sshd: ALL: ALLOW
{% endhighlight %}

Now you can access the appliance via SSH.

[![](/images/image9.png)]({{site.url}}/images/image9.png)

Juanma.
