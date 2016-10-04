---
layout: post
title: Running OpenStack Rally tests from a Docker container
date: 2016-10-04
type: post
published: true
status: publish
categories:
- OpenStack
- Networking
- Docker
tags:
- OpenStack
- Rally
- docker
- linux
- containers
author: juan_manuel_rey
comments: true
image:
  feature: openstack-banner.jpg
---

[OpenStack Rally](https://wiki.openstack.org/wiki/Rally) is a benchmarking and testing project for OpenStack, it provides a framework for performance, scalability and stress tests. Rally can simulate different use-cases on an existing cloud.

My OpenStack operation teams have Rally deployed in its own virtual machine and with all tests files in an external shared filesystem and versioned with git, which a totally valid and recommendable way of running your Rally tests. However running Rally from a Docker container has also unquestionable benefits:
- Portability - If like me you are always on the road and like to have an immutable rally environment always at hand.
- Integration with your CI/CD infrastructure, like Jenkins.

With the portability in mind I decided to do a learning exercise and dockerize my rally environment, which by the way was on a Centos 7 virtual machine running on VMware Fusion. Rally already provides a Dockerfile and building your own rally Docker container is as simple as:

{% highlight text %}
git clone https://github.com/openstack/rally.git
docker build -t rally-docker .
{% endhighlight %}

However this container is built using Ubuntu 16.04 as the base image and I prefer to use CentOS or Fedora. I have created and publish a Docker container with all the Rally bits from RDO Mitaka repositories, if you just want to test it run the following command.

{% highlight text %}
docker run --name rally-mitaka -t -i -v ~/rally_home:/home/rally jreypo/rally-rdo-mitaka
{% endhighlight %}

To manually build the container clone my [Github repo](https://github.com/jreypo/rally-docker-containers) change to `rally-rdo-mitaka` directory and edit the Dockerfile to make any modifications to suit your needs.

{% highlight dockerfile %}
FROM centos:latest
MAINTAINER Juan Manuel Rey "juanmanuel.reyportal@gmai.com"
RUN yum install -y https://repos.fedorapeople.org/repos/openstack/openstack-mitaka/rdo-release-mitaka-5.noarch.rpm
RUN yum update -y
RUN yum install -y openstack-rally \
                openstack-utils \
                openstack-selinux \
                gcc \
                libffi-devel \
                python-devel \
                openssl-devel \
                gmp-devel \
                libxml2-devel \
                libxslt-devel \
                postgresql-devel \
                redhat-rpm-config \
                wget \
                which

VOLUME ["/home/rally"]
WORKDIR /home/rally

CMD ["bash", "--login"]
RUN rally-manage db create
{% endhighlight %}

Build and run the container.

{% highlight text %}
docker build -t jreypo/rally-rdo-mitaka .
docker run --name rally-mitaka -t -i -v ~/rally_home:/home/rally jreypo/rally-rdo-mitaka
{% endhighlight %}

Once you are in the container bash prompt create a deployment file called `deployment.json`

{% gist jreypo/2e9397dfac8f91d2a893bdb55f20e19e %}

Create the deployment with `rally` command line.

{% highlight text %}
[root@c82cd7302a9e rally]# rally deployment create --file=deployment.json --name=existing
2016-10-04 13:31:27.674 93 INFO rally.deployment.engine [-] Deployment efc2457e-1dff-4edb-bee2-2ea11fb94e68 | Starting:  OpenStack cloud deployment.
2016-10-04 13:31:27.831 93 INFO rally.deployment.engine [-] Deployment efc2457e-1dff-4edb-bee2-2ea11fb94e68 | Completed: OpenStack cloud deployment.
+--------------------------------------+----------------------------+----------+------------------+--------+
| uuid                                 | created_at                 | name     | status           | active |
+--------------------------------------+----------------------------+----------+------------------+--------+
| efc2457e-1dff-4edb-bee2-2ea11fb94e68 | 2016-10-04 13:31:27.579275 | existing | deploy->finished |        |
+--------------------------------------+----------------------------+----------+------------------+--------+
Using deployment: efc2457e-1dff-4edb-bee2-2ea11fb94e68
~/.rally/openrc was updated

HINTS:
* To get your cloud resources, run:
        rally show [flavors|images|keypairs|networks|secgroups]

* To use standard OpenStack clients, set up your env by running:
        source ~/.rally/openrc
  OpenStack clients are now configured, e.g run:
        glance image-list
[root@c82cd7302a9e rally]#
{% endhighlight %}

Test the connection between the rally container and your OpenStack installation by retrieving the configured flavors.

{% highlight text %}
[root@c82cd7302a9e rally]# rally show flavors

Flavors for user `admin` in tenant `admin`:
+----+-----------+-------+----------+-----------+-----------+
| ID | Name      | vCPUs | RAM (MB) | Swap (MB) | Disk (GB) |
+----+-----------+-------+----------+-----------+-----------+
| 1  | m1.tiny   | 1     | 512      | n/a       | 1         |
| 2  | m1.small  | 1     | 2048     | n/a       | 20        |
| 3  | m1.medium | 2     | 4096     | n/a       | 40        |
| 4  | m1.large  | 4     | 8192     | n/a       | 80        |
| 5  | m1.xlarge | 8     | 16384    | n/a       | 160       |
+----+-----------+-------+----------+-----------+-----------+
[root@c82cd7302a9e rally]#
{% endhighlight %}

To run your first test create an execution file, you can grab one from the sample files in Rally repository. In my testing environment I am using the Nova scenario sample file `nova-boot-delete.json`. Launch the test with `rally` command line.

{% highlight text %}
[root@c82cd7302a9e ~]# rally task start nova_boot_delete.json
--------------------------------------------------------------------------------
 Preparing input task
--------------------------------------------------------------------------------

Input task is:

{
    "NovaServers.boot_and_delete_server": [
        {
            "args": {
                "flavor": {
                    "name": "m1.tiny"
                },
                "image": {
                    "name": "cirros"
                },
                "force_delete": false
            },
            "runner": {
                "type": "constant",
                "times": 10,
                "concurrency": 2
            },
            "context": {
                "users": {
                    "tenants": 3,
                    "users_per_tenant": 2
                }
            }
        },
        {
            "args": {
                "flavor": {
                    "name": "m1.tiny"
                },
                "image": {
                    "name": "cirros"
                },
                "auto_assign_nic": true
            },
            "runner": {
                "type": "constant",
                "times": 10,
                "concurrency": 2
            },
            "context": {
                "users": {
                    "tenants": 3,
                    "users_per_tenant": 2
                },
                "network": {
                    "start_cidr": "10.2.0.0/24",
                    "networks_per_tenant": 2
                }
            }
        }
    ]
}

Task syntax is correct :)
2016-10-04 14:28:15.536 204 INFO rally.task.engine [-] Task 0c3f4378-e46e-4903-a1c6-2c42628b06a9 | Starting:  Task validation.
2016-10-04 14:28:15.616 204 INFO rally.task.engine [-] Task 0c3f4378-e46e-4903-a1c6-2c42628b06a9 | Starting:  Task validation of scenarios names.
2016-10-04 14:28:15.619 204 INFO rally.task.engine [-] Task 0c3f4378-e46e-4903-a1c6-2c42628b06a9 | Completed: Task validation of scenarios names.
2016-10-04 14:28:15.619 204 INFO rally.task.engine [-] Task 0c3f4378-e46e-4903-a1c6-2c42628b06a9 | Starting:  Task validation of syntax.
...
{% endhighlight %}

Export your report in HTML format and store on the Docker volume attached to the container.

{% highlight text %}
rally task report 0c3f4378-e46e-4903-a1c6-2c42628b06a9 --html-static --out /home/rally/nova-report.html
{% endhighlight %}

Access the report from your browser.

[![](/images/rally_nova.png)]({{site.url}}/images/rally_nova.png)

As you can see running Rally from a Docker container is pretty easy, keep in mind that my Docker images are very opinionated towards RDO/Red Hat OpenStack releases although it should be valid to be used with any OpenStack distribution. Anyway if you want a more neutral container but still based on Fedora/CentOS I encourage you to use the one created by my colleague [Juanma Parrilla](https://twitter.com/kerbeross), you can check his Github repo [docker-osp-rally](https://github.com/padajuan/docker-osp-rally).

-- Juanma
