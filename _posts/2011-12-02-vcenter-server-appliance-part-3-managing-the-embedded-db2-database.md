---
title: vCenter Server Appliance. Part 3 - Managing the embedded DB2 database
date: 2011-12-02
type: post
classes: wide
published: true
status: publish
categories:
- Virtualization
- VMware
tags:
- IBM DB2
- SLES 11
- SuSE Linux
- vCenter Server
- vCenter Server Appliance
- VCSA
- VMware
- vSphere 5
author: juan_manuel_rey
comments: true
---

In this last post about the vCenter Server Appliance we will see a glimpse on how to manage the embedded database that comes bundle with the VCSA.

First I must say that **I AM NOT A DB2 ADMIN**. I got this info by playing with the VCSA in my homelab, digging a bit into the DB2 documentation and googling a lot. Use the information provided in this post at your own risk.

If you still want to risk the integrity of your precious appliance please keep reading :-)

### "Disassembling" the database installation

Before starting to launch commands against the database we need to know a bit about it. Since this is SuSE Linux check the `rpm` packages installed.

[![](/assets/images/check_db2_version.png "Check DB2 installed version")]({{site.url}}/assets/images/check_db2_version.png)

Now that we now it is DB2 Express version 9.7.2-1, list the files in the package.

[![](/assets/images/list_db2_rpm_files.png "List DB2 rpm files")]({{site.url}}/assets/images/list_db2_rpm_files.png)

This is very interesting, the package doesn't contain the database but the installation files. The reason for this is very simple, by default DB2 is not installed in the appliance. The Web UI gives you the option to use an Oracle external database or an embedded one.

When you select **embedded** and click **Save Settings** is when vCSA starts the installation and configuration of the database. Change to `/opt/db2/db2expc_9.7.2_install` and look at the contents.

There are four are files, the same showed by the `rpm` command.

-   `db2exc_972_LNX_x86_64.tar.gz` - The DB2 database itself.
-   `do_db2_install` - The installation script.
-   `db2_create_script.sql` - SQL script used by the installation script to create the vCenter database and the schema.
-   `db2expc.rsp` - An answer file used during the installation.

Feel free to take a more detailed look at the files.

Now move to the parent directory, `/opt/db2`, here you will find the installation directory and two links named `current` and `home`. The first will always point to the latest installed version and the second to the home directory for the db2 user. We'll see more about this user later.

[![](/assets/images/opt_db2_directory.png "/opt/db2 directory contents")]({{site.url}}/assets/images/opt_db2_directory.png)

Change to `current` and the database directory structure will show up. There is a `logs` symlink, this link point the installation log directory.

[![](/assets/images/logs_symlink.png)]({{site.url}}/assets/images/logs_symlink.png)

The log file is called `db2install.history` again my advice is to review this log file, along with the installation files it can be a real learning experience about the DB2 installation process.

### Identifying the database

OK we know how the database was installed now we need to know how it works. Check for the DB2 processes.

[![](/assets/images/check_db2_processes.png "Check DB2 processes")]({{site.url}}/assets/images/check_db2_processes.png)

Apart from root there are three other users:

-   `db2inst1`
-   `db2fenc1`
-   `dasusr1`

Look for these users in `/etc/passwd`.

[![](/assets/images/db2_users.png "DB2 users")]({{site.url}}/assets/images/db2_users.png)

The user `db2inst1` is the only one who has a login shell, this is the database admin user.

The home directory for the three users is the same that symlink `home` pointed at in `/opt/db2`. This is where the DB2 environment is loaded
from.

Make yourself `db2inst1` to load the DB2 environment. For the majority of the operations we will use the `db2` command. `db2` is the **IBM DB2 Command Line Processor**, it runs SQL statements against the database and it can be used in interactive mode, command mode and batch processing mode.

First thing is to know which DB2 version is installed. Use the `db2ls` command.

[![](/assets/images/dbls2_command.png)]({{site.url}}/assets/images/dbls2_command.png)

As you can see the VCSA is running IBM DB2 9.7.0 FixPack 2. The Install Path is also displayed.

Next is checking DB2 database manager. Use the `db2start` command to check if the manager is running.

[![](/assets/images/check_db2_db_manager.png "Check DB2 database manager")]({{site.url}}/assets/images/check_db2_db_manager.png)

Try to stop the manager with `db2stop`.

[![](/assets/images/db2stop_fails.png)]({{site.url}}/assets/images/db2stop_fails.png)

Since the vCenter database is active the operation is not allowed, to force the stop use `db2stop force`.

[![](/assets/images/stop_db2_db_manager.png "Stop DB2 database manager")]({{site.url}}/assets/images/stop_db2_db_manager.png)

Then start again the database manager.

[![](/assets/images/start_db2_db_manager.png "Start DB2 database manager")]({{site.url}}/assets/images/start_db2_db_manager.png)

We are going now to get the running instances. There are two commands to perform this operation, `db2ilist` and `db2 get instance`.

[![](/assets/images/get_db2_instance.png "Get DB2 instance")]({{site.url}}/assets/images/get_db2_instance.png)

As you probably know many databases can be created within the same instance so we are going to list the databases created.

[![](/assets/images/get_db2_directory_details.png "List created DB's")]({{site.url}}/assets/images/get_db2_directory_details.png)

As expected only one database is created and its name is VCDB... Surprise!

In a DB2 installation we can also list the active databases, of course in the vCSA appliance only one will be active.

[![](/assets/images/list_active_databases.png "List active databases")]({{site.url}}/assets/images/list_active_databases.png)

Open a connection to the database and retrieve connection state.

[![](/assets/images/get_db_connection_state.png "Get database connection state")]({{site.url}}/assets/images/get_db_connection_state.png)

Once the connection is established we can get detailed information about the database, using again the `db2` command line processor.

[![](/assets/images/get_vcdb_config.png "Get VCDB database configuration")]({{site.url}}/assets/images/get_vcdb_config.png)

List the tablespaces of the database.

[![](/assets/images/list_db_tablespaces.png "List database tablespaces")]({{site.url}}/assets/images/list_db_tablespaces.png)

There are many more options available within db2 utility, I'll let up to you to investigate them further.

### Querying the DB2 database

The final part of our trip is to interrogate the DB2 database. We will use the `isql` utility, that comes bundles with the VCSA, to perform a few basic SQL queries. This tool is part of the [unixODBC](http://www.unixodbc.org/) project, you can find more about it in their website.

And again we will use the db2 command line processor.

#### isql

You don't need `db2inst1` user to use `isql`, being `root` will suffice. To connect to the vCenter database first we need vc user credentials. This is not a system user but a database one.

To get vc user password list the contents of `/etc/vmware-vpx/embedded_db.cfg`.

[![](/assets/images/get_vc_user_password.png "Get vc user password")]({{site.url}}/assets/images/get_vc_user_password.png)

The `EMB_DB_PASSWORD` variable contains the password.

Open a connection to the database passing the database ID, user and password as arguments.

[![](/assets/images/open_connection_to_db2.png "Open connection to database")]({{site.url}}/assets/images/open_connection_to_db2.png)

Now we will interrogate the database tables. Please take into account that in my installation these tables are empty since this a lab environment, in a production one they will be populated.

If you want to know which tables are created have a look at the SQL file `VCDB_db2.sql`. This file is in the vCenter Server media, the Windows one, in the **vCenter-Server\\dbschema** folder. This file is used by the Windows-based vCenter to create the database schema during the installation process when it is connected to an IBM DB2 database.

[![](/assets/images/vcdb_db2_sql_script.png "VCDB DB2 SQL script from Windows vCenter")]({{site.url}}/assets/images/vcdb_db2_sql_script.png)

Following are a couple of SQL commands you can use. Feel free to investigate the above file, I found it very helpful to understand how he vCenter database is constructed.

Get contents of `vpx_product` and `vpx_version` tables.

[![](/assets/images/get_vpxproduct_vpxversion_tables.png "Get vpx_product and vpx_version tables")]({{site.url}}/assets/images/get_vpxproduct_vpxversion_tables.png)

Get the virtual datacenter ID, contained in the `vpx_datacenter` table.

[![](/assets/images/get_virtual_dc_id.png "Get Virtual Datacenter ID")]({{site.url}}/assets/images/get_virtual_dc_id.png)

#### db2 command line processor

Make yourself `db2inst1` user and launch the `db2` shell.

[![](/assets/images/launch_db2_shell.png "Launch db2 shell")]({{site.url}}/assets/images/launch_db2_shell.png)

Connect to the database using the same `connect to VCDB` statement we saw in the previous section.

Now we can run our SQL queries. In `db2` there is no need to end the command with `;` as we did in `isql`.

For the tables you need to prefix the tables with `vc`, the owner of the tables.

[![](/assets/images/vc_tables_prefix.png "Prefix tables with owner, vc")]({{site.url}}/assets/images/vc_tables_prefix.png)

Or set the schema at the beginning.

[![](/assets/images/set_db_schema.png "Set DB schema for operations")]({{site.url}}/assets/images/set_db_schema.png)

And with this we are done with the vCenter Server Appliance series. Hope it will be of help for any of you my dear readers. Please feel free to comment with questions, corrections or any additional tip.

Juanma.
