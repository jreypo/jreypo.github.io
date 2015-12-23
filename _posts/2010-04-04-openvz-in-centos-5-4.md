---
layout: post
title: OpenVZ in CentOS 5.4
date: 2010-04-04
type: post
published: true
status: publish
categories:
- Linux
- Virtualization
tags:
- CentOS
- containers
- Linux
- OpenVZ
- vzctl
- vzlist
author: juan_manuel_rey
comments: true
---

First something I completely forgot in my first post. I discovered OpenVZ thanks to [Vivek Gite's](http://vivekgite.com/) great site [nixCraft](http://www.cyberciti.biz/). This post and the previous one are inspired by his nice series of posts about OpenVZ. Now the show can begin :-)

As I said in my first post about OpenVZ I decided to set-up a test server. Since I didn't had a spare box in my homelab I created a VM inside VMware Workstation, the performance isn't the same as in a physical server but this a test and learn environment so it will suffice.

There is a Debian based bare-metal installer ISO named [**Proxmos Virtual Environment**](http://pve.proxmox.com/wiki/Downloads) and OpenVZ is also supported in many Linux distributions, each one has its own installation method, but I choose CentOS for my Host node server because is one of my favorite Linux server distros.

-   Add the `yum` repository to the server:

{% highlight text %}
[root@openvz ~]# cd /etc/yum.repos.d/
[root@openvz yum.repos.d]# ls
CentOS-Base.repo  CentOS-Media.repo
[root@openvz yum.repos.d]#  wget http://download.openvz.org/openvz.repo
--2010-04-04 00:53:12--  http://download.openvz.org/openvz.repo
Resolving download.openvz.org... 64.131.90.11
Connecting to download.openvz.org|64.131.90.11|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3182 (3.1K) [text/plain]
Saving to: `openvz.repo'

100%[==========================================================================================>] 3,182       --.-K/s   in 0.1s    

2010-04-04 00:53:14 (22.5 KB/s) - `openvz.repo' saved [3182/3182]

[root@openvz yum.repos.d]# rpm --import http://download.openvz.org/RPM-GPG-Key-OpenVZ
[root@openvz yum.repos.d]#
{% endhighlight %}

-   Install the OpenVZ kernel, in my particular case I used the basic kernel but there are SMP+PAE, PAE and Xen kernels available:

{% highlight text %}
[root@openvz yum.repos.d]# yum install ovzkernel
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * addons: ftp.dei.uc.pt
 * base: ftp.dei.uc.pt
 * extras: ftp.dei.uc.pt
 * openvz-kernel-rhel5: openvz.proserve.nl
 * openvz-utils: openvz.proserve.nl
 * updates: ftp.dei.uc.pt
addons                                                                                                       |  951 B     00:00     
base                                                                                                         | 2.1 kB     00:00     
extras                                                                                                       | 2.1 kB     00:00     
openvz-kernel-rhel5                                                                                          |  951 B     00:00     
openvz-utils                                                                                                 |  951 B     00:00     
updates                                                                                                      | 1.9 kB     00:00     
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package ovzkernel.i686 0:2.6.18-164.15.1.el5.028stab068.9 set to be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                 Arch               Version                                         Repository                         Size
====================================================================================================================================
Installing:
 ovzkernel               i686               2.6.18-164.15.1.el5.028stab068.9                openvz-kernel-rhel5                19 M

Transaction Summary
====================================================================================================================================
Install      1 Package(s)         
Update       0 Package(s)         
Remove       0 Package(s)         

Total download size: 19 M
Is this ok [y/N]: y
Downloading Packages:
ovzkernel-2.6.18-164.15.1.el5.028stab068.9.i686.rpm                                                          |  19 MB     00:19     
Running rpm_check_debug
Running Transaction Test
Finished Transaction Test
Transaction Test Succeeded
Running Transaction
 Installing     : ovzkernel                                                                                                    1/1

Installed:
 ovzkernel.i686 0:2.6.18-164.15.1.el5.028stab068.9                                                                                 

Complete!
[root@openvz yum.repos.d]#
{% endhighlight %}

-   Install OpenVZ management utilities:

{% highlight text %}
[root@openvz yum.repos.d]# yum install vzctl vzquota
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * addons: centos.cict.fr
 * base: ftp.dei.uc.pt
 * extras: centos.cict.fr
 * openvz-kernel-rhel5: mirrors.ircam.fr
 * openvz-utils: mirrors.ircam.fr
 * updates: ftp.dei.uc.pt
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package vzctl.i386 0:3.0.23-1 set to be updated
--> Processing Dependency: vzctl-lib = 3.0.23-1 for package: vzctl
--> Processing Dependency: libvzctl-0.0.2.so for package: vzctl
---> Package vzquota.i386 0:3.0.12-1 set to be updated
--> Running transaction check
---> Package vzctl-lib.i386 0:3.0.23-1 set to be updated
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                         Arch                       Version                        Repository                          Size
====================================================================================================================================
Installing:
 vzctl                           i386                       3.0.23-1                       openvz-utils                       143 k
 vzquota                         i386                       3.0.12-1                       openvz-utils                        82 k
Installing for dependencies:
 vzctl-lib                       i386                       3.0.23-1                       openvz-utils                       175 k

Transaction Summary
====================================================================================================================================
Install      3 Package(s)         
Update       0 Package(s)         
Remove       0 Package(s)         

Total download size: 400 k
Is this ok [y/N]: y
Downloading Packages:
(1/3): vzquota-3.0.12-1.i386.rpm                                                                             |  82 kB     00:00     
(2/3): vzctl-3.0.23-1.i386.rpm                                                                               | 143 kB     00:00     
(3/3): vzctl-lib-3.0.23-1.i386.rpm                                                                           | 175 kB     00:00     
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               201 kB/s | 400 kB     00:01     
Running rpm_check_debug
Running Transaction Test
Finished Transaction Test
Transaction Test Succeeded
Running Transaction
 Installing     : vzctl-lib                                                                                                    1/3
 Installing     : vzquota                                                                                                      2/3
 Installing     : vzctl                                                                                                        3/3

Installed:
 vzctl.i386 0:3.0.23-1                                           vzquota.i386 0:3.0.12-1                                          

Dependency Installed:
 vzctl-lib.i386 0:3.0.23-1                                                                                                         

Complete!
[root@openvz yum.repos.d]#
{% endhighlight %}

-   Configure the kernel. The following adjustments must be done in the `/etc/sysctl.conf` file:

{% highlight text %}
# On Hardware Node we generally need
# packet forwarding enabled and proxy arp disabled
net.ipv4.ip_forward = 1
net.ipv6.conf.default.forwarding = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.conf.default.proxy_arp = 0

# Enables source route verification
net.ipv4.conf.all.rp_filter = 1

# Enables the magic-sysrq key
kernel.sysrq = 1

# We do not want all our interfaces to send redirects
net.ipv4.conf.default.send_redirects = 1
net.ipv4.conf.all.send_redirects = 0
{% endhighlight %}

-   Disable SELinux:

{% highlight text %}
[root@openvz ~]# cat /etc/sysconfig/selinux   
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#       enforcing - SELinux security policy is enforced.
#       permissive - SELinux prints warnings instead of enforcing.
#       disabled - SELinux is fully disabled.
SELINUX=disabled
# SELINUXTYPE= type of policy in use. Possible values are:
#       targeted - Only targeted network daemons are protected.
#       strict - Full SELinux protection.
SELINUXTYPE=targeted

# SETLOCALDEFS= Check local definition changes
SETLOCALDEFS=0
[root@openvz ~]#
{% endhighlight %}

-   Reboot the sever with the new kernel.

[![OpenVZ kernel boot](/images/openvz-2010-04-04-23-05-08.png "OpenVZ kernel boot")]({{site.url}}/images/openvz-2010-04-04-23-05-08.png)

-   Check OpenVZ service:

{% highlight text %}
[root@openvz ~]# chkconfig --list vz
vz              0:off   1:off   2:on    3:on    4:on    5:on    6:off
[root@openvz ~]# service vz status
OpenVZ is running...
[root@openvz ~]#
{% endhighlight %}

The first part is over, now we are going to create a VPS as a proof of concept.

-   Download the template of the Linux distribution to install as VPS and place it in `/vz/template/cache`

{% highlight text %}
[root@openvz /]# cd vz/template/cache/
[root@openvz cache]# wget http://download.openvz.org/template/precreated/centos-5-x86.tar.gz
--2010-04-04 23:20:20--  http://download.openvz.org/template/precreated/centos-5-x86.tar.gz
Resolving download.openvz.org... 64.131.90.11
Connecting to download.openvz.org|64.131.90.11|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 179985449 (172M) [application/x-gzip]
Saving to: `centos-5-x86.tar.gz'

100%[==========================================================================================>] 179,985,449  987K/s   in 2m 58s  

2010-04-04 23:23:19 (988 KB/s) - `centos-5-x86.tar.gz' saved [179985449/179985449]

[root@openvz cache]#
{% endhighlight %}

-   Create a new virtual machine using the template.

{% highlight text %}
[root@openvz cache]# vzctl create 1 --ostemplate centos-5-x86
Creating container private area (centos-5-x86)
Performing postcreate actions
Container private area was created
[root@openvz cache]#
{% endhighlight %}

-   We have a basic VPS created but it needs more tweaking before we can start it. Set the IP address, the DNS server, hostname, a name to identify it in the Host node and finally set the On Boot parameter to automatically start the container with the host.

{% highlight text %}
[root@openvz cache]# vzctl set 1 --ipadd 192.168.1.70 --save
Saved parameters for CT 1
[root@openvz cache]# vzctl set 1 --name vps01 --save
Name vps01 assigned
Saved parameters for CT 1
[root@openvz cache]# vzctl set 1 --hostname vps01 --save
Saved parameters for CT 1
[root@openvz cache]# vzctl set 1 --nameserver 192.168.1.1 --save
Saved parameters for CT 1
[root@openvz cache]# vzctl set 1 --onboot yes --save
 Saved parameters for CT 1
[root@openvz cache]#
{% endhighlight %}

-   Start the container and check it with `vzlist`.

{% highlight text %}
[root@openvz cache]# vzctl start vps01
Starting container ...
Container is mounted
Adding IP address(es): 192.168.1.70
Setting CPU units: 1000
Configure meminfo: 65536
Set hostname: vps01
File resolv.conf was modified
Container start in progress...
[root@openvz cache]#
[root@openvz cache]#
[root@openvz cache]# vzlist
 CTID      NPROC STATUS  IP_ADDR         HOSTNAME                        
 1         10 running 192.168.1.70    vps01                           
[root@openvz cache]#
{% endhighlight %}

-   Enter the container and check that its operating system is up and running.

{% highlight text %}
[root@openvz cache]# vzctl enter vps01
entered into CT 1
[root@vps01 /]#
[root@vps01 /]# free -m
 total       used       free     shared    buffers     cached
Mem:           256          8        247          0          0          0
-/+ buffers/cache:          8        247
Swap:            0          0          0
[root@vps01 /]# uptime
 02:02:11 up 8 min,  0 users,  load average: 0.00, 0.00, 0.00
[root@vps01 /]#
{% endhighlight %}

-   To finish the test stop the container.

{% highlight text %}
[root@openvz /]# vzctl stop 1
Stopping container ...
Container was stopped
Container is unmounted
[root@openvz /]#
[root@openvz /]# vzlist -a
 CTID      NPROC STATUS  IP_ADDR         HOSTNAME                        
 1          - stopped 192.168.1.70    vps01                           
[root@openvz /]#
{% endhighlight %}

And as I like to say... we are done ;-) Next time will try to cover more advanced topics.

Juanma.
