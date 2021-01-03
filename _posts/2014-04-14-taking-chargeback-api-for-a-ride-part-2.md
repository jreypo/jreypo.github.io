---
title: Taking Chargeback API for a ride - Part 2
date: 2014-04-14
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- API
- automation
- CBM
- Chargeback Manager
- developer
- javascript
- REST
- vCenter Orchestrator
- vCO
author: juan_manuel_rey
comments: true
---

In the [first post]({% post_url 2014-04-03-taking-chargeback-api-for-a-ride-part-1 %}) of the series we discussed Chargeback API basics and how to interact with it using Firefox REST Client. In this second and final post we will see how Chargeback integrates with **vCenter Orchestrator**.

VMware provides a plugin for the integration between vCO and CBM, however there some caveats with that plugin. First it is for version 2.0 and to be able to use it with 2.5 or above there are some changes to perform, the whole process is described in [VMware KB 203145](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2037145).

Second gotcha of the plugin is that it doesn’t cover every possible API within Chargeback, yes you heard it correctly. To cover those parts we need to use the vCO plugin for REST APIs, of course this plugin needs some configuration before being able to interact with Chargeback but will see later how to do it.

### Add Chargeback server to vCenter Orchestrator inventory

Open vCenter Orchestrator configuration website at **https://vco_server:8283** and install Chargeback and REST plugins. I’m not going to describe the process of installing a plugin since it is fairly well documented in [vCO documentation](http://pubs.vmware.com/vsphere-55/index.jsp?topic=%2Fcom.vmware.vsphere.vco_install_config.doc%2FGUID-01019DF6-AF58-4E77-BD66-437907505924.html).

Next we need to import Chargeback SSL certificate, go to **Network** and then to **SSL Trust Manager** tab. Enter Chargeback server URL in the **Import from URL** textbox and click **Import**.

[![](/assets/images/add_cbm_server_vco.png "Chargeback server URL")]({{site.url}}/assets/images/add_cbm_server_vco.png)

A screen with the details of our Chargeback server will appear, click **Import** to accept the certificate.

[![](/assets/images/vco_import_ssl_cert.png "Import SSL certificate")]({{site.url}}/assets/images/vco_import_ssl_cert.png)

Now from the **Chargeback (2.0.0)** area go to **New Server** tab, enter Chargeback server details and click **Apply Changes**.

[![](/assets/images/vco_add_cbm_details.png "Enter Chargeback server details")]({{site.url}}/assets/images/vco_add_cbm_details.png)

In vCO client our new Chargeback server will appear.

[![](/assets/images/cbm_vco_inventory.png "vCO inventory")]({{site.url}}/assets/images/cbm_vco_inventory.png)

### Using vCO plugin for Chargeback Integration

vCenter Orchestrator plugin for Chargeback allows to perform several administration tasks in a Chargeback server and also cost management, service management and configuration tasks. Following is an example workflow for a configuration and management action.

#### List all hierarchies

We are going to automate a very easy task as our first example. Create a new workflow and name it *List Hierarchies*. As workflow parameters add:

-   cbmServer – Your Chargeback server. Type **Chargeback:ChargebackServer**.
-   cbmVersion – Chargeback API version, 2.0. Type **string**.
-   hierarchyList – The list of hierarchies- Type **Array/Chargeback:Hierarchy**.

In the Schema tab drag an **Action element**, a new windows will pop up to choose the action. Search for **getAllHiearchies** and select it.

[![](/assets/images/vco_cbm_get_all_hierarchies.png "Get all hierarchies")]({{site.url}}/assets/images/vco_cbm_get_all_hierarchies.png)

Check that the workflows attributes are correctly mapped as the input (cbmServer and cbmVersion) and output (hierarchyList) parameters of the action.

Next add a **Scriptable task** element to the workflow and edit it. Map **hierarchyList** attribute as input parameter. In the **Scripting** tab paste this code.

{% gist jreypo/11114725 %}

Validate the workflow and save it. The scheme has to look like this.

[![](/assets/images/cbm_wf_schema.png "Workflow schema")]({{site.url}}/assets/images/cbm_wf_schema.png)

In the execution go to the **Logs** sub-tab in the main **Scheme** tab to check the results of the execution, if everything went as expected you should see something similar to the screenshot.

[![](/assets/images/cbm_list_hr_logs.png "Workflow logs")]({{site.url}}/assets/images/cbm_list_hr_logs.png)

You should be thinking that this example is a bit silly, and to be honest it is. There are few use cases that will require to dump the hierarchy list to vCO log. The purpose of this workflow is to illustrate Chargeback plugin Action elements. Open vCO API Explorer and have a look at all the already implemented Actions that can be used in your workflows.

[![](/assets/images/cbm_actions_vco_api_explorer.png "Chargeback Actions")]({{site.url}}/assets/images/cbm_actions_vco_api_explorer.png)

From basic administration tasks to search and reporting. Also Chargeback plugin comes with a set of Scripting Classes and objects that cover all Chargeback objects although as I mentioned before not all available methods are implemented.

[![](/assets/images/vco_cbm_scripting_classes.png "Scripting Classes")]({{site.url}}/assets/images/vco_cbm_scripting_classes.png)

Make a call to a method from a scripting class is relatively easy as we will see in the following example. We want to get a hierarchy and later use in a different task. We can use a method from **CbServer** class: **getHierarchyByName**. It accepts the hierarchy name and CBM API version as parameters. Basic syntax would be:

{% gist jreypo/11114747 %}

The hierarchy scripting object is now instantiated and we can get any property from it, like its ID.

{% gist jreypo/11114757 %}

### Using vCO HTTP-REST plugin with Chargeback

vCenter Orchestrator has available an HTTP-REST plugin that enables it to interface with systems without a plugin or, like CBM case, to be able to execute some operations not implemented in the plugin. Current vCO plugin doesn’t have implemented the possibility to manage fixed costs in Chargeback. However using the plugin for REST APIs we can circumvent that shortage. But before creating any workflow we need to configure the REST plugin for Chargeback.

#### Configure REST plugin

vCenter Orchestrator plugin for REST APIs comes with a set of workflows for configuration purposes.

[![](/assets/images/vco_rest_wfs.png "REST plugin workflows")]({{site.url}}/assets/images/vco_rest_wfs.png)

We need to add first our Chargeback server as REST host. Launch the **Add a REST host** workflow. Enter the name of the REST host and the base URL for the API, leave the timeout settings with the default values.

[![](/assets/images/vco_add_rest_host.png "Add REST Host")]({{site.url}}/assets/images/vco_add_rest_host.png)

In the next two steps select Basic authentication mode…

[![](/assets/images/vco_add_rest_host_auth.png "Select authentication")]({{site.url}}/assets/images/vco_add_rest_host_auth.png)

…shared session and enter Chargeback server credentials.

[![](/assets/images/vco_rest_host_cbm_credentials.png "Chargeback server credentials")]({{site.url}}/assets/images/vco_rest_host_cbm_credentials.png)

Click submit and have a look at the workflow execution.

If everything went fine we cam see our new REST host in vCO inventory under HTTP-REST. Next step is add the API operations we need to execute using this plugin. Or course you have to add a Login and Logout operation and for our example we are going to add the following ones:

-   Create new Hierarchy
-   Add new Fixed Cost
-   Get Task Status – We will not use this one in any of the workflows of the post but I decided to add it to show how to add URL parameters

Launch **Add a REST operation** workflow and enter the following parameters:

-   Parent host – Our previously added REST host.
-   Name – a unique name for the operation.
-   Template URL – This the API signature in the case of Chargeback. The URL can contain placeholder for parameters to be provided during request phase of the operation.
-   HTTP Method.
-   Content Type – Optional parameter only for POST and PUT methods.

Below are the screenshots and parameters for the five REST  operations we need to add.

##### Login

[![](/assets/images/vco_add_rest_op_login.png "Login")]({{site.url}}/assets/images/vco_add_rest_op_login.png)

##### Logout

[![](/assets/images/vco_add_rest_op_logout.png "Logout")]({{site.url}}/assets/images/vco_add_rest_op_logout.png)

##### Create a new hierarchy

[![](/assets/images/vco_add_rest_op_add_new_hierarchy.png "Create new hierarchy")]({{site.url}}/assets/images/vco_add_rest_op_add_new_hierarchy.png)

##### Add a new Fixed Cost

[![](/assets/images/vco_add_rest_op_add_fixed_cost.png "Add Fixed Cost")]({{site.url}}/assets/images/vco_add_rest_op_add_fixed_cost.png)

##### Get task status

[![](/assets/images/vco_add_rest_op_get_task_status.png "Get Task Status")]({{site.url}}/assets/images/vco_add_rest_op_get_task_status.png)

Look at in the inventory in vCenter Orchestrator client and check that all the new added operations appear under Chargeback REST host.

[![](/assets/images/vco_rest_cbm_new_operations.png "New REST operations")]({{site.url}}/assets/images/vco_rest_cbm_new_operations.png)

Now we can proceed to create our workflows.

#### Add a new Hierarchy to Chargeback

We are going to reproduce one the examples from [Part 1]({% post_url 2014-04-03-taking-chargeback-api-for-a-ride-part-1 %}) using vCO. First create a new empty workflow. As attributes we are going to use the following ones:

-   cbmVersion – Chargeback API version, 2.0.
-   cbmUser – User with administrative privileges in Chargeback
-   cbmPassword – Password of the above user
-   restLogin – Login REST Operation
-   restLogout – Logout REST Operation
-   restCreateHierarchy – Create hierarchy REST Operation
-   loginStatus – Status of the login REST Operation.

[![](/assets/images/vco_rest_add_new_hr_attributes.png "Workflow attributes")]({{site.url}}/assets/images/vco_rest_add_new_hr_attributes.png)

Next configure the input parameters, basically for our purposes here we will need the hierarchy name description.

[![](/assets/images/vco_rest_add_new_hr_parameters.png "Workflow parameters")]({{site.url}}/assets/images/vco_rest_add_new_hr_parameters.png)

At the presentation layer the parameters will shown as **Name of the new hierarchy** and **Description for the new hierarchy**.

[![](/assets/images/vco_rest_add_new_hr_parameters_presentation.png "Presentation layer")]({{site.url}}/assets/images/vco_rest_add_new_hr_parameters_presentation.png)

Add two Scriptable tasks to the workflow, name the first as **API Login** and the second as **API Logout** and paste the code from the previous Login and Logout examples. Now add a third scriptable task to be executed after the login, name it as **Create Hierarchy* and edit it. Paste the below code in the **Scripting** tab.

Edit the API Login element and in the scripting tab add the following Javascript code.

{% gist jreypo/11114778 %}

With this chunk of code will suffice to launch the operation, however there is no error control. The HTTP status code is not enough because the login operation can fail even with a 200 code, like the example below.

[![](/assets/images/vco_rest_add_new_hr_logs_failure.png)]({{site.url}}/assets/images/vco_rest_add_new_hr_logs_failure.png)

To solve this we need to parse the response content. REST plugin scripting API provides a method to retrieve the content of the response as a string.

[![](/assets/images/vco_rest_add_new_hr_rest_response_content.png "REST response")]({{site.url}}/assets/images/vco_rest_add_new_hr_rest_response_content.png)

The following code will do the trick.

{% gist jreypo/11114795 %}

Firstly we need to convert the string to and array, then we get the second element of the array since and look for a successful status. The element has **loginStatus** as output parameter bind to the attribute of the same name. The **System.log** statements are not required but can be useful for troubleshooting purposes.

Add a **Decision** element to the workflow and bind the decision to the **loginStatus** parameter.

[![](/assets/images/vco_rest_add_new_hr_if_condition.png "Decision condition")]({{site.url}}/assets/images/vco_rest_add_new_hr_if_condition.png)

Change the failure branch of the decision from **End workflow** to **Throw exception** and bind it to **loginStatus**. With this decision element we can force the workflow to end the execution if the API login operation was unsuccessful.

Next edit the **Create hierarchy** element. Bind **restCreateHierarchy**, **cbmUser**, **cbmPassword** and **cbmVersion** attributes as input parameters, also bind **hierarchyName** and **hierarchyDescription** as input parameters.

[![](/assets/images/vco_rest_add_new_hr_bind_input_scripting_task.png "Bind input parameters")]({{site.url}}/assets/images/vco_rest_add_new_hr_bind_input_scripting_task.png)

In the scripting tab paste this code.

{% gist jreypo/11114815 %}

The last part, this is the REST operation output is optional but again it can be useful if we need to troubleshoot the workflows using vCO logs.

Our workflow is done and it should look like this.

[![](/assets/images/vco_rest_add_new_hr_final_schema.png "Workflow final schema")]({{site.url}}/assets/images/vco_rest_add_new_hr_final_schema.png)

Launch it and enter a name and a description to test it.

[![](/assets/images/vco_rest_add_new_hr_launch.png "Launch workflow")]({{site.url}}/assets/images/vco_rest_add_new_hr_launch.png)

Click Submit, check the workflow logs for any errors and if everything went as expected go to Chargeback UI and see that the new hierarchy is there.

[![](/assets/images/vco_rest_add_new_hr_new_hr_cbm_ui.png "New CBM hierarchy created")]({{site.url}}/assets/images/vco_rest_add_new_hr_new_hr_cbm_ui.png)

#### Add new Fixed Cost

For our second workflow we are going the same structure as before. hence the first task is duplicate our first workflow and edit the copy. As input parameters we will use:

-   cbmVersion – Chargeback API version, 2.0.
-   cbmUser – User with administrative privileges in Chargeback
-   cbmPassword – Password of the above user
-   restLogin – Login REST Operation
-   restLogout – Logout REST Operation
-   restAddFixed Cost – Add new fixed cost REST Operation
-   loginStatus – Status of the login REST Operation.

[![](/assets/images/vco_rest_add_new_fixed_cost_attributes.png "Add New Fixed cost workflow attributes")]({{site.url}}/assets/images/vco_rest_add_new_fixed_cost_attributes.png)

Very similar to the hierarchy one. However in this case as input parameters we will need much more information.

-   fixedCostName – Fixed Cost name
-   fixedCostDescription – Fixed Cost Description
-   fixedCostCurrency – ID for the currency. A full list of the currencies supported by Chargeback can be found in the [API Reference](https://www.vmware.com/support/vcbm/doc/api_ref_2_6.zip). US Dollar is 104 and Euro 31.
-   isProrated – Configure the new fixed cost as prorated or not. Default value is true. Cannot be used with one-time fixed costs.
-   isPowerStateBased – Configure the fixed cost to be applied only if the virtual machine is powered on. It’s not mandatory, default value is false.
-   fixedCostType – Fixed cost type. A value of 0 represents a recurring fixed cost and 1 represents a one-time fixed cost. It’s not mandatory and the default value is 0.

[![](/assets/images/vco_rest_add_new_fixed_cost_input_parameters.png "Workflow input parameters")]({{site.url}}/assets/images/vco_rest_add_new_fixed_cost_input_parameters.png)

Some of the input parameters are optional but we will use all to illustrate the example with all the possible details. At **Presentation** add a new property for the name and description to make them mandatory. You can also set the default values for the rest of the parameters.

[![](/assets/images/vco_rest_add_new_fixed_cost_input_parameters_presentation.png "Add new property")]({{site.url}}/assets/images/vco_rest_add_new_fixed_cost_input_parameters_presentation.png)

Change the name of the second scriptable task to **Add new Fixed Cost** and edit it. Bind all the needed parameters and attributes.

[![](/assets/images/vco_rest_add_new_fixed_cost_bind_input_parameters.png "Bind input parameters")]({{site.url}}/assets/images/vco_rest_add_new_fixed_cost_bind_input_parameters.png)

In the XML payload we need to reflect all these parameters. Below is code for the scripting part.

{% gist jreypo/11114833 %}

Validate the workflow and save it. Execute it and fill in the input parameters.

[![](/assets/images/vco_rest_add_new_fixed_cost_wf_execution.png "Add New Fixed Cost workflow execution")]({{site.url}}/assets/images/vco_rest_add_new_fixed_cost_wf_execution.png)

Check in the Chargeback UI our newly created Fixed Cost.

[![](/assets/images/vco_rest_add_new_fixed_cost_cost_created_cbm_ui.png "New Fixed Cost created")]({{site.url}}/assets/images/vco_rest_add_new_fixed_cost_cost_created_cbm_ui.png)

We can also have a look at the API response in the workflow logs.

[![](/assets/images/vco_rest_add_new_fixed_cost_wf_logs_api_response.png "API response")]({{site.url}}/assets/images/vco_rest_add_new_fixed_cost_wf_logs_api_response.png)

And we are done. This is the end of this two-post series and I hope that know you all have a better understanding of Chargeback API. As your next step my advice is to try to automate simple API tasks, even if they don't seem to be very useful, and then combine them into  more complex automation workflows along with vCenter and vCloud Director plugins.

In future posts I’ll show a couple of real world examples about how I have used CBM and REST plugins in some customer deployments.

Juanma.
