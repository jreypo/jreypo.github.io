---
layout: post
title: Managing VIO with viocli
date: 2016-04-27
type: post
published: true
status: publish
categories:
- OpenStack
- VMware
tags:
- OpenStack
- VIO
- VIO 2.0
- VMware
- VMware Integrated OpenStack
- viocli
author: juan_manuel_rey
comments: true
image:
  feature: openstack-banner.jpg
---

VIO 2.0 comes with several command line utilities, in a previous [post](http://blog.jreypo.io/openstack/vmware/how-to-patch-your-vio-environment/) I showed the usage of `viopatch` to perform patching of an existing VIO installation. In this post I will discuss `viocli` which is the main command line utility that allow, via several subcommands, to perform many maintenance and administration operations from VIO Manager. We will cover the most commonly used and useful `viocli` subcommands.

`viocli` must always be run using `sudo`. The available subcommands can be retrieved with a simple `viocli -h`

```
viouser@vio-manager:~$ viocli -h
usage: viocli [-h]
              {backup,recover,restore,show,upgrade,rollback,deployment,services,hyperic,dbverify,ds-migrate-prep}
              ...

CLI for VIO

positional arguments:
  {backup,recover,restore,show,upgrade,rollback,deployment,services,hyperic,dbverify,ds-migrate-prep}
                        sub-command help
    backup              backup OpenStack database or management server data
    recover             recover broken/missing VIO deployment nodes
    restore             restore OpenStack database or management server data
    show                show information about the deployment
    upgrade             upgrade the deployment to a new version
    rollback            rollback the upgrade to the original deployment
    deployment          run deployment related actions
    services            start or stop VIO services
    hyperic             manage hyperic agents in VIO deployment
    dbverify            verifies the openstack database for known issues
    ds-migrate-prep     prepare a datastore for maintenance

optional arguments:
  -h, --help            show this help message and exit
```

To get the help for any of the subcommands run `viocli <command> -h`. The configuration for `viocli` can be found in `/opt/vmware/vio/etc/viocli.conf`

```
viouser@vio-manager:/opt/vmware/vio/etc$ cat viocli.conf
[DEFAULT]
deployment=VIO

[OMS_CONNECTION]
protocol=https
#hostname=localhost.localdom
cert_path=/opt/vmware/vio/etc/oms.crt
port=8443
viouser@vio-manager:/opt/vmware/vio/etc$
```

## viocli deployment

`viocli deployment` can be used to manage an existing VIO deployment. It allows to start, stop, pause, resume or configure a deployment. Can also be used to create a certificate signing request, update the certificate or get the logs for the deployment.

### Usage

```
viouser@vio-manager:~$ sudo viocli deployment -h
usage: viocli deployment [-h] [-v] [-d [NAME]] [-p]
                         {start,stop,pause,resume,configure,cert-req-create,cert-update,getlogs,default}
                         ...

positional arguments:
  {start,stop,pause,resume,configure,cert-req-create,cert-update,getlogs,default}
    start               start the deployment
    stop                stop the deployment
    pause               pause the deployment
    resume              resume the deployment
    configure           reconfigure the entire deployment
    cert-req-create     create a certificate signing request for signing by a
                        CA
    cert-update         updates VIO with provided certificate.
    getlogs             get logs for the deployment
    default             get/set the default deployment

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         verbose mode
  -d [NAME], --deployment [NAME]
                        deployment to use (default: VIO)
  -p, --progress        show progress of current operation
viouser@vio-manager:~$
```

### Start a deployment

```
viouser@vio-manager:~$ sudo viocli deployment start -d VIO-20
```

### Stop a deployment

```
viouser@vio-manager:~$ sudo viocli deployment stop -d VIO-20
```

### Get the logs

```
viouser@vio-manager:~$ sudo viocli deployment getlogs -d VIO-20
```

## viocli show

Used to retrieve the list of nodes of a VIO deployment and get information about the deployment's inventory file.

```
viouser@vio-manager:~$ sudo viocli show -d VIO-20
hostname         role           vm name                ip
---------------  -------------  ---------------------  ---------------
compute01        ComputeDriver  VIO-20-ComputeDriver-0  192.168.110.179
controller01     Controller     VIO-20-Controller-0     192.168.110.173
database01       DB             VIO-20-DB-0             192.168.110.174
loadbalancer01   LoadBalancer   VIO-20-LoadBalancer-0   192.168.110.177
memorycache01    Memcache       VIO-20-Memcache-0       192.168.110.176
messagequeue01   RabbitMQ       VIO-20-RabbitMQ-0       192.168.110.178
mongodb01        MongoDB        VIO-20-MongoDB-0        192.168.110.180
objectstorage01  ObjectStorage  VIO-20-ObjectStorage-0  192.168.110.175
viouser@vio-manager:~$
```

## viocli services

`viocli services` can be used to start and stop the services within a VIO deployment. The main differerence with `viocli deployment` is that `services` will only start/stop the services running on the virtual machines and `deployment` subcommand will start/stop the whole OpenStack cluster, including the virtual machines.

```
viouser@vio-manager:~$ sudo viocli services start
```

## viocli upgrade

The recommended to perform a major version upgrade in VIO is still the vSphere Web Client, as I shown in [VIO upgrade post](http://blog.jreypo.io/openstack/vmware/upgrading-vmware-integrated-openstack/), however using `viocli upgrade` the upgrade can also be performed from VIO Manager shell.

### Usage

```
viouser@vio-manager:~$ sudo viocli upgrade -h
usage: viocli upgrade [-h] [-v] [-d [NAME]] [-p] [-n [NEW_DEPLOYMENT_NAME]]
                      [--public-vip [PUBLIC_VIP]]
                      [--internal-vip [INTERNAL_VIP]] [-f]

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         verbose mode
  -d [NAME], --deployment [NAME]
                        deployment to use (default: VIO)
  -p, --progress        show progress of current operation
  -n [NEW_DEPLOYMENT_NAME], --new-deployment [NEW_DEPLOYMENT_NAME]
                        new deployment name
  --public-vip [PUBLIC_VIP]
                        temporary public VIP address to be used in a new
                        deployment
  --internal-vip [INTERNAL_VIP]
                        temporary private VIP address to be used in a new
                        deployment
  -f, --force           do not ask for confirmation (default: false)
viouser@vio-manager:~$
```

### Perform an upgrade

```
viouser@vio-manager:~$ sudo viocli upgrade -d VIO-10 -p -n VIO-20 --public-vip 192.168.130.120 --internal-vip 172.16.20.120
```

## viocli rollback

This `viocli` subcommand will allow to rollback a VIO upgrade in case of any failure during the upgrade operation.

```
viouser@vio-manager:~$ sudo viocli rollback --deployment VIO-10 --progress --force
```

## viocli backup

This command allows to perform a backup of the VIO Manager server of the OpenStack database. Requires an NFS volume available, there is no need to be mounted on the manager since `viocli` will take care of that.

### Management Server backup

```
viouser@vio-manager:~$ sudo viocli backup mgmt_server 192.168.1.99:/vio_backup
```

### OpenStack database backup

```
viouser@vio-manager:~$ sudo viocli backup openstack_db -d VIO 192.168.1.99:/vio_backup
```

## viocli restore

`viocli restore` can be used to restore a backup of the VIO management server or the OpenStack database.

### Management Server restore

```
viouser@vio-manager:~$ sudo viocli restore mgmt_server -d VIO vio_os_db_20160202205408 192.168.1.99:/vio_backup
```

### OpenStack database restore

```
viouser@vio-manager:~$ sudo viocli restore openstack_db -d VIO vio_os_db_20160202205408 192.168.1.99:/vio_backup
```

## viocli recover

The recover subcommand can be used to recover a node or a group of nodes after a critical failure. Since most of the nodes are stateless there is no need of a database backup to recover them, for database nodes you must provide the NFS path and the name of the backup file. When you recover a VIO node, it returns to the state of a newly deployed node.

Recovery operations can be executed by node with `-n` or by role with `-r`. Get the list of nodes from a deployment with `viocli show`. Recovery by role will perform a recover of all the nodes of that role while recovery by role allows to specify a single node to be recover.

### Recover by node

The following example shows the recovery operation for the controller01 node.

```
viouser@vio-manager:~$ viocli recover -n controller01
```

### Recover by role

The following example show the recover of the OpenStack DB role, that will perform the recovery of all DB nodes. As it can be seen the name of the backup and the NFS path must be provided.

```
viouser@vio-manager:~$ viocli recover -r DB -dn vio_os_db_20160202205408 -nfs 192.168.1.99:/vio_backup
```

## viocli dbverify

The `dbverify` command allows and administrator to verify an OpenStack database before doing an upgrade. It can find knows issued like duplicated or missing keys, that may casue problems during the upgrade operation.

```
viouser@vio-manager:~$ sudo viocli dbverify -d VIO-20
Running data validation against deployment: VIO-20 ...
Data validation completed, results are as follows:
==============================================
* Checking for duplicate default security groups
No duplicates found

viouser@vio-manager:~$
```

## viocli hyperic

`viocli hyperic` can be used to manage the Hyperic agent. Tasks like installation, configuration, and start/stop can be performed.

```
viouser@vio-manager:~$ sudo viocli hyperic -h
usage: viocli hyperic [-h] [-v] [-d [NAME]]
                      {install,config,uninstall,stop,start} ...

positional arguments:
  {install,config,uninstall,stop,start}
    install             install Hyperic agent
    config              config Hyperic agent
    uninstall           uninstall Hyperic agent
    stop                stop Hyperic agent
    start               start Hyperic agent

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         verbose mode
  -d [NAME], --deployment [NAME]
                        deployment to use (default: VIO)
viouser@vio-manager:~$
```

To install the agent use the following syntax.

```
viouser@vio-manager:~$ sudo viocli hyperic install -d VIO-20
```

And we are done. Comments are welcome.

-- Juanma
