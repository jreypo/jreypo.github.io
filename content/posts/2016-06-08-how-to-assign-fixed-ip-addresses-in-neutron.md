---
title: How to assign fixed IP addresses in Neutron
date: 2016-06-08
categories:
- OpenStack
- Networking
tags:
- OpenStack
- Neutron
- networking
showComments: true
image:
  feature: openstack-banner.jpg
---

In **OpenStack** the most common way to provide connectivity to the instances is to rely on DHCP service provided by Neutron, is simple and clean. However there are some uses cases where a static IP address is preferable, like for a DNS or LDAP server hosted within the tenant.

Fortunately Neutron provides with a mechanism to create ports within a tenant subnet, and their associated IP addresses, and then attach them to an instance at boot time. First we net to need to retrieve the network and subnet IDs.

```text
[root@osp8 ~(keystone_tenant)]# neutron net-show tenant-net
+-----------------+--------------------------------------+
| Field           | Value                                |
+-----------------+--------------------------------------+
| admin_state_up  | True                                 |
| id              | 82c6d713-81e8-4c11-a808-f98184f278d0 |
| mtu             | 0                                    |
| name            | tenant-net                           |
| router:external | False                                |
| shared          | False                                |
| status          | ACTIVE                               |
| subnets         | 649933b2-9cba-4bda-a71e-b1d337b3cc7a |
| tenant_id       | b4e6fc6d2dc242cc91130dbe20733f69     |
+-----------------+--------------------------------------+
[root@osp8 ~(keystone_tenant)]# neutron subnet-show 649933b2-9cba-4bda-a71e-b1d337b3cc7a
+-------------------+--------------------------------------------------+
| Field             | Value                                            |
+-------------------+--------------------------------------------------+
| allocation_pools  | {"start": "172.16.10.2", "end": "172.16.10.254"} |
| cidr              | 172.16.10.0/24                                   |
| dns_nameservers   |                                                  |
| enable_dhcp       | True                                             |
| gateway_ip        | 172.16.10.1                                      |
| host_routes       |                                                  |
| id                | 649933b2-9cba-4bda-a71e-b1d337b3cc7a             |
| ip_version        | 4                                                |
| ipv6_address_mode |                                                  |
| ipv6_ra_mode      |                                                  |
| name              | tenant-subnet                                    |
| network_id        | 82c6d713-81e8-4c11-a808-f98184f278d0             |
| subnetpool_id     |                                                  |
| tenant_id         | b4e6fc6d2dc242cc91130dbe20733f69                 |
+-------------------+--------------------------------------------------+
[root@osp8 ~(keystone_tenant)]#
```

Proceed to create the port with `neutron port-create`command passing the subnet ID and the IP address we want to reserve as arguments.

```text
[root@osp8 ~(keystone_tenant)]# neutron port-create tenant-net --fixed-ip subnet_id=649933b2-9cba-4bda-a71e-b1d337b3cc7a,ip_address=172.16.10.16
Created a new port:
+-----------------------+--------------------------------------------------------------------------------------------------------------+
| Field                 | Value                                                                                                        |
+-----------------------+--------------------------------------------------------------------------------------------------------------+
| admin_state_up        | True                                                                                                         |
| allowed_address_pairs |                                                                                                              |
| binding:vnic_type     | normal                                                                                                       |
| device_id             |                                                                                                              |
| device_owner          |                                                                                                              |
| dns_assignment        | {"hostname": "host-172-16-10-16", "ip_address": "172.16.10.16", "fqdn": "host-172-16-10-16.openstacklocal."} |
| dns_name              |                                                                                                              |
| fixed_ips             | {"subnet_id": "649933b2-9cba-4bda-a71e-b1d337b3cc7a", "ip_address": "172.16.10.16"}                          |
| id                    | 6d9e348a-6f34-4086-8b62-fdc035f8f0e5                                                                         |
| mac_address           | fa:16:3e:bd:9f:19                                                                                            |
| name                  |                                                                                                              |
| network_id            | 82c6d713-81e8-4c11-a808-f98184f278d0                                                                         |
| security_groups       | 2527372a-c9a1-4615-9de4-3b4dc12e2d21                                                                         |
| status                | DOWN                                                                                                         |
| tenant_id             | b4e6fc6d2dc242cc91130dbe20733f69                                                                             |
+-----------------------+--------------------------------------------------------------------------------------------------------------+
[root@osp8 ~(keystone_tenant)]#
```

Now with the port created we can boot our instance using the port ID as the argument for the `--nic` setting.

```text
[root@bnk-osp8 ~(keystone_tenant)]# nova boot --nic port-id=6d9e348a-6f34-4086-8b62-fdc035f8f0e5 --flavor m2.tiny --image rhel-72 --key-name cloud-user dns-server
+--------------------------------------+------------------------------------------------+
| Property                             | Value                                          |
+--------------------------------------+------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                         |
| OS-EXT-AZ:availability_zone          |                                                |
| OS-EXT-STS:power_state               | 0                                              |
| OS-EXT-STS:task_state                | scheduling                                     |
| OS-EXT-STS:vm_state                  | building                                       |
| OS-SRV-USG:launched_at               | -                                              |
| OS-SRV-USG:terminated_at             | -                                              |
| accessIPv4                           |                                                |
| accessIPv6                           |                                                |
| adminPass                            | YAkQEgD3iFE3                                   |
| config_drive                         |                                                |
| created                              | 2016-06-08T07:51:29Z                           |
| flavor                               | m2.tiny (222290c8-d9f0-4550-b25e-fa29f8de878a) |
| hostId                               |                                                |
| id                                   | af93c462-8fa3-4715-b234-e113c4a8af8b           |
| image                                | rhel-72 (9a53a691-d852-45c9-affd-5d64093c84de) |
| key_name                             | cloud-user                                     |
| metadata                             | {}                                             |
| name                                 | dns-server                                     |
| os-extended-volumes:volumes_attached | []                                             |
| progress                             | 0                                              |
| security_groups                      | default                                        |
| status                               | BUILD                                          |
| tenant_id                            | b4e6fc6d2dc242cc91130dbe20733f69               |
| updated                              | 2016-06-08T07:51:29Z                           |
| user_id                              | e9f25838fccc4c939c35d7af0993fc0f               |
+--------------------------------------+------------------------------------------------+
[root@bnk-osp8 ~(keystone_tenant)]#
[root@bnk-osp8 ~(keystone_tenant)]# nova list
+--------------------------------------+------------+--------+------------+-------------+----------------------------------+
| ID                                   | Name       | Status | Task State | Power State | Networks                         |
+--------------------------------------+------------+--------+------------+-------------+----------------------------------+
| af93c462-8fa3-4715-b234-e113c4a8af8b | dns-server | ACTIVE | -          | Running     | tenant-net=172.16.10.16          |
+--------------------------------------+------------+--------+------------+-------------+----------------------------------+
[root@bnk-osp8 ~(keystone_tenant)]#
```

At this point you can proceed to configure the instance networking with the static IP address or you can simply leave it using DHCP, the great thing about this solution is that the instance will always get the same static IP address from the DHCP server as long as the port remain attached to it.

Comments are welcome.

--Juanma
