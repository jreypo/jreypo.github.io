---
title: Infrastructure Navigator lose configured hostname after a reboot
date: 2013-11-20
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- sysadmin
- vCenter Infrastructure Navigator
- vIN
- VMware
showComments: true
---

I found this error last week during a deployment in a customer. The **vCenter Infrastructure Navigator** appliance does not maintain its configured hostname after a reboot, it gets reset to the default `localhost.localdom` value.

[![](/images/vin_fail_hostname.png "vIN hostname")](/images/vin_fail_hostname.png)

Setting it again in the administration web interface doesn’t solve problem, it will be lost again after the next reboot.

The problem is in the `vami_set_hostname` script, it has a `HOSTNAME` variable set to `localhost.localdom` and if it fails to make the reverse lookup of the hostname from the IP address using the host command it will be set to the default value.

[![](/images/vami_script_hostname_variable.png "HOSTNAME variable")](/images/vami_script_hostname_variable.png)

To fix this edit that file, it can be found on `/opt/vmware/share/vami`, and set the value of the variable to your hostname. After that reboot the appliance to check that everything works as expected.

Juanma.
