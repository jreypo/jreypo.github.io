---
title: Combining Heat with OpenStack Vagrant provider
date: 2016-10-18
categories:
- OpenStack
- DevOps
- Sysadmin
tags:
- OpenStack
- Vagrant
- DevOps
- Heat
showComments: true
image:
  feature: openstack-banner.jpg
---

After my [first post]({{< ref "posts/2016-10-06-playing-with-openstack-vagrant-provider.md" >}}) about the OpenStack Vagrant provider I decided to play a bit more with the provider and I found it can be combined with an [OpenStack Heat](https://wiki.openstack.org/wiki/Heat) stack. Since the provider itself cannot provision any new network resource, just use the existing ones, with this method you can create all the new network resources with Heat and the instances with Vagrant.

To test the integration I created a simple Vagrantfile and a simple Heat stack. In the Vagrantfile, we will define a new setting called `os.stacks` with the name of the stack and the path to the template file. This is the YAML file that define all our new network configuration. We also define the name of the network in the `os.networks` setting.

Finally we define the instance to be provisioned and override any additional parameter, like `ssh.username`.

```ruby
require 'vagrant-openstack-provider'

Vagrant.configure("2") do |config|
  config.ssh.private_key_path = "~/.ssh/id_rsa"

  config.vm.provider "openstack" do |os|
    os.openstack_auth_url = "http://192.168.1.11:5000/v2.0"
    os.username = 'labadmin'
    os.password = 'redhat123'
    os.tenant_name = 'starlabs'
    os.keypair_name = 'starlabs'
    os.flavor = 'm1.small'
    os.image = 'fedora-24-x64'
    os.security_groups = ['default','core_services']
    os.floating_ip_pool = 'public'
    os.networks << 'labnet05'
    os.stacks << {name: 'vg-heat-stack',
                  template: "#{File.dirname(__FILE__)}/vg-heat-stack.yaml"}
  end

  config.vm.define 'instance-1' do |s|
    s.vm.provider "openstack" do |os, overrride|
      os.server_name = 'instance-1'
      overrride.ssh.username = "fedora"
    end
  end
end
```

In the Heat stack we define the creation of the new network, subnet and router resources, in this case `labnet05`, `labsubnet05` and `router5` respectively.

```yaml
heat_template_version: 2015-04-30

description: >
    Create the network infrastructure to deploy an instance

parameters:
  external_network_id:
    type: string
    label: public
    description: UUID for the external network
    default: 57b7661e-fc3b-4cb8-9336-cc3fbbe9fdb7

resources:
  labnet05:
    type: OS::Neutron::Net
    properties:
      name: labnet05
  labsubnet05:
    type: OS::Neutron::Subnet
    properties:
      name: labsubnet05
      cidr: 172.16.22.0/24
      dns_nameservers: [8.8.8.8]
      network_id: { get_resource: labnet05 }
      allocation_pools:
        - { start: 172.16.22.100, end: 172.16.22.150 }
  router5:
    type: OS::Neutron::Router
    properties:
      name: router5
      admin_state_up: True
      external_gateway_info: { "network": { get_param: external_network_id }}
  router5_int0:
    type: OS::Neutron::RouterInterface
    properties:
      router_id: { get_resource: router5 }
      subnet_id: { get_resource: labsubnet05 }
```

As described in the first post execute a simple `vagrant up --provider openstack` and Vagrant will create the stack, kick off the instance, connect it to the new network and assign a floating IP address.

Both files can be found in my `vagrant-openstack-samples` [Github repository](https://github.com/jreypo/vagrant-openstack-samples).

-- Juanma
