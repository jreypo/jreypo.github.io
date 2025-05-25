---
title: How to integrate SUSE Linux Enterprise 11 with Windows Active Directory
date: 2012-02-01
type: post
classes: wide
published: true
status: publish
categories:
- Linux
- Sysadmin
- Windows
tags:
- Active Directory
- Kerberos
- Linux
- Microsoft
- samba
- SLES11
- SUSE
- sysadmin
- systems administration
- Windows
- yast2
author: juan_manuel_rey
comments: true
---

Getting [SUSE Enterprise Linux](http://www.suse.com/products/server/) integrated with Microsoft Active Directory is much easier than it sounds.

There are a few prerequisites to meet before:

- Samba client must be installed.
- Packages samba-winbind and krb5-client must be installed.
- The primary DNS server must be the Domain Controller.

For this task we will use **YaST2**, the SUSE configuration tool.

YaST2 can be run either in graphical...

[![](/assets/images/screenshot-yast2-control-center-1_thumb.png "YaST2 Control Center")]({{site.url}}/assets/images/screenshot-yast2-control-center-1.png)

...or in text mode.

[![](/assets/images/yast2_text_mode.png "YaST2 text mode")]({{site.url}}/assets/images/yast2_text_mode.png)

I decided to use the text mode since it will be by far the most common use case, anyway in both cases the procedure is exactly the same.

Go to `Network Services` section and later select `Windows Domain Membership`. The `Windows Domain Membership` configuration screen will appear.

In the `Membership` area enter the domain name and configure the options that best suit your environment, including the other sections of the screen.

[![](/assets/images/yast2_windom_config.png "YaST2 Windows Domain configuration")]({{site.url}}/assets/images/yast2_windom_config.png)

I configure it to allow SSH single sign-on, more on this later, and to create a home directory for the user on his first login.

You should take into account the NTP configuration since it’s a critical component in Active Directory authentication.

Select `OK` to acknowledge your selection and a small pop-up will show up to inform that the host is not part of the domain and if you want to join it.

[![](/assets/images/yast2_domain_confirmation.png "YaST2 domain confirmation")]({{site.url}}/assets/images/yast2_domain_confirmation.png)

Next you must enter the password of the domain Administrator.

[![](/assets/images/yast2_domain_admin_password.png "Domain Administrator password")]({{site.url}}/assets/images/yast2_domain_admin_password.png)

And YaST will finally confirm the success of the operation.

[![](/assets/images/yast2_domain_joined.png "Domain join confirmation")]({{site.url}}/assets/images/yast2_domain_joined.png)

At this point the basic configuration is done and the server should be integrated on the Windows Domain.

Under the hood this process has modified several configuration files in order the get the system ready to authenticate against Active Directory:

- `smb.conf`
- `krb5.conf`
- `nsswitch.conf`

## smb.conf

The first is the configuration file for the [`samba`](http://samba.org/) service. As you should know Samba is an open source implementation of the Windows SMB/CIFS protocol, it allows Unix systems to integrate almost transparently into a Windows Domain infrastructure and also provides file and print services for Windows clients.

The file resides in `/etc/samba`. Take a look at the contents of the file, the relevant part is the `global` section.

```
[global]
        workgroup = VJLAB
        passdb backend = tdbsam
        printing = cups
        printcap name = cups
        printcap cache time = 750
        cups options = raw
        map to guest = Bad User
        include = /etc/samba/dhcp.conf
        logon path = \%L\profiles\.msprofile
        logon home = \%L\%U\.9xprofile
        logon drive = P:
        usershare allow guests = No
        idmap gid = 10000-20000
        idmap uid = 10000-20000
        realm = VJLAB.LOCAL
        security = ADS
        template homedir = /home/%D/%U
        template shell = /bin/bash
        winbind refresh tickets = yes
```

## krb5.conf

`krb5.conf` file is the Kerberos daemon configuration file which contains the necessary information for the Kerberos library.

```
jreypo@sles11-01:/etc> cat krb5.conf
[libdefaults]
        default_realm = VJLAB.LOCAL
        clockskew = 300
[realms]
        VJLAB.LOCAL = {
                kdc = dc.vjlab.local
                default_domain = vjlab.local
                admin_server = dc.vjlab.local
        }
[logging]
        kdc = FILE:/var/log/krb5/krb5kdc.log
        admin_server = FILE:/var/log/krb5/kadmind.log
        default = SYSLOG:NOTICE:DAEMON
[domain_realm]
        .vjlab.local = VJLAB.LOCAL
[appdefaults]
        pam = {
                ticket_lifetime = 1d
                renew_lifetime = 1d
                forwardable = true
                proxiable = false
                minimum_uid = 1
        }
jreypo@sles11-01:/etc>
```

## nsswitch.conf

`nsswitch.conf` file as stated by its man page is the *System Databases and Name Service Switch configuration file*. Basically it includes the different databases of the system to look for authentication information when user tries to log into the server.

Have a quick look into the file and you will notice the two fields changed, `passwd` and `group`. In both the `winbind `` option has been added in order to indicate the system to use `winbind`, the Name Service Switch daemon used to resolve NT server names.

```
passwd: compat winbind
group:  compat winbind
```

## SSH single sign-on

Finally we need to test the SSH connection to the host using a user account of the domain. When asked for the login credentials use the `DOMAIN\USER` formula for the user name.

[![](/assets/images/ssh_auth.png "SSH authentication")]({{site.url}}/assets/images/ssh_auth.png)

This kind of integration is very useful, specially for the bigger shops, because you don’t have to maintain the user list of your SLES servers individually, just only the root account since the other accounts can be centrally managed from the Windows Domain.

However there is one issue that must be taken into account, the SSH single sign-on authentication means that anyone with a domain account can log into your Linux servers and we don’t want that.

To prevent this potentially dangerous situation we are going to limit the access only to those groups of users that really need it. I’m going to use the Domain Admins to show you how.

First we need to look for the Domain Admins group ID within our Linux box. Log in as `DOMAIN\Administrator` and use the `id` command to get the user info.

```
VJLAB\administrator@sles11-01:~> id
uid=10000(VJLAB\administrator) gid=10000(VJLAB\domain users) groups=10000(VJLAB\domain users),10001(VJLAB\schema admins),10002(VJLAB\domain admins),10003(VJLAB\enterprise admins),10004(VJLAB\group policy creator owners)
VJLAB\administrator@sles11-01:~>
```

There are several group IDs, for our purposes we need the `VJLAB\domain admins` which is `10002`.

You should be asking yourself, but the GID is not 10002 but 10000? Yes you are right and because of that we need to make some changes at Domain Controller level.

Fire up Server Manager and go to **Roles -> Active Directory Domain Services –> Active Directory Users and Computers –> DOMAIN –> Users**.

[![](/assets/images/server_manager.png "Windows Server Manager")]({{site.url}}/assets/images/server_manager.png)

On the right pane edit the properties of the account you want to be able to access the linux server via SSH. In my case I used my own account **juanma**. In the **Member of** tab select the Domain Admins group and click **Set Primary Group**.

[![](/assets/images/member_of.png "Member of tab")]({{site.url}}/assets/images/member_of.png)

Now we need to modify how the pam daemon manage the authentication. Go back to SLES and edit `/etc/pam.d/sshd` configuration file.

```
#%PAM-1.0
auth     requisite      pam_nologin.so
auth     include        common-auth
account  include        common-account
password include        common-password
session  required       pam_loginuid.so
session  include        common-session
```

Delete the account line and add the following two lines.

```
account  sufficient     pam_localuser.so
account  sufficient     pam_succeed_if.so gid = 10002
```

The `sshd` file should look like this:

```
#%PAM-1.0
auth     requisite      pam_nologin.so
auth     include        common-auth
account  sufficient     pam_localuser.so
account  sufficient     pam_succeed_if.so gid = 10002
password include        common-password
session  required       pam_loginuid.so
session  include        common-session
```

What we did? First eliminated the ability to login via SSH for every user and later we allow the server local users and the Domain Admins to log into the server.

And we are done. Any comment would be welcome as always :-)

Juanma.
