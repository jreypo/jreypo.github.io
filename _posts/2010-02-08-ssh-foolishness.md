---
layout: post
title: SSH foolishness
date: 2010-02-08
type: post
published: true
status: publish
categories:
- HP-UX
- Sysadmin
tags:
- commands
- HP-UX
- networking
- ssh
- sysadmin
- systems administration
author: juan_manuel_rey
comments: true
---

Today it's been one of those days that you wish to hide very deep under your desk.

For a couple of days I've been battling with a 11.31 server over its [`sshd`](http://www.docs.hp.com/en/5992-4213/index.html) configuration, somehow this was the only node which refused the + `public-key` SSH authentication method from my management server and always asked for root password, that means I could't run scripts remotely with a convenient for loop over the sever list, no remote tests, no cron tasks from the management node against it, etc; and that's unacceptable for me.

I almost wiped out the config of the target server and recreated it from scratch using the `sshd_config` file of a working 11.31 node as my starting point and still the damn server asked for a password. I was desperate, looking through the config file one time after another, checked file permissions, running the same test from other servers with same result.

Finally I ask a colleague if he could review my `sshd_config` file, at first look he found nothing wrong and then he performed some test and MAGIC!! it worked. I asked him about his "guru trick" and he said 'Dude, your root home had 777 permissions' 'WHAT?!?!?!?'

I was terribly embarrassed, one of the first tasks I do after install any HP-UX box is to move the root home from `*/*` to `*/root*` and in this almost newly deployed 11.31 I didn't change the permissions. Of course I quickly checked all my servers just in case and fortunately for me none of them had a misconfigured `sshd`.

Two days completely lost fighting against my foolishness, got out of bed on the wrong side this morning.

Oh I almost forgot... thanks [Javi](http://es.linkedin.com/pub/francisco-javier-bravo-d%C3%ADaz/12/2b3/324), you're the best :-)

Juanma.
