---
title: Vagrant ssh in Windows 10 from WSL
date: 2017-04-18
type: post
classes: wide
published: true
status: publish
categories:
- DevOps
- Microsoft
- Windows 10
tags:
- DevOps
- Vagrant
- Bash
- Windows Subsystem for Linux
- WSL
- Microsoft
- Windows 10
- Bash on Ubuntu on Windows
author: juan_manuel_rey
comments: true
---

Using [Vagrant](https://www.vagrantup.com/) from WSL is perfectly possible as long as you use the Windows version and the `Vagrantfile` is hosted somewhere under `/mnt/c`. You can run `vagrant.exe up --provider virtualbox` and it will work.

```
$ vagrant.exe up --provider virtualbox
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Checking if box 'hashicorp/precise64' is up to date...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 4.2.0
    default: VirtualBox Version: 5.1
==> default: Mounting shared folders...
    default: /vagrant => C:/Users/jurey/Documents/workspace/vagrant-vms/precise64
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
```

The problem appears when you try to ssh your box with `vagrant.exe ssh`.

```
$ vagrant.exe ssh
`ssh` executable not found in any directories in the %PATH% variable. Is an
SSH client installed? Try installing Cygwin, MinGW or Git, all of which
contain an SSH client. Or use your favorite SSH client with the following
authentication information shown below:

Host: 127.0.0.1
Port: 2222
Username: vagrant
Private key: C:/Users/jurey/Documents/workspace/vagrant-vms/precise64/.vagrant/machines/default/virtualbox/private_key
```

Of course WSL comes with the `ssh` client installed but `vagrant.exe` is a Windows program and it looks under the Windows PATH for ssh.exe and of course there is none. Many people have been solving this by using Vagrant from Git Bash or by adding the Git for Windows bin folder to the Windows PATH variable, I wouldn't recommend this last one since it can get you into some issues with overlapping names for Windows and unix-like binaries. Of course you can use PuTTY or any other SSH client for Windows to do open an SSH session with the vagrant box using the provided credentials. For me none of those solutions were fine since my final goal is to use WSL as my main shell in Windows 10. 

To solved it I coded a small shell script that allow me to seamlessly ssh into the vagrant box. It support with single and with multbox configurations. For the multi box configurations it accepts the name of the box as argument, for single box however there is a caveat, it will work as long as the box is named `default` if it is a different name then pass the name of the box as argument. 

```bash
#!/bin/bash

pwd_alias () { echo "$PWD"; }
get_port () {
        SSH_PORT="`/mnt/c/HashiCorp/Vagrant/bin/vagrant.exe ssh-config $1 | grep Port | awk '{ print $2 }' | dos2unix`"
        echo $SSH_PORT
}

if [[ "$1" == "" ]]; then
  VM="default"
  ssh -p `get_port` -i `pwd_alias`/.vagrant/machines/default/virtualbox/private_key vagrant@127.0.0.1
else
  ssh -p `get_port` -i `pwd_alias`/.vagrant/machines/$1/virtualbox/private_key vagrant@127.0.0.1
fi
``` 

Finally I have added this script and the one from the Docker post to a new [WSL Tools](https://github.com/jreypo/WSL-Tools) repo on Github and will be adding there more scripts and workarounds for WSL. 

Please comment if you have any tip around using Vagrant from WSL. 

-- Juanma