---
title: Generate a Show All report in HP c-Class Blade Enclosures
date: 2011-05-04
type: postclasses: wide
published: true
status: publish
categories:
- HP
tags:
- HP
- HP BladeSystem
- HP c-Class enclosures
- HP Onboard Administrator
author: juan_manuel_rey
comments: true
---

One nice feature of the [HP Onboard Administrator](http://h18004.www1.hp.com/products/blades/components/onboard/index.html?jumpid=reg_R1002_USEN) used to manage the [c-Class Blade Enclosures](http://h18004.www1.hp.com/products/blades/components/enclosures/c-class/index.html) is the possibility to generate a report in plain text format that contains the configuration of the enclosure and the whole inventory, including device bays, interconnect modules, MAC, WWNs, etc.

This report can be very useful for troubleshooting purposes and it's for sure that if you have a problem with an enclosure and open a case to HP Support the Show All report will be amongst the first things you'll be asked for.

Login into the enclosure Onboard Administrator administration interface. From the main screen go to **Enclosure Settings -> Configuration Scripts**.

[![](/assets/images/oa-config-scripts.png "OA configuration scripts")]({{site.url}}/assets/images/oa-config-scripts.png)

You will see the interface to upload your customized scripts, a text box to point to an URL where the script is stored and two links named **SHOW CONFIG** and **SHOW ALL.**

[![](/assets/images/showall.png "Show All")]({{site.url}}/assets/images/showall.png)

Click the **SHOW ALL** link, a new window or tab will open and after a few seconds a report will appear.

[![](/assets/images/report.png "Report")]({{site.url}}/assets/images/report.png)

Save it to your system and you are done. Now you can open the file with any text editor to look into the enclosure data.

Juanma.
