---
layout: post
title: Playing with OpenStack Vagrant provider
date: 2016-10-06
type: post
published: true
status: publish
categories:
- OpenStack
- DevOps
- Sysadmin
tags:
- OpenStack
- Vagrant
- DevOps
author: juan_manuel_rey
comments: true
image:
  feature: openstack-banner.jpg
---

Since [Vagrant](http://vagrantup.com) is so tightly integrated in my personal development and learning workflow and my job basically revolves around OpenStack it was about time for me to combine both of them.

I have to admit that at first I did not see the value of using Vagrant to provision machines in OpenStack. OpenStack already provides me with a very powerful command line and web interface, cloud-init to customize the machines and with Heat I can provision complex environments with ease. However after playing a bit with it I can see real value on combining both technologies for testing and learning purposes.

## Anatomy of the Vagrantfile for OpenStack

Besides of the common options for any Vagrantfile there a set of minimum options for the OpenStack provider as described in the [provider Github repository](https://github.com/ggiamarchi/vagrant-openstack-provider).

- `os.openstack_auth_url` - Keystone authentication URL.
- `os.tenant_name` - The tenant where we will provision our instance.
- `os.username` and `os.password` - The credentials for the above tenant.
- `os.flavor` and `os.image` -  Defining the flavor and base image for the instance.
- `os.floating_ip_pool` -  The IP pool for Nova assign the ip pool from in order to allow SSH to the instance.

There are other options that are mandatory if you ask me, like `os.keypair_name` and `config.ssh.private_key_path` that define the SSH key pair to be injected to the instance or `os.networks` which defines the tenant network to connect the instance to.

Below is the Vagrantfile I have used for my test in my lab Red Hat OpenStack installation.

{% highlight ruby %}
require 'vagrant-openstack-provider'

Vagrant.configure("2") do |config|
  config.ssh.username = "fedora"
  config.ssh.private_key_path = "~/.ssh/id_rsa"

  config.vm.provider "openstack" do |os|
    os.openstack_auth_url = "http://192.168.1.11:5000/v2.0"
    os.username = "labadmin"
    os.password = "redhat123"
    os.tenant_name = "starlabs"
    os.server_name = "vagrant-instance"
    os.flavor = "m1.small"
    os.image = "fedora-24"
    os.floating_ip_pool = "public"
    os.networks = "labnet01"
    os.keypair_name = "starlabs"
    os.security_groups = ["default","core_services"]
  end
end
{% endhighlight %}

## Provision your first instance

With our Vagrantfile ready, kicking off a new instance follows the same logic as always with a simple `vagrant up --provider openstack`.

{% highlight text %}
┌─[~/workspace/vagrant-vms/osp-vagrant][jrey:trantor]
└─▪ vagrant up --provider openstack
Bringing machine 'default' up with 'openstack' provider...
==> default: Finding flavor for server...
==> default: Finding image for server...
==> default: Finding network(s) for server...
==> default: Launching a server with the following settings...
==> default:  -- Tenant          : starlabs
==> default:  -- Name            : vagrant-instance
==> default:  -- Flavor          : m1.small
==> default:  -- FlavorRef       : 2
==> default:  -- Image           : fedora-24
==> default:  -- ImageRef        : f6a54d5b-fcbc-4d76-b851-b7bee477bbc3
==> default:  -- KeyPair         : starlabs
==> default:  -- Network         : 601a692c-9eb5-467d-8bb2-ef51e2b79419
==> default: Waiting for the server to be built...
==> default: Using floating IP 192.168.1.192
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 192.168.1.192:22
    default: SSH username: fedora
    default: SSH auth method: private key
    default: Warning: Connection refused. Retrying...
==> default: Machine booted and ready!
==> default: Rsyncing folder: /Users/jrey/workspace/vagrant-vms/osp-vagrant/ => /vagrant
┌─[~/workspace/vagrant-vms/osp-vagrant][jrey:trantor]
└─▪ vagrant ssh
[fedora@vagrant-instance ~]$
[fedora@vagrant-instance ~]$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1400
        inet 172.16.10.5  netmask 255.255.255.0  broadcast 172.16.10.255
        inet6 fe80::f816:3eff:fe2f:fc60  prefixlen 64  scopeid 0x20<link>
        ether fa:16:3e:2f:fc:60  txqueuelen 1000  (Ethernet)
        RX packets 538  bytes 66790 (65.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 458  bytes 55262 (53.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[fedora@vagrant-instance ~]$
{% endhighlight %}

I am planning on investigating more in deep this provider so you can expect more articles around this in the future. Also I have created a new [Github repo]() with the above Vagrantfile and will upload more samples from my day to day tests.

Courteous comments are welcome as always.

-- Juanma
