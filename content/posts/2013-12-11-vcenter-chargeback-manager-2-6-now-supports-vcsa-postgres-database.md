---
title: vCenter Chargeback Manager 2.6 now supports vCSA Postgres database
date: 2013-12-11
tags:
- vmware
- vsphere
showComments: true
---

Today VMware has released the latest release of **vCenter Chargeback Manager**. Although this release is more an update than a completely new one that doesn’t mean it comes without new features, on the contrary. The full list of new features and more information about 2.6 release of Chargeback can be found on its [Release Notes](https://www.vmware.com/support/vcbm/doc/vcbm_2_6_release_notes.html). Some of the most interesting are:

- Compatibility with 5.5 versions of vSphere and vCloud Director
- Windows Server 2012 Standard support as host operating system
- Microsoft SQL Server 2012 s supported database

However amongst every other new feature the one that has immediately captured my attention is that finally the vCenter Server Appliance vPostgres embedded database is supported. For me it is a very welcomed new addition, combining CBM 2.6 with vCSA 5.5 you can now manage the cost of your vSphere environment without the need of having an external Oracle database or a Windows-based vCenter Server

In the **Add a New vCenter Server** dialog you will notice that Postgres now appears as an option.

[![](/images/cbm_vc_vpostgres.png "Postgres DB for vCenter")](/images/cbm_vc_vpostgres.png)

For this option there no need to configure database user, instance or port; just provide vCSA IP/FQDN, root user password and we are done.

[![](/images/vc_postgres_db_config_cbm.png)](/images/vc_postgres_db_config_cbm.png)

When the process is completed the newly configured vCenter Server and its database can be checked as always in the settings tab.

[![](/images/cbm_vc_current_status.png "vCenter Server settings")](/images/cbm_vc_current_status.png)

Juanma.
