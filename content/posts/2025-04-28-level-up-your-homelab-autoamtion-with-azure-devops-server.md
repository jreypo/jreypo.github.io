---
title: Level up your homelab automation with Azure DevOps Server
date: 2025-04-28 10:10:00 +0100
description: "Run Azure DevOps Server Express in your homelab for free CI/CD pipelines"
categories:
- Microsoft
- DevOps
- Homelab
tags:
- Microsoft
- Azure DevOps Server
- DevOps
- Homelab
- Self-Hosting
showComments: true
---

**Hello friends!!**

It's been a while — almost two years! Anyway, I'm not going to write the classic "where I've been" post. I'll just say that I missed writing, and here I am again.
So, let's get this party started!

If you're running a homelab (and I'm assuming you are) and looking to improve your automation, development, or CI/CD workflows, **Azure DevOps Server Express** is an excellent option.

**Azure DevOps Server** is the on-premises version of Microsoft's Azure DevOps platform and the 'Express' is the free version of Azure DevOps Server. Designed for small teams and individual developers, supports up to 5 users, and brings the power of Azure DevOps pipelines, repos, and project management to your own infrastructure, making it perfect for homelab use.
In this guide, we’ll explore why Azure DevOps Server Express is worth considering, how to set it up, and what you can do with it in your homelab.

## Why Azure DevOps Server Express?

For homelab enthusiasts, Azure DevOps Server Express provides a robust, on-premises alternative to Azure DevOps Services (the cloud version). Here’s what makes it appealing:

- **Free for up to 5 users** — Perfect for solo developers or small teams.
- **CI/CD Pipelines** — Automate builds and deployments.
- **Version Control** — Use Git or Team Foundation Version Control (TFVC).
- **Work Item Tracking** — Manage tasks, bugs, and features.
- **On-premises Control** — Keep everything inside your network.
- **Integrations** — Works with Docker, Kubernetes, and more.
- **Extensibility** — Extend the functionality of Azure DevOps Server with extensions from the marketplace.
  
Some practical use cases for Azure DevOps Server in your homelab:

- **Automated Home Server Backups** — Use build pipelines to schedule and automate backups of your critical data.
- **Infrastructure as Code (IaC)** — Manage your homelab infrastructure using tools like PowerShell DSC or Ansible, store your configurations in Git, and automate deployments through pipelines.
- **Software Development** — Develop and test your own applications and scripts in a controlled environment.
- **Home Automation Projects** — Use pipelines to deploy and manage your home automation scripts and configurations.
- **Documentation and Knowledge Base** — Use work items and wikis to document your homelab setup and projects.

If you’re running a homelab for development, learning, or automation, this is a powerhouse tool.
I personally use it to manage all my homelab deployments, and my local development, to host the content of my DocFx instance were I run my lab wiki and much more.
Of course for me it was the perfect choice, since I work at Microsoft Azure engineering and already use Azure DevOps Services on a daily basis. Besides, running a local instance of Azure DevOps Server, even if you are not using it to automate your lab infrastructure, is the perfect way to **learn about Azure DevOps Services** without having to run anything in the cloud.

## Azure DevOps Server Express Installation

The installation of Azure DevOps Server Express is very straightforward, and is very well described in [Azure DevOps documentation](https://learn.microsoft.com/en-us/azure/devops/server/install/single-server?view=azure-devops-2022). Review the prerequisites before starting.

- **Windows Server or Windows 10/11** machine - I used Windows Server 2022 to host my ADO instance. Windows Server 2025 is not supported yet.
- **SQL Server Express** - Comes bundled with Azure DevOps Server Express.
- **At least 4 CPU cores and 8 GB RAM** - I deployed a virtual machine in my vSphere lab with those hardware settings, and a 90GB disk.
- **Administrator access** to the machine.

Additionally I set a static IP address in the VM, created and entry on my local DNS server and joined the server to my Active Directory domain — although joining the domain is optional.

You can download the software from the [Azure DevOps Server Express download page](https://visualstudio.microsoft.com/downloads/#azure-devops-server-express).  
You have the option to download either the ISO or an EXE.  
The executable will fetch components during installation, but the ISO is better if you prefer (or need) an offline install.

[![Azure DevOps Server Express installer](/images/ado-installer.png)](/images/ado-installer.png)

### Installation Tips

My first recommendation: **choose "Basic" installation**.

If you choose **Advanced**, the installer won't offer to set up SQL Server Express automatically — it will require you to connect to an existing database manually.

[![Azure DevOps Server Express installer main deployment screen](/images/ado-new-deployment-basic.png)](/images/ado-new-deployment-basic.png)

**Troubleshooting Tip:**  
You may encounter an error where the installer can't connect to the SQL database — this happened to me.  
The solution is simple: re-run the installer. It will detect the existing database, test the connection, and continue the installation.

---

During setup, the installer will ask for the **Application Tier configuration**.
I selected **Custom** and configured **Site Bindings** for both HTTP and HTTPS protocols.
I used a self-signed certificate, but you can also use one from your internal CA — something I plan to do in the future.

[![Azure DevOps Express site bindings](/images/ado-site-bindings.png)](/images/ado-site-bindings.png)

Next, move to the **Search** configuration and choose your preferred authentication options.

[![Azure DevOps Express search configuration](/images/ado-search.png)](/images/ado-search.png)

The installer will validate your settings, and you can then proceed with the final Azure DevOps Server installation.

After the installation completes, launch the **Azure DevOps Server Configuration Center** from the Windows Start menu.  
Here you can configure the proxy, review the search configuration, and fix any issues.

**If the installer reports a problem with the Search service:**  
Uninstall the ElasticSearch service manually using:

```CMD
sc delete elasticsearch-service-x64
```

Then reconfigure the Search service via the Configuration Center.

---

## Post-Installation Configuration

Before you can access your shiny new ADO Server from a browser, a few more steps are needed:

### 1. Configure an Administrator User

Launch the **Azure DevOps Server Express Administration Console**.  
I created a local account named `adoadmin` and added it — along with the local `Administrator` account — to the **Administration Console Users** list in the **Application Tier** section.

[![Azure DevOps Server Express Administration Console](/images/ado-admin-console.png)](/images/ado-admin-console.png)

### 2. Create a Collection

In Azure DevOps Server, a **Collection** is equivalent to an **Organization** in Azure DevOps Services.

Go to **Application Tier -> Team Project Collections** in the Administration Console, and follow the wizard to create your first collection.

---

Finally, access your Azure DevOps Server at:

```text
https://<ado_server_ip_or_fqdn>/
```

or

```text
https://<ado_server_ip_or_fqdn>/<your_collection_name>
```

Authenticate with your administrator account, and create your first project!

[![Azure DevOps Server Express portal](/images/ado-server-portal.png)](/images/ado-server-portal.png)

---

In future posts, I'll explore more topics around Azure DevOps Server and how I use it in my homelab.  
Let me know in the comments if you're using any DevOps solutions in your homelab — whether it's ADO Server, GitLab, or something else.

It’s good to be back.
**Happy homelabbing!**

— Juanma
