---
layout: post
title: Taking Chargeback API for a ride - Part 1
date: 2014-04-03 22:25:19.000000000 +02:00
type: post
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- API
- CBM
- Chargeback Manager
- developer
- REST
- VMware
author: juan_manuel_rey
comments: true
---

**vCenter Chargeback** provides a fully featured API that allows to automate many tasks like user and rights management, cost configuration or reporting.

Chargeback API is REST-based, this means that it will receive requests and send responses using HTTP protocol and methods. CBM API implements a set of basic CRUD operations, and each of them maps with an HTTP method as shown in he below table.

METHOD | OPERATION
:-:|:-:
POST | CREATE
GET | READ
PUT | UPDATE/CREATE
DELETE | DELETE

API syntax is actually very easy, it is composed of:

-   Request method
-   Base URL
-   API signature

It’s better illustrated with an example:

{% highlight text %}
POST https://chargeback.corp.local/vCenter-CB/api/login?version=2.5
{% endhighlight %}

We can map the above example with the different elements of the API syntax:

-   `POST`  –This is the request method
-   `https://chargeback.corp.local/vCenter-CB/api` – This the base URL
-   `/login` – This is the API signature, composed of the path to the API we want to call.

We have also included the API version, I usually includes the version as an URL parameter but as we will see later is not really required. Some of the tasks will need also URL parameters that will be placed after the signature.

If there is need for more complex information either in the request or the response an XML payload have to be sent, just like in many other REST APIs. Even to perform a simple login an XML has to be sent, just like the next example.

{% gist jreypo/11113319 %}

For our first ride with CBM API we will use Firefox REST Client add-on, can be found here, this handy add-on provides a visual an easy way to quickly ramp up with any REST API. I personally have used it a lot with Chargeback to try the different API operations during a development project for a customer.

I’m not going to review every possible API, just a few examples to illustrate how it works.

### Login operation

This is the most basic operation of all. In the REST Client paste the XML payload in the Body area, select POST as the method to use and fill the URL field.

[![](/images/rest_login.png "Chargeback API Login with REST client")]({{site.url}}/images/rest_login.png)

### Get hierarchy list

Not every task needs an XML payload, in the following example we are going to get a list of the hierarchies using a GET method and with no message body. The URL to make the request would be:

{% highlight text %}
https://<chargeback_server>/vCenter-CB/api/hierarchies?version=2.5
{% endhighlight %}

After executing the request we can see in the REST Client the response from Chargeback in XML format.

[![](/images/get_hierarchies.PNG "Get hierarchies")]({{site.url}}/images/get_hierarchies.PNG)

If we go to Chargeback web UI we’ll see the listed hierarchies.

[![](/images/hierarchy_cbm_ui.PNG "Chargeback hierarchy")]({{site.url}}/images/hierarchy_cbm_ui.PNG)

### Get all Pricing Models

Another simple request with no XML payload, with a similar syntax to the previous one:

{% highlight text %}
GET https://<chargeback_server>/vCenter-CB/api/costModels?version=2.5
{% endhighlight %}

It will produce however a much more detailed response XML with the details of each of the configured Pricing Models.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<Response xmlns="http://www.vmware.com/vcenter/chargeback/2.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" status="success" isValidLicense="true">
  <CostModels>
    <CostModel id="31">
      <Name>Default Allocation Based Chargeback Pricing Model</Name>
      <Description>(DONT DELETE) This is only for optimization reports, only base rates are allowed for editing.</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
    <CostModel id="30">
      <Name>Default Chargeback Pricing Model</Name>
      <Description>This is the default pricing model shipped with VMware vCenter Chargeback Manager.</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
    <CostModel id="558">
      <Name>VMware Cloud Director Actual Usage Pricing Model</Name>
      <Description>Apply this pricing model to charge for actual usage in hierarchy</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
    <CostModel id="548">
      <Name>VMware Cloud Director Allocation Pool Pricing Model</Name>
      <Description>Apply this pricing model on vDC with Allocation model as 'Allocation Pool' in hierarchy</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
    <CostModel id="550">
      <Name>VMware Cloud Director Networks Pricing Model</Name>
      <Description>Apply this pricing model on organization networks in hierarchy</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
    <CostModel id="556">
      <Name>VMware Cloud Director Overage Allocation Pool Pricing Model</Name>
      <Description>Apply this pricing model to charge for overage on vDC with Allocation model as 'Allocation Pool' in hierarchy</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
    <CostModel id="554">
      <Name>VMware Cloud Director Pay As You Go - Fixed Charging Pricing Model</Name>
      <Description>Apply this pricing model for 'Fixed charging' on vDC with Allocation model as 'Pay As You Go' in hierarchy</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
    <CostModel id="552">
      <Name>VMware Cloud Director Pay As You Go - Resource Based Charging Pricing Model</Name>
      <Description>Apply this pricing model for 'Resource based charging' on vDC with Allocation model as 'Pay As You Go' in hierarchy</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
    <CostModel id="546">
      <Name>VMware Cloud Director Reservation Pool Pricing Model</Name>
      <Description>Apply this pricing model on vDC with Allocation model as 'Reservation Pool' in hierarchy</Description>
      <Currency id="104">
        <Name>USD</Name>
      </Currency>
    </CostModel>
  </CostModels>
</Response>
{% endhighlight %}

### Add a new hierarchy

Invoked using a POST method, that corresponds with the CREATE operation from the table at the beginning of the post. The syntax for the request would be:

{% highlight text%}
POST https://<chargeback_server>/vCenter-CB/api/hierarchy
{% endhighlight %}

In this case I’m not going to put the version as a parameter. An XML payload with the details of the new hierarchy is required.

{% gist jreypo/11114576 %}

[![](/images/cbm_api_add_new_hierarchy.png "Add new hierarchy")]({{site.url}}/images/cbm_api_add_new_hierarchy.png)

Login to Chargeback web interface to check that the new hierarchy is there.

[![](/images/new_cbm_hierarchy_use_api.png "New hierarchy created")]({{site.url}}/images/new_cbm_hierarchy_use_api.png)

I hope that now you have at least a general understanding of how Chargeback API works and how easy is to interact with it. In the second post of the series we will review how to automate Chargeback using vCenter Orchestrator.

Juanma.
