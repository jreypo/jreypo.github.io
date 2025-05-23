---
title: "Managing Secrets Securely in Azure DevOps Server with Variable Groups"
date: 2025-05-13 13:10:00 +0100
type: post
classes: wide
published: true
status: publish
categories:
- DevOps
- Homelab
tags:
- Microsoft
- Azure DevOps Server
- DevOps
author: juan_manuel_rey
comments: true
---

Welcome to the next article in the Azure DevOps Server series. In this article, we’ll dive into what Variable Groups are, how they can be used securely in Azure DevOps Server, and walk through some practical examples.

In self-hosted DevOps environments, keeping secrets secure while maintaining flexibility can be a challenge. Let's be honest, we are all guilty of hardcoding credentials in a local script to keep things simple and move fast. But that is not the proper way to do it and if you are using your homelab to level up your skills, better do it the right way since the beginning.

**Azure DevOps Server** offers a powerful yet often underutilized feature—**Variable Groups**—to help manage credentials, tokens, and configuration values safely across your pipelines.

## What Are Variable Groups?

Variable Groups are collections of key-value pairs that can be shared across multiple pipelines within a project. These variables can include:

- Non-sensitive values (e.g., environment names, region codes)
- **Secrets** like passwords, tokens, and API keys

When a value is marked as a **secret**, Azure DevOps ensures that it is encrypted at rest and masked in logs.

Variable Groups are stored in the Azure DevOps Library and can be linked to both Classic and YAML pipelines. Although in my homelab I only use YAML pipelines.

## Why Use Variable Groups?

Using Variable Groups in Azure DevOps Server helps:

- Promote **DRY (Don't Repeat Yourself)** principles by centralizing shared configuration
- Simplify **environment management** (e.g., dev, staging, prod)
- Reduce risk by **abstracting secrets** out of pipeline code
- Allow **controlled access** through permission management

## Creating a Variable Group in Azure DevOps Server

These steps apply to **Azure DevOps Server 2020+**, if you remember from my [previous article]({% post_url 2025-04-28-level-up-your-homelab-autoamtion-with-azure-devops-server %}) I am running Azure DevOps Server 2022 in my homelab.

1. Navigate to your project in Azure DevOps Server.
2. Click **Pipelines** > **Library**.
3. Select **+ Variable group**.
4. Name your group (e.g., `vsphere-secrets`).
5. Add variables:
   - `vsphere_admin`
   - `vsphere_password` (check the **"Keep this value secret"** box)
6. Click **Save**.

[![Variable Groups](/assets/images/ado-variable-group.png)]({{site.url}}/assets/images/ado-variable-group.png)

## Using Variable Groups in Pipelines

### In Classic Pipelines

1. Edit your pipeline in the visual editor.
2. Under **Variables**, click **Variable groups**.
3. Click **Link variable group** and choose your group.

The variables will now be available in your tasks using the syntax:

```bash
$(variableName)
```

### In YAML Pipelines

YAML pipelines in Azure DevOps Server (from 2020 onward) also support variable groups:

```yaml
variables:
- group: vsphere-secrets
```

You must grant pipeline access to the variable group for YAML to use it.

## Best Practices for Secrets Management

- Mark all sensitive values as **secret**.
- Use **permissions** on variable groups to control access.
- Consider storing secrets in a **secure external store** (like Azure Key Vault or Hashicorp Vault) and referencing them if your pipelines are hybrid cloud-based.
- Avoid echoing secret variables in logs, even accidentally (e.g., `echo $(secret)`).

## YAML Pipeline to Deploy a VM in vSphere

Here is a sample Azure DevOps Server YAML pipeline that deploys a VM to vSphere from a template. You can trigger this pipeline manually from Azure DevOps Server web console, just why `trigger` is set to `none`. This assumes you have PowerCLI available in your self-hosted agent, more on self-hosted agents in the next article.

```yaml
trigger:
- none

variables:
- group: vsphere-secrets

pool:
  name: Default

steps:
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      $vcenter = "vcsa-01.starlabs.local"
      $username = "$(vsphere_admin)$"
      $password = '$(vsphere_password)'
      $datacenter = "STARLABS"
      $cluster = "LabCluster"
      $template = "ubuntu-22"
      $vmName = "ubuntu-deploy-$(Build.BuildId)"
      $network = "WRK-DVPortGroup"

      # Connect to vCenter
      Connect-VIServer -Server $vcenter -User $username -Password $password

      # Clone VM from template
      New-VM -Name $vmName `
            -Template $template `
            -Datacenter $datacenter `
            -VMHost (Get-Cluster $cluster | Get-VMHost | Sort-Object MemoryGB -Descending | Select-Object -First 1) `
            -NetworkName $network `
            -ResourcePool (Get-Cluster $cluster | Get-ResourcePool | Where-Object { $_.Name -eq "Resources" })

      Write-Host "VM $vmName deployed successfully."
```

## References

- [Azure DevOps Server Documentation – Library Variable Groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops-2020)
- [Secrets in Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops-2020&tabs=yaml%2Cbatch#secret-variables)
- [Secure pipeline secrets with permissions](https://learn.microsoft.com/en-us/azure/devops/pipelines/security/secrets?view=azure-devops-2020)
- [VMware PowerCLI Documentation](https://developer.vmware.com/powercli)

---

If you're managing your own build infrastructure with Azure DevOps Server, **Variable Groups** are an indispensable tool for staying secure and scalable. Take the time to centralize secrets, enforce permissions, and eliminate credential sprawl across your pipelines.

In the next article I will cover self-hosted agents, what they are, how they work and how to deploy different types of self-hosted in you lab.

--Juanma
