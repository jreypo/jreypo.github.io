---
title: Regenerate Chargeback Manager SSL certificate
date: 2013-10-09
type: post
classes: wide
published: true
status: publish
categories:
- Sysadmin
- Virtualization
- VMware
tags:
- CBM
- Chargeback Manager
- OpenSSL
- SSL
- sysadmin
- vCenter Chargeback
- VMware
author: juan_manuel_rey
comments: true
---

**vCenter Chargeback Manager** gives you the possibility during the installation process to generate an SSL certificate. But this certificate is generated with an expiration period of 60 days.

No problem with that, you can always regenerate it again. Actually Chargeback provides the mechanism to do it. The process can be launched from the *vCenter Chargeback Manager Tools* folder as it can be seen in the screenshot below.

[![](/assets/images/cbm_generate_ssl.png "CBM_generate_SSL")]({{site.url}}/assets/images/cbm_generate_ssl.png)

However this new certificate will come with the same limitation of 60 days of valid period. To easily avoid that we only need to edit the `.bat` file that generates the certificate and modify the correspondent value.

The script is called `Generate_Ssl_Certificate.bat` and can be found `C:\Program Files(x86)\VMware\VMware vCenter Chargeback\Apache2.2\bin`.

[![](/assets/images/gen_ssl_bat_file.png)]({{site.url}}/assets/images/gen_ssl_bat_file.png)

Edit the file using your favorite text editor, in my case I’m using Notepad++, and go to line number 72 as in the capture.

[![](/assets/images/generate_ssl_certificate_bat.png)]({{site.url}}/assets/images/generate_ssl_certificate_bat.png)

Change the `–days` flag from 60 to your desired value, in the example the value is 365 this is the certificate will expire in one year.

{% gist jreypo/11115289 %}

After we can launch the generation process. The batch file will open a `cmd` window, stop the CBM Load Balancer service and asks for the passphrase of default.key. You’ll have to enter it three times and after that the process will ask for information about the State, City, common name (usually the server FQDN), company name, email, etc.

[![](/assets/images/bat_ssl_execution.png)]({{site.url}}/assets/images/bat_ssl_execution.png)

After that it will generate the new certificate and will start Load Balancer services.

[![](/assets/images/ssl_bat_execution_done.png)]({{site.url}}/assets/images/ssl_bat_execution_done.png)

Juanma.
