---
title: Getting network card driver version in ESXi 5.0
date: 2011-11-15
tags:
- esxi
- scripting
- sysadmin
- vmware
- vsphere
showComments: true
---

This a quick follow-up post to the [How to check the driver version of a network interface in ESX(i)]({{< ref "posts/2011-03-15-how-to-check-the-driver-version-of-a-network-interface-in-esxi.md" >}}) one. That post covered **ESX(i) 4.x** so I decided to write a small update for **ESXi 5.0**.

First I have to say that the two methods described in my first post still work in ESXi 5.0 Shell.

```text
~ # vmware -l
VMware ESXi 5.0.0 GA
~ #
~ # vmkload_mod -s e1000 | grep Version
Version: Version 8.0.3.1-NAPI, Build: 456551, Interface: 9.2 Built on: Jul 29 2011
~ #
~ # ethtool -i vmnic0
driver: e1000
version: 8.0.3.1-NAPI
firmware-version: N/A
bus-info: 0000:02:00.0
~ #
```

Thanks to the new changes made by VMware in ESXi 5.0 we can now use `esxcli` to get the same result.

```text
~ # esxcli system module get -m e1000
   Module: e1000
   Module File: /usr/lib/vmware/vmkmod/e1000
   License: GPL
   Version: Version 8.0.3.1-NAPI, Build: 456551, Interface: 9.2 Built on: Jul 29 2011
   Signed Status: VMware Signed
   Signature Issuer: VMware, Inc.
   Signature Digest: 1049 0611 a944 efc3 b683 341d 34b1 bebc 552d cb81 a874 ef4c 0562 8f25 2775 8c8d
   Signature FingerPrint: cb44 247a 1614 cea1 2079 362d ec86 9d0e
   Provided Namespaces:
   Required Namespaces: com.vmware.driverAPI@9.2.0.0, com.vmware.vmkapi@v2_0_0_0
~ #
~ # esxcli system module get -m e1000 | grep Version
   Version: Version 8.0.3.1-NAPI, Build: 456551, Interface: 9.2 Built on: Jul 29 2011
~ #
```

There is a big advantage on using `esxcli` over the other methods. In ESX(i) 4.x and ESXi 5.0 with the old procedure you had to be logged into the host but with `esxcli` it can be performed remotely using vSphere CLI.

```text
vi-admin@vma:~[esxi5.vjlab.local]> esxcli system module get -m e1000
   Module: e1000
   Module File: /usr/lib/vmware/vmkmod/e1000
   License: GPL
   Version: Version 8.0.3.1-NAPI, Build: 456551, Interface: 9.2 Built on: Jul 29 2011
   Signed Status: VMware Signed
   Signature Issuer: VMware, Inc.
   Signature Digest: 1049 0611 a944 efc3 b683 341d 34b1 bebc 552d cb81 a874 ef4c 0562 8f25 2775 8c8d
   Signature FingerPrint: cb44 247a 1614 cea1 2079 362d ec86 9d0e
   Provided Namespaces:
   Required Namespaces: com.vmware.driverAPI@9.2.0.0, com.vmware.vmkapi@v2_0_0_0
vi-admin@vma:~[esxi5.vjlab.local]>
vi-admin@vma:~[esxi5.vjlab.local]> esxcli system module get -m e1000 | grep Version
   Version: Version 8.0.3.1-NAPI, Build: 456551, Interface: 9.2 Built on: Jul 29 2011
vi-admin@vma:~[esxi5.vjlab.local]>
```

But there is more, thanks to `Get-EsxCli` cmdlet the same operation can be done using PowerCLI, here it is how.

First we need to setup the `esxcli` instance.

[![](/assets/assets/images/get-esxcli_instance.png)](/assets/assets/images/get-esxcli_instance.png)

And now we issue the command using the name of the module as the argument, please pay attention to the syntax.

[![](/assets/assets/images/get-esxcli_get_nic_driver.png)](/assets/assets/images/get-esxcli_get_nic_driver.png)

As you should have imagined this procedure can be used to get info about any VMkernel module in the host, not just the network interface one.

Juanma.
