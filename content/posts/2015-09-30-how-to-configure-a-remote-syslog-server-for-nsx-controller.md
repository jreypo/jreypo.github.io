---
title: How to configure a remote syslog server for NSX Controller
date: 2015-09-30
tags:
- devops
- networking
- nsx
- scripting
- sysadmin
showComments: true
---

As with the rest of **NSX for vSphere** components any competent admin would like to configure a remote syslog server for the NSX Controllers, in my homelab I have vRealize Log Insight and recently I decided to configure it on my NSX Controllers and document the procedure here mostly as self-reference.

NSX Manager has the option to configure a remote syslog server using its management web site, but where is the option for the Controllers? Well, if you lurk around NSX interface in vSphere Web Client will quickly notice that the option is somehow missing.  Actually the only option to enable it is using NSX REST API. Let's see how to do it.

For this post I will use Firefox REST Client Add-on but you can use your favorite REST client. Firstly any REST API call will require at least the Authentication header, in Firefox REST Client click on **Authentication** drop-down menu, select **Basic Authentication** and enter the admin credentials.

[![](/images/screen-shot-2015-09-30-at-02-25-19.png)](/images/screen-shot-2015-09-30-at-02-25-19.png)

Additionally PUT and POST methods will require you to set a custom header with the following values that will define the content of the HTTP Request body. Use these values.

- Name: Content-Type
- Value: application/xml

With these two headers set enter the API URL, in my case it is:

```
https://nsxm-01.mcorp.local/api/2.0/vdn/controller/controller-1/syslog
```

To construct this URL you will need the controller ID that can be get in the NSX interface in vSphere Web Client as shown below.

[![](/images/screen-shot-2015-09-30-at-02-50-38.png)](/images/screen-shot-2015-09-30-at-02-50-38.png)

Select POST as the method. You will need to enter the body for the HTTP Request, in XML format. Use the below code as an example to build the content for the HTTP Request body.

{{< jreypo 8840b4e887095e5a460d >}}

This XML code will indicate the NSX Manager to set the IP address in the `syslogServer` node as the remote syslog server for the controller in the URL. The protocol, port and log level are also defined.

[![](/images/screen-shot-2015-09-17-at-01-14-19.png)](/images/screen-shot-2015-09-17-at-01-14-19.png)

Submit the request and if everything is configured as described you will receive a `200 OK` status code.

[![](/images/screen-shot-2015-09-17-at-01-15-06.png)](/images/screen-shot-2015-09-17-at-01-15-06.png)

At this point the syslog server is configured for all NSX Controllers, you can check the status using also an API call with the same URL and selecting GET method.

Comments are welcome.

Juanma.
