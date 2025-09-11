---
title: You can run Cockpit in Photon OS
date: 2016-01-12
type: post
classes: wide
published: true
status: publish
categories:
- Cloud-Native
- DevOps
- VMware
- Sysadmin
tags:
- Cloud
- cloud-native
- containers
- Photon
- Linux
- Cockpit
- sysadmin
- systems management
author: juan_manuel_rey
comments: true
---

I have written in the past about using [Cockpit](http://cockpit-project.org) with Fedora and CentOS 7 [here]({% post_url 2015-01-08-managing-your-fedora-server-with-cockpit %}) and [here]({% post_url  2015-01-09-how-to-install-cockpit-on-centos-7 %}). Today playing with one of my Lightwave instances I discovered that Cockpit is also available for VMware Photon OS. The Cockpit packages are available in the `photon-extras` repository. If you do have it enabled in your Photon instances add the following `photon-extras.repo` file in `/etc/yum.repos.d/`.

```ini
[photon-extras]
name=VMware Photon Extras 1.0(x86_64)
baseurl=https://dl.bintray.com/vmware/photon_extras
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY
gpgcheck=0
enabled=1
skip_if_unavailable=True
```

Install `cockpit` package.

```text
root@lightwave01 [ ~ ]# tdnf install cockpit
Installing:
device-mapper                                                              x86_64                                    2.02.116-2.ph1tp2
lvm2                                                                       x86_64                                    2.02.116-2.ph1tp2
libsepol                                                                   x86_64                                         2.4-1.ph1tp2
libselinux                                                                 x86_64                                         2.4-1.ph1tp2
device-mapper-libs                                                         x86_64                                    2.02.116-2.ph1tp2
device-mapper-event-libs                                                   x86_64                                    2.02.116-2.ph1tp2
lvm2-libs                                                                  x86_64                                    2.02.116-2.ph1tp2
mozjs17                                                                    x86_64                                             17.0.0-1
libssh                                                                     x86_64                                              0.6.4-1
polkit-libs                                                                x86_64                                              0.112-1
polkit                                                                     x86_64                                              0.112-1
keyutils-libs                                                              x86_64                                              1.5.9-1
json-glib                                                                  x86_64                                       1.0.2-3.ph1tp2
cockpit-ws                                                                 x86_64                                               0.55-1
cockpit-bridge                                                             x86_64                                               0.55-1
cockpit-shell                                                              noarch                                               0.55-1
cockpit                                                                    x86_64                                               0.55-1
Is this ok [y/N]:
```

Next enable and start `cockpit.socket` with `systemctl`.

```text
root@lightwave01 [ ~ ]# systemctl enable cockpit.socket
root@lightwave01 [ ~ ]# systemctl start cockpit.socket
root@lightwave01 [ ~ ]#
root@lightwave01 [ ~ ]# systemctl status cockpit.socket
* cockpit.socket - Cockpit Web Server Socket
   Loaded: loaded (/usr/lib/systemd/system/cockpit.socket; enabled)
   Active: active (listening) since Tue 2016-01-12 09:57:49 UTC; 38s ago
     Docs: man:cockpit-ws(8)
   Listen: [::]:9090 (Stream)
root@lightwave01 [ ~ ]#
```

Open your favorite web browser and access Cockpit at **http://photon_server:9090**.

[![](/assets/images/cockpit_login_photon.png)]({{site.url}}/assets/images/cockpit_login_photon.png)

You can notice the nice Photon logo :)

The rest of the interface is basically the same, if you want to learn more about Cockpit I encourage you to read the [official documentation](http://cockpit-project.org/guide/latest/) and also the articles I linked at the beginning of this post.

Let's be honest, you will not install and use Cockpit in your average Docker host but for your Lightwave domain controllers it can be useful.

-- Juanma
