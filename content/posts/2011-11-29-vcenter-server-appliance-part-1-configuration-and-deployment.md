---
title: vCenter Server Appliance. Part 1 - Configuration and deployment
date: 2011-11-29
tags:
- linux
- vmware
- vsphere
showComments: true
---

With vSphere 5 VMware has released the vCenter Server Appliance, or vCSA, a linux based alternative to the classic Windows vCenter. During the next three articles I will detail how to deploy and configure the vCSA, the vCenter additional services and how to manage the embedded database.

## vCSA Feature and Limitations

The vCSA appliance is a [SuSE Linux Enterprise Server 11](http://www.suse.com/products/server/) 64-bit virtual machine with the vCenter Server software and its associated services pre-installed. These services include:

- ESXi Dump Collector
- ESXi Syslog Collector
- vSphere Auto Deploy

I will explain how these services are configured in the vCSA in the next article.

The appliance has a minimum requirements of 4GB of RAM, 7GB of disk space and 2 vCPUs. For a more detailed descriptions of vCSA requirements you should check this VMware Knowledge Base article:

- [Minimum requirements for the VMware vCenter Server 5.x Appliance](http://kb.vmware.com/kb/2005086)

The are some limitations for the vCSA, the following vCenter Server features are not supported:

- IPv6
- Linked mode
- SQL Server as backend database
- Security Support Provider Interface (SSPI)
- VMware Update Manager can't be installed in the VvSA, you have to use an additional Windows based VM or physical server. ****

## vCSA Configuration

The vCenter appliance can be deployed only on hosts ESX(i) 4.x or later and like the appliance produced by VMware it comes in OVF format.

Deploy your vCSA from the vSphere client. I will not describe this process since it is very well known and has been very well described in many blog articles and in VMware documentation.

Once vCSA is deployed check it within you vSphere Client. As you will see the appliance is configured with 2 vCPUs and 8GB of RAM by default.

[![](/images/vcsa_hw_config.png "vCSA hardware configuration")](/images/vcsa_hw_config.png)

Power on the vCenter Server Appliance and open its console.

[![image](/images/vcsa_console.png "vCSA Console")](/images/vcsa_console.png)

From the console we can configure the vCSA networking and timezone and we can log into the SLES console.

Select **Configure Network**, a new screen will show and the appliance will ask for its IP address, hostname, gateway and DNS configuration. Answer the questions according to your network environment.

[![](/images/vcsa_net_config.png "vCSA initial network configuration")](/images/vcsa_net_config.png)

After this enter the time zone configuration.

[![image](/images/vcsa_time_zone_config.png "Time Zone Configuration")](/images/vcsa_time_zone_config.png)

And select from the list your time zone.

[![image](/images/vcsa_time_zone_config_selection.png "vCSA Time Zone selection")](/images/vcsa_time_zone_config_selection.png)

With the network and time zone properly configured proceed to your browser and point the URL showed in the console main screen, **https://<VCSA_IP_ADDRESS>:5480**.

The default username and password for the appliance are **root/vmware**.

[![](/images/vcsa_web_login.png)](/images/vcsa_web_login.png)

You will now be presented with a tabbed interface. After accepting the EULA move to the **Database** section within the same tab.

In this screen you have to select the database type. vCSA can only be configured to use the embedded DB2 database or an Oracle external one.

[![](/images/vcsa_db_config.png "Database configuration")](/images/vcsa_db_config.png)

For my homelab vCSA I decided to use the embedded DB2.

If you are going to use the external Oracle option the credentials and network information for the database server have to be provided. Once you are done click **Save Settings**. For the external option you can test your configuration and a database reset option is also provided in case you need it.

[![](/images/vcsa_db_ok.png)](/images/vcsa_db_ok.png)

In the same screen move to the **Settings** section. Here you can specify the inventory size and the vCenter ports. Click Save Settings when you are and like with the database configuration you can perform a test.

[![](/images/vc_server_settings.png "vCenter Server Settings")](/images/vc_server_settings.png)

In **Administration** you can change the administrator account password and enable or disable SSH access to the appliance.

[![](/images/vcsa_vami_admin_settings.png "Admin Settings")](/images/vcsa_vami_admin_settings.png)

Last for the *vCenter Server* tab is the **Storage** screen where you can configure a NFS share to store the log and core files. keep in mind that for this changes to take effect you will need to restart the appliance.

[![](/images/vcsa_log_core_files.png "vCSA log and core file storage settings")](/images/vcsa_log_core_files.png)

The next tab is **Services**. From this tab you can configure, start/stop the **ESXi Services** (Syslog, Netdumper, Auto Deploy) and start/stop the vSphere Web Client.

[![](/images/vc_services_status.png "vCenter services status")](/images/vc_services_status.png)

In the **Status** section you can start and stop the services and in the other sections the ports for each one of the **ESXi Services** can be defined, below is the Syslog screen as an example.

[![](/images/vcsa_esxi_syslog_collector.png "ESXi Syslog Collecto settings")](/images/vcsa_esxi_syslog_collector.png)

Move to **Authentication** tab. The vCenter Server Appliance can be configured to use a NIS or Active Directory. Again if you set any of them you'll need to restart the vCSA for the changes to take effect.

[![](/images/vcsa_ad_settings.png "Active Directory settings")](/images/vcsa_ad_settings.png)

In the **Network** tab you will be able to set the network configuration for the appliance and a proxy server if you want the appliance to be able to access Internet in order to get its updates.

[![](/images/vcsa_network_address.png "Network Address settings")](/images/vcsa_network_address.png)

The **System** tab is quite simple, here you can reboot or shutdown the appliance and the Time Zone.

[![](/images/vcsa_system_info.png "System Information")](/images/vcsa_system_info.png)

Next is the **Update** tab. From the **Status** section you can get information about the VCSA and check for updates.

[![](/images/vcsa_update_status.png "vCSA Update Status")](/images/vcsa_update_status.png)

In **Settings** you can configure how the updates are performed and set an update repository different from the VMware default one.

[![](/images/vcsa_update_settings.png "vCSA Update Settings")](/images/vcsa_update_settings.png)

Finally there is the Upgrade tab. You are not going use this tab until the next release of vSphere 5.

[![](/images/vcsa_prepare_for_upgrade.png)](/images/vcsa_prepare_for_upgrade.png)

The vCSA can not be upgraded in the same manner as its Windows counterpart. Instead you'll have to deploy the new version within your infrastructure and use this interface to establish a trusted connection between the new and old vCSAs. The new appliance will import all data, shutdown the old one and finally take control of its inventory.

We are done with the configuration of the appliance. In the second post of the series I will discuss about the vCenter associated services.

Juanma.
