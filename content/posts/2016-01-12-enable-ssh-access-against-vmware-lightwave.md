---
title: Enable SSH access against VMware Lightwave
date: 2016-01-12
tags:
- cloud-native
- containers
- vmware
showComments: true
---

After my latest [post]({{< ref "posts/2016-01-11-vmware-lightwave-multi-node-domain-setup.md" >}}) about setting up a Lightwave multi-node domain in this post I'll describe how to configure SSH authentication against Lightwave.

## Configure the service

The first task is enable `PAM` and `nsswitch` for the authentication, use the command `/opt/likewise/bin/domainjoin-cli`.

```text
root [ ~ ]# /opt/likewise/bin/domainjoin-cli configure --enable pam
SUCCESS
root [ ~ ]# /opt/likewise/bin/domainjoin-cli configure --enable nsswitch
SUCCESS
root [ ~ ]#
```

Next using `lwregshell`, the Likewise Registry Shell, we need to update the authentication providers load order. We will indicate Likewise to authenticate first against Active Directory, second vmDir and finally local authentication.

```text
root [ ~ ]# /opt/likewise/bin/lwregshell set_value '[HKEY_THIS_MACHINE\Services\lsass\Parameters\Providers]' LoadOrder "ActiveDirectory" "VmDir" "Local"
root [ ~ ]#
root [ ~ ]# /opt/likewise/bin/lwregshell list_values '[HKEY_THIS_MACHINE\Services\lsass\Parameters\Providers]'
+  "LoadOrder" REG_MULTI_SZ[0] "ActiveDirectory"
              REG_MULTI_SZ[1] "VmDir"
              REG_MULTI_SZ[2] "Local"
root [ ~ ]#
```

Finally restart `lsass` service.

```text
root [ ~ ]# /opt/likewise/bin/lwsm restart lsass
Stopping service: lsass
Starting service: lsass
root [ ~ ]#
```

## Test the authentication

To test the new configuration open an SSH connection to your Docker host with the user created in the previous post, `jreypo`.

```text
root@lightwave01 [ ~ ]# ssh -l jreypo@lightwave.local docker-host01.jreypo.io
The authenticity of host 'docker-host01.jreypo.io (192.168.1.51)' can't be established.
ECDSA key fingerprint is 91:b2:78:4e:47:a4:2c:75:8f:c9:a5:6c:b0:e5:78:19.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'docker-host01.jreypo.io,192.168.1.51' (ECDSA) to the list of known hosts.
Password:
lightwave.local\jreypo [ ~ ]$ who
root     pts/0        Jan 10 11:31 (fedwst23.jreypo.io)
lightwave.local\jreypo pts/1        Jan 11 17:10 (lightwave01.jreypo.io)
lightwave.local\jreypo [ ~ ]$
```

This is a non-priviledge user and will have limited rights within the Docker host, like for example listing the running containers with `docker ps`. Close the connection and open a new one this time with `administrator@lightwave.local` user and run `docker ps`.

```text
lightwave.local\Administrator [ ~ ]$ docker ps
CONTAINER ID        IMAGE                           COMMAND             CREATED             STATUS              PORTS               NAMES
2691b3f1837e        docker-hub:5000/centos:latest   "/bin/bash"         9 seconds ago       Up 9 seconds                            admiring_mclean
lightwave.local\Administrator [ ~ ]$
```

And this is it. Courteous comments are welcome.

-- Juanma
