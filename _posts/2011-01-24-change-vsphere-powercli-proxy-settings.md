---
title: Change vSphere PowerCLI proxy settings
date: 2011-01-24
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- PowerCLI
- vCenter Server
- VMware
- vSphere
author: juan_manuel_rey
comments: true
---

Today I was performing a test in the vSphere cluster I have in my laptop and when I tried to connect to my vCenter Server with PowerCLI I got the following error.

```text
C:\Users\juanma
[vSphere PowerCLI] % get-vc vcenter.mlab.local -user Administrator -Password vmwarerules!
Connect-VIServer : 24/01/2011 12:58:33    Connect-VIServer        Could not connect using the requested protocol.   
At line:1 char:7
+ get-vc <<<<  vcenter.mlab.local -user Administrator -Password J3d1kn1gh/
 + CategoryInfo          : ObjectNotFound: (:) [Connect-VIServer], ViServerConnectionException
 + FullyQualifiedErrorId : Client20_ConnectivityServiceImpl_Reconnect_ProtocolError,VMware.VimAutomation.ViCore.Cmdlets.Commands.ConnectVIServer

C:\Users\juanma
[vSphere PowerCLI] %
```

My first thought after that was to check network connectivity and the firewall configuration of the vCenter Server but everything was OK. Then after a quick search in the VMware Communities I found the solution in this [post](http://communities.vmware.com/message/1657689).

The problem was the proxy server configuration of PowerCLI, I'm used to do everything directly from the vCenter desktop, where I have installed PowerCLI and the vSphere Client, but this time I tried to connect directly from my laptop and since I was connected to the corporate network PowerCLI was trying to connect through the proxy server. Following is how I fixed this thanks to the above VMTN post.

First retrieve the PowerCLI proxy configuration with the `Get-PowerCLIConfiguration` cmdlet.

```text
C:\Users\juanma
[vSphere PowerCLI] % Get-PowerCLIConfiguration

Proxy Policy    Default Server
                Mode          
------------    ---------------
UseSystemProxy  Multiple      

C:\Users\juanma
[vSphere PowerCLI] %
```

As you can see `Proxy Policy` is set to `UseSystemProxy`. To set this value to `NoProxy` use the cmdlet `Set-PowerCLIConfiguration`.

```text
C:\Users\juanma
[vSphere PowerCLI] % Set-PowerCLIConfiguration -ProxyPolicy NoProxy

Perform operation?
Performing operation 'Update vSphere PowerCLI configuration.'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"): y

Proxy Policy    Default Server
                Mode          
------------    ---------------
NoProxy         Multiple      

C:\Users\juanma
[vSphere PowerCLI] %
```

Now try to reconnect to the vCenter Server and everything should go without errors.

```text
C:\Users\juanma
[vSphere PowerCLI] % get-vc vcenter.mlab.local -user Administrator -Password vmwarerules!
WARNING: There were one or more problems with the server certificate:

* The X509 chain could not be built up to the root certificate.

* The certificate's CN name does not match the passed value.

Name                                           Port                                        User                                        
----                                           ----                                        ----                                        
vcenter.mlab.local                             443                                         Administrator                               

C:\Users\juanma
[vSphere PowerCLI] %
```

Juanma.
