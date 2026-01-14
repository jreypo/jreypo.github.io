---
title: 'HP P4000: Generating the CLIQ key file'
date: 2010-11-02
tags:
- hp-servers
- storage
showComments: true
---

As I explained in my [first post]({{< ref "posts/2010-06-10-cliq-the-hp-lefthand-saniq-command-line.md" >}}) about the SAN/iQ command line, to remotely manage a P4000 storage array instead of providing the username/password credentials in every command you can specify an encrypted file which contains the user/password information.

To create this file, known as the key file, just use the `createKey` command and provide the username, password, array IP address or DNS name and the name of the file.

[![](/images/key-file.png "key-file")](/images/key-file.png)

By default the key file is created in the user's home directory, `C:\Documents and Settings\<username>` in Windows XP/2003 and `C:\Users\<username>` in Windows Vista/2008/7.

The file can also be stored in a secure location on the local network, in that case the full path to the key file must be provided.

[![](/images/getclusterinfo.png "getclusterinfo")](/images/getclusterinfo.png)

Of course the main reason to create a key file, apart from ease the daily management, is to provide a valid authentication mechanism for any automation script that you can create using the `cliq`.

Juanma.
