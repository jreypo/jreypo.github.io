---
title: vCenter Chargeback Manager services and processes
date: 2014-01-30
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- CBM
- Chargeback Manager
- sysadmin
- vCenter Chargeback
- VMware
showComments: true
---

Every customer usually asks about how to monitor their vCenter Chargeback installations, hence I finally decided to write a small post listing the services and processes of the different Chargeback components.

|  **Windows Service**                                            |  **Path to executable** |
|:---------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------:|
|VMware vCenter Chargeback                                         | C:\\Program Files (x86)\\VMware\\VMware vCenter Chargeback\\apache-tomcat\\bin\\tomcat6.exe|
|VMware vCenter Chargeback - VMware Cloud Director DataCollector   | C:\\Program Files (x86)\\VMware\\VMware vCenter Chargeback\\VMware Cloud Director DataCollector\\JavaService.exe|
|VMware vCenter Chargeback - vShield Manager DataCollector         | C:\\Program Files (x86)\\VMware\\VMware vCenter Chargeback\\vShield Manager DataCollector\\JavaService.exe|
|VMware vCenter Chargeback DataCollector-Embedded                  | C:\\Program Files (x86)\\VMware\\VMware vCenter Chargeback\\DataCollector-Embedded\\JavaService.exe|
|VMware vCenter Chargeback Load Balancer                           | C:\\Program Files (x86)\\VMware\\VMware vCenter Chargeback\\Apache2.2\\bin\\httpd.exe|
|----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------|

Bear in mind that if vShield and vCloud DataCollectors are installed on the same server as Chargeback Server the path will be slightly different:

 | **Windows Service**                                                 |  **Path to executable** |
|:--------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------:|
|VMware vCenter Chargeback - vShield Manager DataCollector-Embedded     | C:\\Program Files (x86)\\VMware\\VMware vCenter Chargeback\\vShield Manager DataCollector-Embedded\\JavaService.exe|
|VMware vCenter Chargeback DataCollector-Embedded                       | C:\\Program Files (x86)\\VMware\\VMware vCenter Chargeback\\DataCollector-Embedded\\JavaService.exe|
|---------------------------------------------------------------------  | -------------------------------------------------------------------------------------------------------------------|

Juanma.
