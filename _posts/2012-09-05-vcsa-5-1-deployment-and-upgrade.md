---
title: vCSA 5.1 – Deployment and upgrade
date: 2012-09-05
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- vCenter Server
- vCenter Server Appliance
- VCSA
- vSphere
- vSphere 5.1
author: juan_manuel_rey
comments: true
---

In the [last post]({% post_url 2012-08-28-vcva-5-1-a-quick-look %}) we discussed about the new features and changes that comes with version 5.1 of the **VMware vCenter Server Appliance**. In this new one I will explain how to deploy it and perform an upgrade.

## vCSA Deployment

The deployment of the vCSA 5.1 is very similar to the previous version. Use the vSphere Client to deploy a new OVF template and browse to the vCSA OVA package, follow the instructions until you get to the **Networking Properties** screen.

Here you can pre-set the networking configuration values of the appliance.

[![](/assets/images/deploy_ovf_net_properties.png "Networking Properties")]({{site.url}}/assets/images/deploy_ovf_net_properties.png)

If you are going to do a fresh vCSA or vSphere installation enter the appropriate values on each field. If you want to perform an upgrade instead leave them blank to get the info by DHCP or put a temporal IP address if there is no DCHP server in place. You don’t need to put here the final values since during the upgrading process all the network settings will be migrated.

Let the deployment process finish and power on the vCenter appliance. During the boot process you can see how the network configuration is applied to the VM.

[![](/assets/images/vcsa_network_config_applied.png "Network configuration applied")]({{site.url}}/assets/images/vcsa_network_config_applied.png)

And finally you will reach the familiar blue screen.

[![](/assets/images/vcsa_vm_os_console.png "vCSA console")]({{site.url}}/assets/images/vcsa_vm_os_console.png)

That’s it, now proceed to the appliance web UI to complete vCSA setup. In your first log into the vCenter a wizard will appear.

[![](/assets/images/vcenter_config_wizard.png "vCenter configuration wizard")]({{site.url}}/assets/images/vcenter_config_wizard.png)

After accepting the EULA you will presented with four options:

- Configure with default settings
- Upgrade from a previous version, which also gives you the option to use the default Single Sign On configuration.
- Upload configuration file. Very useful in case your vCenter Server virtual machine gets corrupted or you messed it up, if have a saved copy of the most recent configuration file you can deploy a new appliance and quickly restore its settings by uploading it.
- Set custom configuration - I used this option for my homelab testing.

[![](/assets/images/vc_configuration_options.png "vCenter Configuration Options")]({{site.url}}/assets/images/vc_configuration_options.png)

In the next screen you choose which database you want to use, the vPostgres embedded or an Oracle external.

[![](/assets/images/select_vcsa_db.png "Select vCenter database")]({{site.url}}/assets/images/select_vcsa_db.png)

Now we must choose the options for the **Single Sign On** server. vCenter 5.1 comes with a new component known as the Single Sign On server, or SSO.

SSO allows an administrator to login through the vSphere Web Client or the API and perform operations across all components of the vCloud Suite without having to log into them separately. It integrates with multiple identity sources like Windows Active Directory, NIS and OpenLDAP. The SSO is a requirement for the Inventory Service, vCenter Server and the Web Client.

In the Windows based vCenter installer SSO comes as a separate component that can be installed in the same VM as the vCenter or in a different one as a stand-alone server, in High Availability mode or even in a multisite environment.

vCSA comes with the SSO embedded however it is prepared to use an external SSO server too. If choose the external SSO deployment mode all the appropriate information must be provided in this screen. Again as with the vCenter itself the database type must be set.

[![](/assets/images/vcsa_vc_wizard_sso_settings.png "Configure vCenter SSO")]({{site.url}}/assets/images/vcsa_vc_wizard_sso_settings.png)

Finally set the Active Directory configuration accordingly to your environment, review the configuration and click Start to begin.

[![](/assets/images/review_vcsa_vc_config.png "Review vCenter configuration")]({{site.url}}/assets/images/review_vcsa_vc_config.png)

At the end if everything goes fine you will see a screen with a confirmation, click close and will see al the vCenter services up and ready in the main screen of the WebUI.

[![](/assets/images/vcsa_vima_main_screen.png "vCSA main screen")]({{site.url}}/assets/images/vcsa_vima_main_screen.png)

The deployment and basic setup of the vCSA is done, at this point all other components and settings can be setup from here.

## vCSA Upgrade

The upgrade of the vCenter Server Appliance will allow to update to a different OS version and migrate to a different database.

If you are upgrading from 5.0 to 5.1 and using the embedded database, it will be migrated from IBM DB2 to VMware vPostgres.  The configuration state will be preserved and the schema will be upgraded in order to be compatible with 5.1. VCSA 5.0 Update 1 already comes with vPostgres instead of DB2.

The major upgrade is supported from 5.0 and updates to 5.1.

The upgrading process is relatively simple:

- Deploy vCSA 5.1.
- Set the 5.0 appliance as source and 5.1 as destination.
- Establish a connection between both VCSAs.
- Import network configuration of existing VCSA.

Prior to the upgrading the vCSA 5.1 must have a valid network connection and vCenter Server service must be stopped and un-configured.

Deploy the appliance as shown above, log into the WebUI and in the wizard accept the EULA in the first screen and select **Upgrade from previous version** in the second, let **Use default SSO configuration** as it comes by default.

The next screen that will be displayed is the Local and Remote Appliance keys.

[![](/assets/images/vcsa_local_remote_keys.png "Local and Remote appliance keys")]({{site.url}}/assets/images/vcsa_local_remote_keys.png)

Here we have put the current vCSA 5.0 key. To do so go to the **Upgrade** tab in vCSA 5.0 web interface. In the **Prepare** section select **source** and click **Set role**.

[![](/assets/images/set_appliance_upgrade_role.png "Set upgrade role")]({{site.url}}/assets/images/set_appliance_upgrade_role.png)

Go back to the vCSA 5.1 and copy the **Local appliance key**. On the 5.0 appliance click on **Establish Trust** and paste the copied key in the **Remote key appliance** key field. Click on **Import remote key** and wait for the import to complete.

[![](/assets/images/import_local_appliance_key.png "Establish trust relationship")]({{site.url}}/assets/images/import_local_appliance_key.png)

In the vCSA 5.0 copy the **Local appliance key**. Go to 5.1 vCenter, paste that key on the **Remote appliance key** field of the setup wizard screen and click **Next**. The Pre-Upgrade Checker screen will show up.

[![](/assets/images/vcsa_pre_upgrade_checker.png "Pre-Upgrade checker")]({{site.url}}/assets/images/vcsa_pre_upgrade_checker.png)

After this a check will be run against the ESX/ESXi managed by the old vCSA and it will generate a report.

[![](/assets/images/vcsa_pre_upgrade_report.png "vCenter Appliance Pre-upgrade report")]({{site.url}}/assets/images/vcsa_pre_upgrade_report.png)

And the final screen will appear asking for confirmation. Click on the confirmation checkbox and in **Start**.

[![](/assets/images/vcsa_ready_to_upgrade.png "Ready to upgrade")]({{site.url}}/assets/images/vcsa_ready_to_upgrade.png)

During the upgrade vCSA 5.1 will shutdown the 5.0 virtual appliance and assumes its network identity.

When the process is done a final screen will appear.

[![](/assets/images/vcsa_upgrade_finished.png "vCSA Upgrade done")]({{site.url}}/assets/images/vcsa_upgrade_finished.png)

If you want to check if the process is done log into the vCSA through SSH and list `vmware` services, `vmware-sso` just to name one will show up on the list.

[![](/assets/images/vcsa_list_vmware_services.png "List vmware services")]({{site.url}}/assets/images/vcsa_list_vmware_services.png)

Also you can access the vSphere Web Client and will see the new 5.1 client.

[![](/assets/images/vsphere_51_web_client.png "vSphere Web Client 5.1")]({{site.url}}/assets/images/vsphere_51_web_client.png)

Juanma.
