---
title: vCenter Server Appliance. Part 2 - vCenter associated services
date: 2011-11-30
categories:
- Virtualization
- VMware
tags:
- ESXi Netdumper
- ESXi Syslog Collector
- SLES 11
- SuSE Linux
- vCenter Server
- vCenter Server Appliance
- VCSA
- VMware
- vSphere 5
- vSphere Auto Deploy
showComments: true
---

Welcome back to this three-part series of articles about the new vCenter Server Appliance. In this second post we will see how the additional vCenter services work in the vCSA and how to configure them.

- Syslog Collector
- ESXi Dump Collector (AKA Netdumper)
- Auto Deploy

Additionally and although is not a service I decided to include a section about how to collect the vm-support scripts in the VCSA.

## Syslog Collector

Unlike the vCenter Windows version Syslog Collector comes bundled with the VCSA. As we saw in the previous post it can be configured from the VCSA web interface.

[![](/images/image_thumb151_thumb.png "Syslog collector settings")](/images/image_thumb151_thumb.png)

But there are also a limited range of operations that can be done from the command line. Access the vCSA via SSH and make yourself root.

Look if the Syslog Collector is enabled.

[![](/images/image26.png)](/images/image26.png)

Check its status.

[![](/images/image27.png)](/images/image27.png)

Start and stop the Syslog Collector service.

[![](/images/vcsa_syslog_collector_startup.png "Syslog Collector service startup")](/images/vcsa_syslog_collector_startup.png)

This last option is quite useful since the web interface only allows to stop/start ALL the ESXi Services at once and not individually.

If you want to take a look at the Syslog Collector configuration, the configuration file is located at `/etc/syslog-ng/syslog-collector.conf`.

[![](/images/vcsa_syslog_collector_conf.png "syslog-collector.conf file")](/images/vcsa_syslog_collector_conf.png)

## ESXi Dump Collector

Like the other services ESXi Dump Collector, also known as `netdumper`, comes installed with the VCSA and like the Syslog Collector is enabled by default.

It can be configured from the appliance Web UI in the **Services** tab.

[![](/images/esxi_network_coredump_settings.png "ESXi Network Coredump service settings")](/images/esxi_network_coredump_settings.png)

From a root shell you can check the status of the service.

[![](/images/check_netdumper_service.png "Check Network Coredump service")](/images/check_netdumper_service.png)

And start/stop the service.

[![](/images/start_stop_netdumper.png "ESXi Network Coredump service startup from shell")](/images/start_stop_netdumper.png)

The configuration of the Dump Collector is located at `/etc/sysconfig/netdumper`.

[![](/images/netdumper_config_file.png "ESXi Network Coredump configuration file")](/images/netdumper_config_file.png)

Take a look at the following variables:

- `NETDUMPER_DIR` - Storage point for the cores.
- `NETDUMPER_DIR_MAX` - Amount of space configured for the cores.
- `NETDUMPER_PORT` - TCP port of the service, set from the web UI.
- `NETDUMPER_LOG_FILE` - Netdumper log file location
- `NETDUMPER_OPTIONS`

From this file you can manually customize all those values, however for the port I prefer to use the web interface.

## Auto Deploy

Auto Deploy is the only one of the three services that is not enabled by default. As in the Windows based vCenter version Auto Deploy relies on two services:

- DHCP
- TFTP

In the vCenter Server Appliance those services are included in the SuSE Linux the appliance is based on. However by default those services are 0stopped and are configured to do not start during the system startup process.

These services require of some configuration before they can be used.

### DHCP

The configuration file for DHCP is `/etc/dhcpd.conf` but instead of using the default configuration file make a backup of this file and substitute the original with a copy of `/etc/dhcpd.conf.template`.

Once that is done edit the file, it should look like this.

[![](/images/vcsa_dhcp_config_file.png "vCSA dhcp.conf file")]({{site.rul}}/assets/images/vcsa_dhcp_config_file.png)

Substitute the values between `@@` with the values for your network. You may have to comment some of the lines. My `dhcpd.conf` file is below as
reference:

[![](/images/vjlab_dhcp_config_file.png "My dhcp.conf file")](/images/vjlab_dhcp_config_file.png)

Next you have to modify the `/etc/sysconfig/dhcpd` file. In this file is where the interfaces to listen at for the DHCP server are configured.

Check the `DCHP_INTERFACE` variable.

[![](/images/dhcpd_interface.png "DHCPD interface variable")](/images/dhcpd_interface.png)

If it is empty edit the file and set the value to `eth0`.

[![](/images/dhcpd_interface_eth0.png "dhcpd interface set to eth0")](/images/dhcpd_interface_eth0.png)

With the configuration done we need to start the service.

[![](/images/dhcpd_service_start.png "dhcpd service start")](/images/dhcpd_service_start.png)

And configure the service startup level.

[![](/images/dhcpd_service_level_configuration.png "dhcpd service startup level configuration")](/images/dhcpd_service_level_configuration.png)

### TFTP

The configuration file for TFTP server is `/etc/sysconfig/atftp`. There is no need to modify this file since it will work with the default values.

[![](/images/atftpd_config_file.png "atftpd confguration file")](/images/atftpd_config_file.png)

To use a different directory for TFTP server modify the `ATFTPD_DIRECTORY` variable. If you list the contents of that directory you can see the PXE boot files used during the boot of the ESXi server by the Auto Deploy service.

[![](/images/tftpboot_directory.png "/tftpboot directory")](/images/tftpboot_directory.png)

Start the `atftpd` daemon.

[![](/images/atftpd_service_start.png "atftpd service start")](/images/atftpd_service_start.png)

And set the startup level for the service.

[![](/images/atftpd_service_level_configuration.png "atftpd service startup level configuration")](/images/atftpd_service_level_configuration.png)

With the DHCP and TFTP service properly configured and running we can now go back to the VCSA web administration interface and start Auto Deploy. To perform the startup of the service simply click on the **Start ESXi Services** button.

[![](/images/esxi_autodeploy_status_vcsa_ui.png "ESXI Autodeploy service status")](/images/esxi_autodeploy_status_vcsa_ui.png)

## Collecting vm-support scripts

We all know how to generate a support bundle in previous vCenter Server versions and in the 5.0 Windows based one using the vSphere Client or from Windows OS.

For the VCSA the vSphere Client method is completely valid but of course since it is running on SuSE Linux the Windows method doesn't apply. Instead VMware has provided us with two additional methods, one from the Web administration interface and one from the Linux shell.

### Linux shell method

As root go to `/usr/lib/vmware-vpx` and run the `vc-support.sh` script. By default this script will generate the bundle in the current directory but you specify an alternate location with the `-w` flag.

[![](/images/vc_support_script_logs.png "vc-support.sh script execution")](/images/vc_support_script_logs.png)

When the operation is done the following message will appear.

[![](/images/vc_support_finish_message.png)](/images/vc_support_finish_message.png)

Go to the directory where the file has been generated to check it. You can have a quick look of the contents of the bundle using `unzip -l`.

[![](/images/list_log_bundle_content.png "List log bundle content")](/images/list_log_bundle_content.png)

You can download the bundle to your system using you favorite SCP client.

### Web UI method

Go the **vCenter Server** tab and in the **Status** section there is a link to generate the bundle.

[![](/images/download_vc_support_bundle_vcsa_ui.png "Download vCenter Server log Support Bundle from vCSA web")](/images/download_vc_support_bundle_vcsa_ui.png)

Click on the link, a new tab/windows will show up where the log of the operation is displayed. The page refresh every ten seconds until the operation is done.

[![](/images/vc_support_bundle_ready.png "vCenter log Support Bundle ready for download")](/images/vc_support_bundle_ready.png)

Then a link to download the bundle will appear. If you look carefully at the log you will see that this method is no more than a more user friendly version of the Linux shell one.

This file is located at `/tmp/vc-support-bundle/<randomly_generated_directory>`.

[![](/images/vc_support_bundle_location.png "vCenter Server log Support Bundle location")](/images/vc_support_bundle_location.png)

We are done with the vCenter services post. In the next posts I'll show you how to manage the embedded DB2 database.

Juanma.
