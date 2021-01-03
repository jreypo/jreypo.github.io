---
title: How to clean Ironic after a failed overcloud deployment
date: 2016-09-30
type: post
classes: wide
published: true
status: publish
categories:
- OpenStack
- Networking
- Red Hat
tags:
- OpenStack
- Neutron
- networking
- Red Hat
- TripleO
- Red Hat OSP
- Ironic
- Heat
- RDO
- OSP-d
author: juan_manuel_rey
comments: true
image:
  feature: openstack-banner.jpg
---

If your TripleO deployment fails is relatively easy to clean your failed overcloud environment, use `heat stack-delete overcloud` and Heat will take charge of deleting all the stack, the associated deployments and power off the Ironic nodes.

```
[stack@undercloud ~]$ heat stack-delete overcloud
Are you sure you want to delete this stack(s) [y/N]? y
+--------------------------------------+------------+---------------+---------------------+--------------+
| id                                   | stack_name | stack_status  | creation_time       | updated_time |
+--------------------------------------+------------+---------------+---------------------+--------------+
| 45dd9db1-d4c2-48ce-8453-21007cd573b7 | overcloud  | CREATE_FAILED | 2016-09-15T08:49:57 | None         |
+--------------------------------------+------------+---------------+---------------------+--------------+
[stack@undercloud ~]$
[stack@undercloud ~]$ heat stack-list
+--------------------------------------+------------+--------------------+---------------------+--------------+
| id                                   | stack_name | stack_status       | creation_time       | updated_time |
+--------------------------------------+------------+--------------------+---------------------+--------------+
| 45dd9db1-d4c2-48ce-8453-21007cd573b7 | overcloud  | DELETE_IN_PROGRESS | 2016-09-15T08:49:57 | None         |
+--------------------------------------+------------+--------------------+---------------------+--------------+
[stack@undercloud ~]$
```

However there are sometimes when it does not work that way and you will need to manually reset your Ironic nodes. After enduring some pain during a Red Hat OSP Director deployment I decided to document the cleaning process and publish it here.

Power off all Ironic nodes.

```
ironic node-set-power-state <IRONIC_NODE_ID> off`
```

Set provision state to available, this was for me the tricky one and took me some trial error tests until I figured out because the parameter for `ironic node-set-provision-state` is not available but provide, actually there is no available parameter for this Ironic command.

```
ironic node-set-provision-state <IRONIC_NODE_ID> provide
```

Disassociate the nova instances from the nodes.

```
ironic node-update <IRONIC_NODE_ID> remove instance_uuid
```

Get the nodes out of maintenance state.

```
ironic node-set-maintenance <IRONIC_NODE_ID> false
```

List the nodes to verify the executed steps.

```
[stack@undercloud ~]$ ironic node-list
+--------------------------------------+-----------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name      | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+-----------+---------------+-------------+--------------------+-------------+
| fcd44525-ebc4-4e46-8586-143068672490 | ctrl01    | None          | power off   | available          | False       |
| 0336f8f2-4b4a-4d67-a676-3fc85b4bb3bb | ctrl02    | None          | power off   | available          | False       |
| c35bf6e0-0113-45f5-927b-eb6e5b077772 | ctrl03    | None          | power off   | available          | False       |
| c7ee039f-31c8-4bfa-a47b-d2ad41104b65 | compute01 | None          | power off   | available          | False       |
+--------------------------------------+-----------+---------------+-------------+--------------------+-------------+
[stack@undercloud ~]$
```

Finally clean all the nova instances with `nova delete`.

Hopes this helps if you are into OpenStack deployments using TripleO. Comments are welcome.

--Juanma
