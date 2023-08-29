---
title: Automate your Azure Bastion tunnels with a shell script 
date: 2023-08-29 18:10:00 +0100
type: post
classes: wide
published: true
status: publish
categories:
- Microsoft
- Linux
tags:
- Microsoft
- Azure
- Linux
author: juan_manuel_rey
comments: true
---

[Azure Bastion](https://learn.microsoft.com/en-us/azure/bastion/) service enables conectivity to Windows and Linux virtual mahciens running on Azure without the need of having RDP or SSH ports open to the public. By default an Azure Bastion connection to a VM will be open in a new tab in the browser, but Bastion also gives you the possibility of establishing a tunnel using Azure CLI and connect with native RDP or SSH clients.

I run many virtual machines in Azure for my day-to-day work, from jumpboxes to development VMs, and I use Azure Bastion all the time to access them. To quickly automate establishing tunnels I created the below shell script, I use it from macOS since I have a MacBook Pro as my daily driver but it can be run in any Linux instance or in WSL as well.

The only requriment is to have [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/) installed and configured.

```bash
#!/bin/bash
#
# Azure Bastion tunnel script by @jreypo
#

echo -n "Enter resource group name: "
read rg
echo -n "Enter Bastion name: "
read bastion
echo -n "Enter virtual machine name: "
read vmname
echo -n "Enter remote resource port: "
read resourceport
echo -n "Enter local port: "
read port

vmid=$(az vm show --resource-group $rg --name $vmname --query id --output tsv)
az network bastion tunnel --resource-group $rg --target-resource-id $vmid --resource-port $resourceport --port $port --name $bastion
```

Hope it helps. I created a [Gist in GitHub](https://gist.github.com/jreypo/babf475d1a2b678b7dc1347f6dc78f9f) with the script code in case you have any fedback or find any issue. Comments are welcome as always.

--Juanma
