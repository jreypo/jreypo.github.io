---
title: My Docker workflow in Windows Subsystem for Linux
date: 2017-04-17
type: post
classes: wide
published: true
status: publish
categories:
- Containers
- Microsoft
- Windows 10
tags:
- Docker
- Docker Toolbox
- Bash
- Windows Subsystem for Linux
- WSL
- Microsoft
- Windows 10
- Bash on Ubuntu on Windows
author: juan_manuel_rey
comments: true
---

This will be the first of a small series of articles around my usage of WSL in Windows 10 as my main shell. This time I will explain how I use Docker from WSL, first of all lets put this clear: There is no way to use native docker engine on WSL, it just user space and there is no Linux kernel. Knowing that from the beginning when I decided to move all the way to Windows I had to consider the different options and adjust my workflow to them.

I tried [Docker for Windows](https://www.docker.com/docker-windows) and although the experience is very good and is tightly integrated into the OS I decided against it because of its dependency on Hyper-V. I have nothing against Hyper-V as hypervisor on the server, quite the contrary since I work everyday on Azure and love it, but having Hyper-V enabled will deny me the possibility of using other virtualization software like Virtualbox. And to be honest on the desktop I prefer Virtualbox since it works better with Vagrant, also I use VMware Workstation for other virtualization needs. That left with only one option, [Docker Toolbox](https://www.docker.com/products/docker-toolbox). Docker Toolbox installation will get you Docker client, Docker Compose and Docker Machine utilities, the Windows versions of them and fully supported by Docker.

## Docker Machine

In the first versions of WSL there was no way to call Windows excecutables from the Bash shell, but during one of the Insider builds the interoperability feature was introduced and now WSL can see and execute tools like `ipconfig.exe` from Bash. Thanks to this feature I found that was feasible to use `docker-machine.exe` for creating, starting and managing the created machines.

```
[jurey@trantor]-[~]
$ docker-machine.exe start default
Unable to translate current working directory. Using C:\Users\jurey
Starting "default"...
(default) Check network to re-create if needed...
(default) Waiting for an IP...
Machine "default" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
[jurey@trantor]-[~]
$ docker-machine.exe ls
Unable to translate current working directory. Using C:\Users\jurey
NAME        ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
default     -        virtualbox   Running   tcp://192.168.99.100:2376           v17.03.1-ce
openshift   -        virtualbox   Stopped                                       Unknown
[jurey@trantor]-[~]  
$
```

## Docker Client

With the check on Docker Machine I moved to Docker client. Docker Toolbox comes with the Docker client, but as I said the Windows version of it. Instead I wanted to use the Linux version because I wanted a near native Linux eperience and also the Linux version provides command completion which is nice to have. There are several options to get the Docker client into WSL.

- Get the `docker` binary.
- Install `docker.io` package from Ubuntu repositories.
- Install `docker-ce` from official Docker repositories

I choose the third option since it will provide me with the latest version of the binaries and will be in sync with the versions provided by Docker Toolbox. There is a minor issue, the `docker-ce` package gets you also the engine but I didn't mind. The installation process for `docker-ce` is pretty well explained in [Docker official documentation](https://docs.docker.com/engine/installation/linux/ubuntu/).

After this the next step was to get my docker client configured, usually the best way to do do it would be to execute `docker-machine.exe env default` and the needed commands will appear but since `docker-machine.exe` is a Windows command it will get out the syntax for Windows CMD. If you try to use the `--shell bash` option it will show the syntax for Bash but with the Windows path format. This is because Docker Toolbox for Windows comes with the Docket Quickstart Terminal that uses the Bash from Git for Windows and it understands Windows paths.

To circumvent that I cmae up with a small script called `config-docker.sh` to configure the proper variables for my Linux docker client, put it on `$HOME/bin` and aliased it as `docker` on my `.bashrc` file, this way if I need to manage conttainers in a remote host I can quickly unlias it and use the `docker` binary to connect to the host. If you have named your Docker machine differently you must adjust accordingly the `DOCKER_MACHINE_NAME` variable.

```bash
#!/bin/bash

MACHINE_IP=`'/mnt/c/Program Files/Docker Toolbox/docker-machine.exe' ls 2>/dev/null | grep default | awk '{ print $5}'`

export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="$MACHINE_IP"
export DOCKER_CERT_PATH="/mnt/c/Users/$USER/.docker/machine/machines/default"
export DOCKER_MACHINE_NAME="default"

eval /usr/bin/docker $@
```

The solution worked very well until I needed to mount a volume to one of the containers. The issue is that Docker Machine mounts `C:\Users` on the Docker host on `//c`, however from WSL when you expose a volume from `/mnt/c` it does not understand that path. I experimented a bit with `sed` and found this [post](https://jakob.soy/blog/2017/run-docker-from-wsl/) from [Jakob Jarosch](https://twitter.com/foxylion) which gave me the solution for this. Based on his script I replaced the line `eval /usr/bin/docker $@` with this.

```bash
DOCKER_COMMAND=`echo -n "$@" | sed -E 's/\/mnt\/([a-z])\//\/\/\1\//g'`
eval /usr/bin/docker $DOCKER_COMMAND
```

This solution allow me to seamlessly mount any volume as long as it is somewhere under `/mnt/[a-z]`.

Finally I decided to mask those `Unable to translate current working directory. Using C:\Users\jurey` message, this is very a very common message when you execute Windows binaries from WSL and although it do not represent a real error wrong it can be a bit annoying. So instead of launching `docker-machine.exe` directly I have the command aliased with `STDERR` redirected to `/dev/null`.

```
alias docker-machine='/mnt/c/Program\ Files/Docker\ Toolbox/docker-machine.exe 2>/dev/null'
```

## Docker Compose

For Docker Compose I created a similar script to configure the Docker variables using the Linux native `docker-compose` command however couldn't make `docker-compose up` work as I wanted and I am still investigating and for now is the only part I am executing from a Powershell prompt instead of WSL. Will continue to work and post an update as soon as I make it work.

Hope this is helpful if any of you wants to use Docker from Windows Subsystem for Linux. Please comment if you have any tip or have had any issues. 

-- Juanma
