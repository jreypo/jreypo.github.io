---
layout: post
title: How to fix VIO Web Client Plugin after a failed deployment
date: 2015-06-08
type: post
published: true
status: publish
categories:
- OpenStack
- VMware
tags:
- OpenStack
- VIO
- VMware
- VMware Integrated OpenStack
author: juan_manuel_rey
comments: true
image:
  feature: openstack-banner.jpg
---

It occurred to me recently that after recovering a VIO failed deployment, in my case an issue with one of the database nodes, in the Web Client Plugin the OpenStack Cluster still was in **Error** state.

[![](/images/screen-shot-2015-06-01-at-12-25-15.png)]({{site.url}}/images/screen-shot-2015-06-01-at-12-25-15.png)

After investigating a bit internally and thanks to my colleague Dimitri Desmidt I was able to solve it.

Open an SSH connection as `viouser` to the VIO Management Server and elevate to `root`. Launch `psql` with the following command:

{% highlight text %}
/opt/vmware/vpostgres/current/bin/psql -U omsdb
{% endhighlight %}

From `psql` prompt execute:

{% highlight text %}
update cluster set status='RUNNING';
{% endhighlight %}

After that logout and login back to Web Client and the cluster will be now in **Running** status.

Juanma.