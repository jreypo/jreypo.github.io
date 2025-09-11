---
title: The Atomic Series. Part 1 - Introduction to Atomic Host
date: 2016-11-15
type: post
classes: wide
published: true
status: publish
categories:
- Red Hat
- Linux
- Cloud-native
tags:
- Project Atomic
- Kubernetes
- Atomic
- Fedora
- Docker
- Linux
- Containers
- Cloud-native
author: juan_manuel_rey
comments: true
---

Welcome to the Atomic Series! In this new series of articles I will discuss the different part of the Project Atomic ecosystem starting with the Atomic Host in this first part. I sincerely hope you like it found it helpful. The planned posts, for now are:

- Part 1 - Introduction to Atomic Host
- Part 2 - Atomic Clusters
- Part 3 - Atomic Registry

In the future I am planning to write more posts around Atomic but without numbering.

## What is Project Atomic?

[**Project Atomic**](http://www.projectatomic.io/) is the name of the community, sponsored by Red Hat, that encompass a set of Open Source projects geared towards creating of a new family of operating systems and infrastructure to run container-based workloads.  

The basic building block of the project is the Atomic Host, a lightweight container operating system, however the project also serves as the umbrella of projects like rpm-ostree, Atomic Registry, Atomic App or Cockpit which are also components of Openshift Origin, the upstream of Red Hat Openshift Container Platform. Project Atomic includes the upstream work on Docker and Kubernetes communities.

## What is Atomic Host

Atomic Host is a lightweight container optimized operating system designed to be immutable, currently available as Fedora and CentOS flavors, is also the upstream of Red Hat Enterprise Linux Atomic Host. It is very important to understand that Atomic Host is not a new Linux distribution but it is built on the foundation of Fedora, CentOS and Red Hat Enterprise Linux.

It differs from traditional operating systems, which by the way can also run containers, in that it is optimized to run and managing containers. Every application is run and managed as a container in Atomic Host.  

Besides of the container optimized host it provides tools like [rpm-ostree](https://github.com/projectatomic/rpm-ostree), [Docker](https://www.docker.com/), [Kubernetes](http://kubernetes.io/), [Cockpit](http://cockpit-project.org/) and the Atomic command line tools. Currently the only components that come installed in Atomic, besides of the Fedora, CentOS or RHEL base images, are the Atomic command line, Docker and Kubernetes; however there are plans to implement Kubernetes as containers in the future as well.

## rpm-ostree

Based on [OSTree](https://ostree.readthedocs.io/en/latest/), the first thing you have to know about rpm-ostree is that it is not a package management system like `yum`, `dnf` or `apt`. Instead rpm-ostree is an open source tool to manage bootable, immutable, versioned filesystem trees. The main idea behind rpm-ostree is to use a client-server architecture to keep Linux hosts updated and in sync with the latest packages in a reliable manner.

### Check current status

```text
[fedora@atomic-01 ~]$ sudo rpm-ostree status
  TIMESTAMP (UTC)         VERSION   ID             OSNAME            REFSPEC
* 2016-06-15 09:57:04     24.39     2c7d41e8a6     fedora-atomic     fedora-atomic:fedora-atomic/24/x86_64/docker-host
[fedora@atomic-01 ~]$
[fedora@fed-atomic-01 ~]$ sudo rpm-ostree status -p
============================================================
  * DEFAULT ON BOOT
----------------------------------------
  version    24.39
  timestamp  2016-06-15 09:57:04
  id         2c7d41e8a67931fe21bc92100c59cff8a94c2df5a0e6a1b75957bda141601481.0
  osname     fedora-atomic
  refspec    fedora-atomic:fedora-atomic/24/x86_64/docker-host
============================================================
[fedora@fed-atomic-01 ~]$
```

### Upgrade the system

The upgrade option allows you also to preview the changes, display the current version, etc.

```text
[fedora@fed-atomic-01 ~]$ sudo rpm-ostree upgrade --help
Usage:
  rpm-ostree upgrade [OPTION...] - Perform a system upgrade

Help Options:
  -h, --help            Show help options

Application Options:
  --os=OSNAME           Operate on provided OSNAME
  -r, --reboot          Initiate a reboot after an upgrade is prepared
  --allow-downgrade     Permit deployment of chronologically older trees
  --preview             Just preview package differences
  --check               Just check if an upgrade is available
  --sysroot=SYSROOT     Use system root SYSROOT (default: /)
  --peer                Force a peer-to-peer connection instead of using the system message bus
  --version             Print version information and exit

[fedora@fed-atomic-01 ~]$
[fedora@fed-atomic-01 ~]$ sudo rpm-ostree upgrade --version
rpm-ostree 2015.11
  +compose
[fedora@fed-atomic-01 ~]$
```

Performing the upgrade is a simple `rpm-ostree upgrade`.

```text
[fedora@fed-atomic-01 ~]$ sudo rpm-ostree upgrade
Updating from: fedora-atomic:fedora-atomic/24/x86_64/docker-host

2061 metadata, 9718 content objects fetched; 354168 KiB transferred in 551 seconds
Copying /etc changes: 20 modified, 0 removed, 46 added
Transaction complete; bootconfig swap: yes deployment count change: 1
[fedora@fed-atomic-01 ~]$
```

After the upgrade reboot the host and verify the new tree is in use.

```text
[fedora@fed-atomic-01 ~]$ sudo rpm-ostree status
State: idle
Deployments:
● fedora-atomic:fedora-atomic/24/x86_64/docker-host
       Version: 24.81 (2016-11-14 20:46:13)
        Commit: 49dd9520a7c537ced9c846c2e2f47643b5f52a22768d944b6d8c1108da38f39e
        OSName: fedora-atomic

  fedora-atomic:fedora-atomic/24/x86_64/docker-host
       Version: 24.39 (2016-06-15 09:57:04)
        Commit: 2c7d41e8a67931fe21bc92100c59cff8a94c2df5a0e6a1b75957bda141601481
        OSName: fedora-atomic
[fedora@fed-atomic-01 ~]$
```

### Rollback an upgrade

```text
[fedora@fed-atomic-01 ~]$ sudo rpm-ostree rollback
Moving '2c7d41e8a67931fe21bc92100c59cff8a94c2df5a0e6a1b75957bda141601481.0' to be first deployment
Transaction complete; bootconfig swap: yes deployment count change: 0
Changed:
  NetworkManager 1:1.2.4-3.fc24 -> 1:1.2.2-1.fc24
  NetworkManager-libnm 1:1.2.4-3.fc24 -> 1:1.2.2-1.fc24
  atomic 1.13.1-3.git5dfcaa9.fc24 -> 1.8-5.gitcc5997a.fc24
  audit 2.6.7-1.fc24 -> 2.5.2-1.fc24
  audit-libs 2.6.7-1.fc24 -> 2.5.2-1.fc24
  audit-libs-python 2.6.7-1.fc24 -> 2.5.2-1.fc24
  audit-libs-python3 2.6.7-1.fc24 -> 2.5.2-1.fc24
  bash 4.3.42-7.fc24 -> 4.3.42-5.fc24
  bash-completion 1:2.4-1.fc24 -> 1:2.3-1.fc24
  bind99-libs 9.9.9-2.P3.fc24 -> 9.9.9-1.P1.fc24
  bind99-license 9.9.9-2.P3.fc24 -> 9.9.9-1.P1.fc24
  boost-iostreams 1.60.0-7.fc24 -> 1.60.0-5.fc24
  boost-program-options 1.60.0-7.fc24 -> 1.60.0-5.fc24
  boost-random 1.60.0-7.fc24 -> 1.60.0-5.fc24
  boost-regex 1.60.0-7.fc24 -> 1.60.0-5.fc24
  boost-system 1.60.0-7.fc24 -> 1.60.0-5.fc24
  boost-thread 1.60.0-7.fc24 -> 1.60.0-5.fc24
  ca-certificates 2016.2.10-1.0.fc24 -> 2016.2.7-1.0.fc24
  ceph-common 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  checkpolicy 2.5-6.fc24 -> 2.5-2.fc24
  chkconfig 1.8-1.fc24 -> 1.7-2.fc24
  cockpit-bridge 0.117-1.fc24 -> 0.103-1.fc24
  cockpit-docker 0.117-1.fc24 -> 0.103-1.fc24
  cockpit-networkmanager 0.117-1.fc24 -> 0.103-1.fc24
  cockpit-ostree 0.117-1.fc24 -> 0.103-1.fc24
  cockpit-shell 0.117-1.fc24 -> 0.103-1.fc24
  coreutils 8.25-7.fc24 -> 8.25-5.fc24
  coreutils-common 8.25-7.fc24 -> 8.25-5.fc24
  cronie 1.5.1-2.fc24 -> 1.5.0-4.fc24
  cronie-anacron 1.5.1-2.fc24 -> 1.5.0-4.fc24
  cryptsetup 1.7.2-1.fc24 -> 1.7.1-1.fc24
  cryptsetup-libs 1.7.2-1.fc24 -> 1.7.1-1.fc24
  curl 7.47.1-9.fc24 -> 7.47.1-4.fc24
  dbus 1:1.11.6-1.fc24 -> 1:1.11.2-1.fc24
  dbus-glib 0.108-1.fc24 -> 0.106-1.fc24
  dbus-libs 1:1.11.6-1.fc24 -> 1:1.11.2-1.fc24
  device-mapper 1.02.122-2.fc24 -> 1.02.122-1.fc24
  device-mapper-event 1.02.122-2.fc24 -> 1.02.122-1.fc24
  device-mapper-event-libs 1.02.122-2.fc24 -> 1.02.122-1.fc24
  device-mapper-libs 1.02.122-2.fc24 -> 1.02.122-1.fc24
  device-mapper-persistent-data 0.6.3-1.fc24 -> 0.6.2-0.1.rc6.fc24
  dhcp-client 12:4.3.4-3.fc24 -> 12:4.3.4-2.fc24
  dhcp-common 12:4.3.4-3.fc24 -> 12:4.3.4-2.fc24
  dhcp-libs 12:4.3.4-3.fc24 -> 12:4.3.4-2.fc24
  dnsmasq 2.76-1.fc24 -> 2.75-4.fc24
  docker 2:1.10.3-54.gite03ddb8.fc24 -> 2:1.10.3-9.git667d6d1.fc24
  docker-v1.10-migrator 2:1.10.3-54.gite03ddb8.fc24 -> 2:1.10.3-9.git667d6d1.fc24
  dracut 044-21.fc24 -> 044-18.git20160108.fc24
  dracut-config-generic 044-21.fc24 -> 044-18.git20160108.fc24
  dracut-live 044-21.fc24 -> 044-18.git20160108.fc24
  dracut-network 044-21.fc24 -> 044-18.git20160108.fc24
  efibootmgr 14-3.fc24 -> 0.12-3.fc24
  efivar-libs 30-4.fc24 -> 0.23-1.fc24
  elfutils-default-yama-scope 0.167-1.fc24 -> 0.166-2.fc24
  elfutils-libelf 0.167-1.fc24 -> 0.166-2.fc24
  elfutils-libs 0.167-1.fc24 -> 0.166-2.fc24
  emacs-filesystem 1:25.1-2.fc24 -> 1:25.0.94-1.fc24
  etcd 2.3.3-1.fc24 -> 2.2.5-5.fc24
  expat 2.1.1-2.fc24 -> 2.1.1-1.fc24
  fedora-release 24-2 -> 24-1
  fedora-repos 24-3 -> 24-1
  findutils 1:4.6.0-7.fc24 -> 1:4.6.0-3.fc24
  flannel 0.5.5-6.fc24 -> 0.5.5-5.fc24
  fuse-libs 2.9.7-1.fc24 -> 2.9.4-4.fc24
  gawk 4.1.3-8.fc24 -> 4.1.3-3.fc24
  gettext 0.19.8.1-2.fc24 -> 0.19.7-4.fc24
  gettext-libs 0.19.8.1-2.fc24 -> 0.19.7-4.fc24
  glib2 2.48.2-1.fc24 -> 2.48.1-1.fc24
  glibc 2.23.1-11.fc24 -> 2.23.1-7.fc24
  glibc-all-langpacks 2.23.1-11.fc24 -> 2.23.1-7.fc24
  glibc-common 2.23.1-11.fc24 -> 2.23.1-7.fc24
  glusterfs 3.8.5-1.fc24 -> 3.8.0-0.2.rc2.fc24
  glusterfs-client-xlators 3.8.5-1.fc24 -> 3.8.0-0.2.rc2.fc24
  glusterfs-fuse 3.8.5-1.fc24 -> 3.8.0-0.2.rc2.fc24
  glusterfs-libs 3.8.5-1.fc24 -> 3.8.0-0.2.rc2.fc24
  gmp 1:6.1.1-1.fc24 -> 1:6.1.0-2.fc24
  gnupg2 2.1.13-2.fc24 -> 2.1.11-3.fc24
  gnutls 3.4.16-1.fc24 -> 3.4.12-1.fc24
  gpgme 1.6.0-3.fc24 -> 1.4.3-7.fc24
  gssproxy 0.5.1-3.fc24 -> 0.5.0-4.fc24
  guile 5:2.0.13-1.fc24 -> 5:2.0.11-9.fc24
  info 6.1-3.fc24 -> 6.1-2.fc24
  ipcalc 0.1.8-1.fc24 -> 0.1.6-2.fc24
  iputils 20160308-3.fc24 -> 20160308-2.fc24
  json-glib 1.2.2-1.fc24 -> 1.2.0-1.fc24
  kernel 4.8.6-201.fc24 -> 4.5.5-300.fc24
  kernel-core 4.8.6-201.fc24 -> 4.5.5-300.fc24
  kernel-modules 4.8.6-201.fc24 -> 4.5.5-300.fc24
  krb5-libs 1.14.4-4.fc24 -> 1.14.1-6.fc24
  kubernetes 1.2.0-0.26.git4a3f9c5.fc24 -> 1.2.0-0.20.git4a3f9c5.fc24
  kubernetes-client 1.2.0-0.26.git4a3f9c5.fc24 -> 1.2.0-0.20.git4a3f9c5.fc24
  kubernetes-master 1.2.0-0.26.git4a3f9c5.fc24 -> 1.2.0-0.20.git4a3f9c5.fc24
  kubernetes-node 1.2.0-0.26.git4a3f9c5.fc24 -> 1.2.0-0.20.git4a3f9c5.fc24
  libarchive 3.2.2-1.fc24 -> 3.1.2-17.fc24
  libassuan 2.4.3-1.fc24 -> 2.4.2-2.fc24
  libbasicobjects 0.1.1-29.fc24 -> 0.1.1-28.fc24
  libblkid 2.28.2-1.fc24 -> 2.28-2.fc24
  libcap-ng 0.7.8-1.fc24 -> 0.7.7-4.fc24
  libcephfs1 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  libcollection 0.7.0-29.fc24 -> 0.7.0-28.fc24
  libcurl 7.47.1-9.fc24 -> 7.47.1-4.fc24
  libfdisk 2.28.2-1.fc24 -> 2.28-2.fc24
  libgcc 6.2.1-2.fc24 -> 6.1.1-2.fc24
  libgcrypt 1.6.6-1.fc24 -> 1.6.4-2.fc24
  libgomp 6.2.1-2.fc24 -> 6.1.1-2.fc24
  libgpg-error 1.24-1.fc24 -> 1.21-2.fc24
  libicu 56.1-5.fc24 -> 56.1-4.fc24
  libidn 1.33-1.fc24 -> 1.32-2.fc24
  libini_config 1.3.0-29.fc24 -> 1.2.0-28.fc24
  libksba 1.3.5-1.fc24 -> 1.3.4-1.fc24
  libmnl 1.0.4-1.fc24 -> 1.0.3-11.fc24
  libmount 2.28.2-1.fc24 -> 2.28-2.fc24
  libnfsidmap 0.26-6.rc4.fc24 -> 0.26-4.2.fc24
  libnl3 3.2.28-3.fc24 -> 3.2.27-3.fc24
  libpath_utils 0.2.1-29.fc24 -> 0.2.1-28.fc24
  libpng 2:1.6.26-1.fc24 -> 2:1.6.21-2.fc24
  libpsl 0.13.0-3.fc24 -> 0.13.0-1.fc24
  librados2 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  libradosstriper1 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  librbd1 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  libref_array 0.1.5-29.fc24 -> 0.1.5-28.fc24
  librgw2 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  libselinux 2.5-9.fc24 -> 2.5-3.fc24
  libselinux-python 2.5-9.fc24 -> 2.5-3.fc24
  libselinux-python3 2.5-9.fc24 -> 2.5-3.fc24
  libselinux-utils 2.5-9.fc24 -> 2.5-3.fc24
  libsemanage 2.5-5.fc24 -> 2.5-2.fc24
  libsemanage-python 2.5-5.fc24 -> 2.5-2.fc24
  libsemanage-python3 2.5-5.fc24 -> 2.5-2.fc24
  libsepol 2.5-8.fc24 -> 2.5-3.fc24
  libsmartcols 2.28.2-1.fc24 -> 2.28-2.fc24
  libsolv 0.6.24-1.fc24 -> 0.6.20-3.fc24
  libsss_idmap 1.14.2-1.fc24 -> 1.13.4-3.fc24
  libsss_nss_idmap 1.14.2-1.fc24 -> 1.13.4-3.fc24
  libsss_sudo 1.14.2-1.fc24 -> 1.13.4-3.fc24
  libstdc++ 6.2.1-2.fc24 -> 6.1.1-2.fc24
  libtasn1 4.9-1.fc24 -> 4.8-1.fc24
  libtool-ltdl 2.4.6-12.fc24 -> 2.4.6-11.fc24
  libuuid 2.28.2-1.fc24 -> 2.28-2.fc24
  libxkbcommon 0.6.1-1.fc24 -> 0.5.0-4.fc24
  linux-firmware 20160923-68.git42ad5367.fc24 -> 20160526-65.git80d463be.fc24
  lua 5.3.3-2.fc24 -> 5.3.2-3.fc24
  lvm2 2.02.150-2.fc24 -> 2.02.150-1.fc24
  lvm2-libs 2.02.150-2.fc24 -> 2.02.150-1.fc24
  mokutil 1:0.3.0-2.fc24 -> 1:0.2.0-4.fc24
  ncurses 6.0-6.20160709.fc24 -> 6.0-5.20160116.fc24
  ncurses-base 6.0-6.20160709.fc24 -> 6.0-5.20160116.fc24
  ncurses-libs 6.0-6.20160709.fc24 -> 6.0-5.20160116.fc24
  nettle 3.2-3.fc24 -> 3.2-2.fc24
  nfs-utils 1:1.3.4-1.rc2.fc24 -> 1:1.3.3-8.rc5.fc24
  nspr 4.13.1-1.fc24 -> 4.12.0-1.fc24
  nss 3.27.0-1.2.fc24 -> 3.23.0-1.2.fc24
  nss-softokn 3.27.0-1.0.fc24 -> 3.23.0-1.0.fc24
  nss-softokn-freebl 3.27.0-1.0.fc24 -> 3.23.0-1.0.fc24
  nss-sysinit 3.27.0-1.2.fc24 -> 3.23.0-1.2.fc24
  nss-tools 3.27.0-1.2.fc24 -> 3.23.0-1.2.fc24
  nss-util 3.27.0-1.0.fc24 -> 3.23.0-1.0.fc24
  oci-register-machine 0-2.4.git352a2a2.fc24 -> 0-1.1.git7d4ce65.fc24
  openssh 7.2p2-13.fc24 -> 7.2p2-6.fc24
  openssh-clients 7.2p2-13.fc24 -> 7.2p2-6.fc24
  openssh-server 7.2p2-13.fc24 -> 7.2p2-6.fc24
  openssl 1:1.0.2j-1.fc24 -> 1:1.0.2h-1.fc24
  openssl-libs 1:1.0.2j-1.fc24 -> 1:1.0.2h-1.fc24
  ostree 2016.12-1.fc24 -> 2016.5-3.fc24
  ostree-grub2 2016.12-1.fc24 -> 2016.5-3.fc24
  pcre 8.39-6.fc24 -> 8.38-11.fc24
  policycoreutils 2.5-13.fc24 -> 2.5-5.fc24
  policycoreutils-python 2.5-13.fc24 -> 2.5-5.fc24
  policycoreutils-python-utils 2.5-13.fc24 -> 2.5-5.fc24
  policycoreutils-python3 2.5-13.fc24 -> 2.5-5.fc24
  python 2.7.12-6.fc24 -> 2.7.11-4.fc24
  python-cephfs 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  python-libs 2.7.12-6.fc24 -> 2.7.11-4.fc24
  python-rados 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  python-rbd 1:10.2.2-2.fc24 -> 1:10.2.0-2.fc24
  python2-pysocks 1.5.6-4.fc24 -> 1.5.6-3.fc24
  python3 3.5.2-3.fc24 -> 3.5.1-7.fc24
  python3-jinja2 2.8-7.fc24 -> 2.8-5.fc24
  python3-libs 3.5.2-3.fc24 -> 3.5.1-7.fc24
  python3-pyserial 3.1.1-1.fc24 -> 2.7-5.fc24
  python3-pysocks 1.5.6-4.fc24 -> 1.5.6-3.fc24
  python3-requests 2.10.0-2.fc24 -> 2.10.0-1.fc24
  python3-sssdconfig 1.14.2-1.fc24 -> 1.13.4-3.fc24
  python3-urllib3 1.15.1-3.fc24 -> 1.15.1-1.fc24
  rpcbind 0.2.3-11.rc1.fc24 -> 0.2.3-10.rc1.fc24
  rpm-ostree 2016.11-1.fc24 -> 2015.11-2.fc24
  runc 1:0.1.1-3.git57b9972.fc24 -> 1:0.0.9-0.3.git94dc520.fc24
  screen 4.4.0-4.fc24 -> 4.3.1-4.fc24
  selinux-policy 3.13.1-191.20.fc24 -> 3.13.1-190.fc24
  selinux-policy-targeted 3.13.1-191.20.fc24 -> 3.13.1-190.fc24
  skopeo 0.1.14-5.git550a480.fc24 -> 0.1.11-1.fc24
  sqlite 3.13.0-1.fc24 -> 3.11.0-3.fc24
  sqlite-libs 3.13.0-1.fc24 -> 3.11.0-3.fc24
  sssd-client 1.14.2-1.fc24 -> 1.13.4-3.fc24
  strace 4.14-1.fc24 -> 4.11.0.163.9720-2.fc24
  sudo 1.8.18p1-1.fc24 -> 1.8.16-3.fc24
  system-python-libs 3.5.2-3.fc24 -> 3.5.1-7.fc24
  systemd 229-16.fc24 -> 229-8.fc24
  systemd-container 229-16.fc24 -> 229-8.fc24
  systemd-libs 229-16.fc24 -> 229-8.fc24
  systemd-udev 229-16.fc24 -> 229-8.fc24
  tzdata 2016i-1.fc24 -> 2016d-1.fc24
  util-linux 2.28.2-1.fc24 -> 2.28-2.fc24
  vim-minimal 2:7.4.1868-1.fc24 -> 2:7.4.1718-1.fc24
  xfsprogs 4.5.0-2.fc24 -> 4.5.0-1.fc24
Removed:
  bubblewrap-0.1.3-2.fc24.x86_64
  container-selinux-2:1.10.3-54.gite03ddb8.fc24.x86_64
  docker-common-2:1.10.3-54.gite03ddb8.fc24.x86_64
  fedora-release-atomichost-24-2.noarch
  fuse-2.9.7-1.fc24.x86_64
  gobject-introspection-1.48.0-1.fc24.x86_64
  libev-4.20-2.fc24.x86_64
  libreport-filesystem-2.7.2-1.fc24.x86_64
  libsigsegv-2.10-10.fc24.x86_64
  libverto-libev-0.2.6-6.fc24.x86_64
  mdadm-3.4-2.fc24.x86_64
  mpfr-3.1.5-1.fc24.x86_64
  nss-pem-1.0.2-2.fc24.x86_64
  python2-requests-2.10.0-2.fc24.noarch
  python2-urllib3-1.15.1-3.fc24.noarch
  python3-dateutil-1:2.5.2-2.fc24.noarch
  python3-gobject-base-3.20.1-1.fc24.x86_64
  skopeo-containers-0.1.14-5.git550a480.fc24.x86_64
Added:
  GeoIP-1.6.9-2.fc24.x86_64
  GeoIP-GeoLite-data-2016.05-1.fc24.noarch
  docker-selinux-2:1.10.3-9.git667d6d1.fc24.x86_64
  gnupg2-smime-2.1.11-3.fc24.x86_64
  hawkey-0.6.3-2.fc24.x86_64
  libhif-0.2.2-4.fc24.x86_64
  libsecret-0.18.5-1.fc24.x86_64
  libtalloc-2.1.6-1.fc24.x86_64
  libtevent-0.9.28-1.fc24.x86_64
  libusb-1:0.1.5-7.fc24.x86_64
  libusbx-1.0.21-0.1.git448584a.fc24.x86_64
  libverto-tevent-0.2.6-6.fc24.x86_64
  pinentry-0.9.7-2.fc24.x86_64
  python-requests-2.10.0-1.fc24.noarch
  python-urllib3-1.15.1-1.fc24.noarch
  Run "systemctl reboot" to start a reboot
[fedora@fed-atomic-01 ~]$
```

Reboot the host and verify it is using the previous tree version.

```text
[fedora@fed-atomic-01 ~]$ sudo rpm-ostree status
  TIMESTAMP (UTC)         VERSION   ID             OSNAME            REFSPEC
* 2016-06-15 09:57:04     24.39     2c7d41e8a6     fedora-atomic     fedora-atomic:fedora-atomic/24/x86_64/docker-host
  2016-11-14 20:46:13     24.81     49dd9520a7     fedora-atomic     fedora-atomic:fedora-atomic/24/x86_64/docker-host
[fedora@fed-atomic-01 ~]$ sudo rpm-ostree status -p
============================================================
  * DEFAULT ON BOOT
----------------------------------------
  version    24.39
  timestamp  2016-06-15 09:57:04
  id         2c7d41e8a67931fe21bc92100c59cff8a94c2df5a0e6a1b75957bda141601481.0
  osname     fedora-atomic
  refspec    fedora-atomic:fedora-atomic/24/x86_64/docker-host
============================================================
    NON-DEFAULT ROLLBACK TARGET
----------------------------------------
  version    24.81
  timestamp  2016-11-14 20:46:13
  id         49dd9520a7c537ced9c846c2e2f47643b5f52a22768d944b6d8c1108da38f39e.0
  osname     fedora-atomic
  refspec    fedora-atomic:fedora-atomic/24/x86_64/docker-host
============================================================
[fedora@fed-atomic-01 ~]$
```

## Atomic command line

Atomic includes a new cli called `atomic`, provides a coherent entry point to manage Atomic Hosts. Under the hood `atomic` command is a wrapper that allows an administrator to perform container and host maintenance operations using a unified interface.

```text
usage: atomic [-h] [-v] [--debug] [-y]
              {containers,diff,help,images,host,info,install,mount,pull,push,upload,run,scan,sign,stop,storage,migrate,top,trust,uninstall,unmount,umount,update,verify,version}
              ...

Atomic Management Tool

positional arguments:
  {containers,diff,help,images,host,info,install,mount,pull,push,upload,run,scan,sign,stop,storage,migrate,top,trust,uninstall,unmount,umount,update,verify,version}
                        commands
    containers          operate on containers
    diff                Show differences between two container images, file
                        diff or RPMS.
    images              operate on images
    host                execute Atomic host commands
    install             execute container image install method
    mount               mount container image to a specified directory
    pull                pull latest image from a repository
    push (upload)       push latest image to repository
    run                 execute container image run method
    scan                scan an image or container for CVEs
    sign                Sign an image
    stop                execute container image stop method
    storage (migrate)   manage container storage
    top                 Show top-like stats about processes running in
                        containers
    trust               Manage system container trust policy
    uninstall           execute container image uninstall method
    unmount (umount)    unmount container image
    update              pull latest container image from repository

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show atomic version and exit
  --debug               show debug messages
  -y, --assumeyes       automatically answer yes for all questions
[fedora@fed-atomic-01 ~]$
```

### Container operations

`atomic` can be used for container management operations in several ways, at first sight it looks like a sot of wrapper for `docker`to perform many operations like run, stop, list images, etc, however for an `atomic run` operation it will grab the run LABEL and execute it with no need for the user to pass any parameters. `atomic` implements the command install which instead of jst importing the container image in the host with its corresponding Kubernetes configuration or Systemd unit file.

```text
[fedora@fed-atomic-01 ~]$ sudo atomic run alpine sh
Trying docker.io/library/alpine:latest
Uploading blob sha256:baa5d63471ead618ff91ddfacf1e2c81bf0612bfeb1daf00eb0843a41fbfade3
 0 B / 1.25 KB [---------------------------------------------------------------]
Uploading blob sha256:3690ec4760f95690944da86dc4496148a63d85c9e3100669a318110092f6862f
 0 B / 2.21 MB [---------------------------------------------------------------]
Uploading manifest to image destination
Storing signatures
 2.21 MB / 2.21 MB [===========================================================]docker run -t -i --name alpine alpine sh
/ #
```

`atomic` also allows to manage the installed Docker images.

```text
[fedora@fed-atomic-01 ~]$ sudo atomic images list
   REPOSITORY             TAG      IMAGE ID       CREATED            VIRTUAL SIZE   TYPE
>  docker.io/nginx        latest   05a60462f8ba   2016-11-08 23:41   181.44 MB      Docker
>  docker.io/cockpit/ws   latest   0c8d8b92a26e   2016-11-02 19:14   532.96 MB      Docker
>  docker.io/alpine       latest   baa5d63471ea   2016-10-18 22:31   4.8 MB         Docker

[fedora@fed-atomic-01 ~]$
[fedora@fed-atomic-01 ~]$ sudo atomic images info 0c8d8b92a26e
Image Name: 0c8d8b92a26e
RUN: /usr/bin/docker run -d --privileged --pid=host -v /:/host IMAGE /container/atomic-run --local-ssh
UNINSTALL: /usr/bin/docker run -ti --rm --privileged -v /:/host IMAGE /container/atomic-uninstall
INSTALL: /usr/bin/docker run -ti --rm --privileged -v /:/host IMAGE /container/atomic-install
[fedora@fed-atomic-01 ~]$
```

### Host operations

For host related operation `atomic` acts as a wrapper for `rpm-ostree` allowing the same kind of operations.

```text
[fedora@fed-atomic-01 ~]$ sudo atomic host -h
usage: atomic host [-h]
                   {rollback,status,upgrade,rebase,deploy,unlock,install,uninstall}
                   ...

positional arguments:
  {rollback,status,upgrade,rebase,deploy,unlock,install,uninstall}
                        host commands
    rollback            switch to alternate installed tree at next boot
    status              list information about all deployments
    upgrade             upgrade to the latest Atomic tree if one is available
    rebase              Download and deploy a new origin refspec
    deploy              deploy a specific commit
    unlock              Make the current deployment mutable (for development
                        or a hotfix)
    install             Install a (layered) RPM package
    uninstall           Remove a layered RPM package

optional arguments:
  -h, --help            show this help message and exit
[fedora@fed-atomic-01 ~]$
```

## Cockpit

Cockpit is a remote management interface of Linux hosts, I have written before about Cockpit [here]({% post_url 2015-01-08-managing-your-fedora-server-with-cockpit %}), [here]({% post_url 2015-01-09-how-to-install-cockpit-on-centos-7 %}) and [here]({% post_url 2016-01-12-you-can-run-cockpit-in-photon-os %}). In an Atomic Host, Cockpit can be used to manage docker containers and Kubernetes clusters.

To use Cockpit in Atomic is as simple as `sudo atomic run cockpit/ws`.  

```text
[fedora@fed-atomic-01 ~]$ sudo atomic install cockpit/ws
Using default tag: latest
Trying to pull repository docker.io/cockpit/ws ...
latest: Pulling from docker.io/cockpit/ws

c46df4a5b63b: Pull complete
99ef0b2c8485: Pull complete
59a5aa6b0031: Pull complete
5951d07fb748: Pull complete
5f76b08ca3d3: Pull complete
29128c9a04f7: Pull complete
681bb27149fa: Pull complete
078f3e279249: Pull complete
080f6c78b22e: Pull complete
d174142110f6: Pull complete
2599a2377eca: Pull complete
Digest: sha256:b208b4e05c625837890345f816402f1e08ecc99fb569c29dca876e822dab4dbf
Status: Downloaded newer image for docker.io/cockpit/ws:latest
/usr/bin/docker run -ti --rm --privileged -v /:/host cockpit/ws /container/atomic-install
+ sed -e /pam_selinux/d -e /pam_sepermit/d /etc/pam.d/cockpit
+ mkdir -p /host/etc/cockpit/ws-certs.d
+ chmod 755 /host/etc/cockpit/ws-certs.d
+ chown root:root /host/etc/cockpit/ws-certs.d
+ mkdir -p /host/var/lib/cockpit
+ chmod 775 /host/var/lib/cockpit
+ chown root:wheel /host/var/lib/cockpit
+ /bin/mount --bind /host/etc/cockpit /etc/cockpit
+ /usr/sbin/remotectl certificate --ensure
[fedora@fed-atomic-01 ~]$
[fedora@fed-atomic-01 ~]$ sudo atomic run cockpit/ws
/usr/bin/docker run -d --privileged --pid=host -v /:/host cockpit/ws /container/atomic-run --local-ssh
/usr/bin/docker run -d --privileged --pid=host -v /:/host cockpit/ws /container/atomic-run --local-ssh
5b300986736c4c4085d2c2c898ad3873f94236c0dd35e6689d2d83e2ff52a568
[fedora@fed-atomic-01 ~]$
[fedora@fed-atomic-01 ~]$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
5b300986736c        cockpit/ws          "/container/atomic-ru"   13 seconds ago      Up 13 seconds                           backstabbing_yonath
[fedora@fed-atomic-01 ~]$
```

As you can see even the management applications in Atomic are run as containers.

Hope this article has been helpful to many of you to initiate into Atomic Hosts. Stay tuned for the next part of the series.

-- Juanma
