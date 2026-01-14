---
title: Cisco UCS Platform Emulator, first impressions
date: 2010-10-12
tags:
- networking
- vmware
showComments: true
---

Although I
work at HP, a few of days ago I decided to try the Cisco UCS Platform Emulator. I've been using HP blades and Virtual Connect for years so I thought that it would be great to get a small taste of its most direct competitor. And, of course, this is my blog and not an HP blog so I can try and write about anything I want :-)

The UCS Platform Emulator can be freely downloaded from the Cisco DeveloperNetwork [here](http://developer.cisco.com/web/unifiedcomputing/ucsemulatordownload), you just need to fill a small form. It is also very recommendable to grab a copy of the emulator guide [here](http://developer.cisco.com/c/document_library/get_file?p_l_id=2049008&groupId=2048839&folderId=2049143&name=DLFE-29106.pdf).

The emulator is virtual machine with a [Linux CentOS](http://www.centos.org/) inside, you can run it with VMware Workstation as I do or with the free VMware Player. After firing it up,
the startup process will go like any RedHat based Linux until you will get to a `Starting cisco_ucspe` message. Then It will start to decompress and install the emulator into the VM.

[![](/images/cisco-ucs-platform-emulator-install.png "Cisco UCS Platform Emulator install")](/images/cisco-ucs-platform-emulator-install.png)

After that process is finished you will get to login screen that ask you to use the user and password *config* in order to access the configuration menus of the UCSPE. From those menus you can set stuff like the number of UCS chassis to be used and the number of blades per chassis for example.

[![](/images/cisco-ucs-platform-emulator-config1.png "Cisco UCS Platform Emulator config")](/images/cisco-ucs-platform-emulator-config1.png)

Once you have configured the emulator point your web browser to the IP address of the emulator to access the UCS Manager. The network the UCSPE virtual machines is connected to must be configured to provide IP addresses with DHCP, I used the default NAT network the UCSPE came with. When you are in the web browser click the **LAUNCH** link.

[![](/images/ucs_web_access.jpg "UCS_web_access")](/images/ucs_web_access.jpg)

And here it is, in all its magnificence, the Cisco Unified ComputingSystem Manager.

[![](/images/ucs_manager.jpg "UCS_manager")](/images/ucs_manager.jpg)

From here I will get familiar with the manager and try things like UCS Service Profile creation or playing with the UCS API. I will try to write about it in a future post.

Just a final word. I am very pleased with emulator; have to admit that Cisco did a great work, the whole "thing" was up and running in a breeze and provides you with a very almost-real experience of the UCS platform.

I firmly believe that this is the correct direction that every vendor should follow, yes including HP. If they release simulators to the IT people out there, in the end it will beneficial for them.

Juanma.
