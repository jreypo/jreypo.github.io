---
layout: post
title: Fixing my own stupidity and a bunch of DRD errors
date: 2010-04-22 00:25:00.000000000 +02:00
type: post
published: true
status: publish
categories:
- HP-UX
- Itanium
tags:
- DRD
- Dynamic Root Disk
- HP-UX
- idisk
- Itanium
- LVM
author: juan_manuel_rey
---

I was playing this afternoon with DRD in an 11.23 machine and just after launching the clone process I decided to stop it with `Ctrl-c` since I wasn't logging the session and I wanted to do it. The process stopped with an error and I was sent back to the shell.

{% highlight text %}
       * Copying File Systems To New System Image
ERROR:   Exiting due to keyboard interrupt.
       * Unmounting New System Image Clone
       * System image: "sysimage_001" on disk "/dev/dsk/c2t1d0"
ERROR:   Caught signal SIGINT.  This process is running critical code, this signal will be handled shortly.
ERROR:   Caught signal SIGINT.  This process is running critical code, this signal will be handled shortly.
ERROR:   Caught signal SIGINT.  This process is running critical code, this signal will be handled shortly.
ERROR:   Caught signal SIGINT.  This process is running critical code, this signal will be handled shortly.
ERROR:   Unmounting the file system fails.
         - Unmounting the clone image fails.
         - The "umount" command returned  "13". The "sync" command returned  "0". The error messages produced are the following: ""
       * Unmounting New System Image Clone failed with 5 errors.
       * Copying File Systems To New System Image failed with 6 errors.
=======  04/21/10 08:20:19 EDT  END Clone System Image failed with 6 errors. (user=root)  (jobid=ivm-v2)
{% endhighlight %}

I know it is a very bad idea but it's not production server, just a virtual machine I use to perform tests. In fact my stupid behavior gaveme the opportunity to discover and play with a funny and pretty bunch of errors. Here it is how I manage to  resolve it.

I launched again the clone process in preview mode, just in case, and DRD fails with the following error.

{% highlight text %}
[ivm-v2]/ # drd clone -p -v -t /dev/dsk/c2t1d0

=======  04/21/10 08:22:01 EDT  BEGIN Clone System Image Preview (user=root)  (jobid=ivm-v2)

        * Reading Current System Information
        * Selecting System Image To Clone
        * Selecting Target Disk
 ERROR:   Selection of the target disk fails.
          - Selecting the target disk fails.
          - Validation of the disk "/dev/dsk/c2t1d0" fails with the following error(s):
          - Target volume group device entry "/dev/drd00" exists. Run "drd umount" before proceeding.
        * Selecting Target Disk failed with 1 error.

=======  04/21/10 08:22:10 EDT  END Clone System Image Preview failed with 1 error. (user=root)  (jobid=ivm-v2)
{% endhighlight %}

It seems that the original process just leaved the image mounted but after trying with `drd umount` just like the DRD output said it didn't worked. The image was only partially created, yeah I created a beautiful mess ;-)

At that point, and in another *clever* movement, instead of simply removing the `drd00` volume group I just deleted `/dev/drd00`... who's da man!! or like we use to say in Spain ¡Con dos cojones!

DRD, of course, failed with a new error.

{% highlight text %}
ERROR:   Selection of the target disk fails.
         - Selecting the target disk fails.
         - Validation of the disk "/dev/dsk/c2t1d0" fails with the following error(s):
         - Target volume group "/dev/drd00" found in logical volume table. "/etc/lvmtab" is corrupt and must be fixed before proceeding.
       * Selecting Target Disk failed with 1 error.
{% endhighlight %}

Well it wasn't so bad. I recreated `/etc/lvmtab` and yes... I fired up my friend Dynamic Root Disk in preview mode.

{% highlight text %}
[ivm-v2]/ # rm -f /etc/lvmtab
[ivm-v2]/ # vgscan -v
Creating "/etc/lvmtab".
vgscan: Couldn't access the list of physical volumes for volume group "/dev/vg00".
Invalid argument
Physical Volume "/dev/dsk/c3t2d0" contains no LVM information

/dev/vg00
/dev/dsk/c2t0d0s2

Following Physical Volumes belong to one Volume Group.
Unable to match these Physical Volumes to a Volume Group.
Use the vgimport command to complete the process.
/dev/dsk/c2t1d0s2

Scan of Physical Volumes Complete.
*** LVMTAB has been created successfully.
*** Do the following to resync the information on the disk.
*** #1.  vgchange -a y
*** #2.  lvlnboot -R
[ivm-v2]/ # lvlnboot -R
Volume Group configuration for /dev/vg00 has been saved in /etc/lvmconf/vg00.conf
[ivm-v2]/ #
[ivm-v2]/ # drd clone -p -v -t /dev/dsk/c2t1d0

=======  04/21/10 08:26:06 EDT  BEGIN Clone System Image Preview (user=root)  (jobid=ivm-v2)

       * Reading Current System Information
       * Selecting System Image To Clone
       * Selecting Target Disk
ERROR:   Selection of the target disk fails.
         - Selecting the target disk fails.
         - Validation of the disk "/dev/dsk/c2t1d0" fails with the following error(s):
         - The disk "/dev/dsk/c2t1d0" contains data. To overwrite this disk use the option "-x overwrite=true".
       * Selecting Target Disk failed with 1 error.

=======  04/21/10 08:26:13 EDT  END Clone System Image Preview failed with 1 error. (user=root)  (jobid=ivm-v2)
{% endhighlight %}

I couldn't believe that. Another error? Why in the hell I got involved with DRD? But I am a Unix Sysadmin, and a stubborn one. Looked at the disk and discovered that it had been partitioned by the first failed DRD cloning process. I just wiped out the whole disk with `idisk` and just in case I used the overwrite option.

{% highlight text %}
[ivm-v2]/ # idisk -p /dev/rdsk/c2t1d0
idisk version: 1.31

EFI Primary Header:
        Signature                 = EFI PART
        Revision                  = 0x10000
        HeaderSize                = 0x5c
        HeaderCRC32               = 0xe19d8a07
        MyLbaLo                   = 0x1
        AlternateLbaLo            = 0x1117732f
        FirstUsableLbaLo          = 0x22
        LastUsableLbaLo           = 0x1117730c
        Disk GUID                 = d79b52fa-4d43-11df-8001-d6217b60e588
        PartitionEntryLbaLo       = 0x2
        NumberOfPartitionEntries  = 0xc
        SizeOfPartitionEntry      = 0x80
        PartitionEntryArrayCRC32  = 0xca7e53ce

  Primary Partition Table (in 512 byte blocks):
    Partition 1 (EFI):
        Partition Type GUID       = c12a7328-f81f-11d2-ba4b-00a0c93ec93b
        Unique Partition GUID     = d79b550c-4d43-11df-8002-d6217b60e588
        Starting Lba              = 0x22
        Ending Lba                = 0xfa021
    Partition 2 (HP-UX):
        Partition Type GUID       = 75894c1e-3aeb-11d3-b7c1-7b03a0000000
        Unique Partition GUID     = d79b5534-4d43-11df-8003-d6217b60e588
        Starting Lba              = 0xfa022
        Ending Lba                = 0x110af021
    Partition 3 (HPSP):
        Partition Type GUID       = e2a1e728-32e3-11d6-a682-7b03a0000000
        Unique Partition GUID     = d79b5552-4d43-11df-8004-d6217b60e588
        Starting Lba              = 0x110af022
        Ending Lba                = 0x11177021

[ivm-v2]/ #
[ivm-v2]/ # idisk -R /dev/rdsk/c2t1d0
idisk version: 1.31
********************** WARNING ***********************
If you continue you will destroy all partition data on this disk.
Do you wish to continue(yes/no)? yes
{% endhighlight %}

Don't know why but I was pretty sure that DRD was going to fail again... and it did.

{% highlight text %}
=======  04/21/10 08:27:02 EDT  BEGIN Clone System Image Preview (user=root)  (jobid=ivm-v2)

       * Reading Current System Information
       * Selecting System Image To Clone
       * Selecting Target Disk
       * The disk "/dev/dsk/c2t1d0" contains data which will be overwritten.
       * Selecting Volume Manager For New System Image
       * Analyzing For System Image Cloning
ERROR:   Analysis of file system creation fails.
         - Analysis of target fails.
         - The analysis step for creation of an inactive system image failed.
         - The default DRD mount point "/var/opt/drd/mnts/sysimage_001/" cannot be used due to the following error(s):
         - The mount point /var/opt/drd/mnts/sysimage_001/ is not an empty directory as required.
       * Analyzing For System Image Cloning failed with 1 error.

=======  04/21/10 08:27:09 EDT  END Clone System Image Preview failed with 1 error. (user=root)  (jobid=ivm-v2)
{% endhighlight %}

After a quick check I found that the original image was mounted.

{% highlight text %}
[ivm-v2]/ # mount
/ on /dev/vg00/lvol3 ioerror=mwdisable,delaylog,dev=40000003 on Wed Apr 21 07:29:37 2010
/stand on /dev/vg00/lvol1 ioerror=mwdisable,log,tranflush,dev=40000001 on Wed Apr 21 07:29:38 2010
/var on /dev/vg00/lvol8 ioerror=mwdisable,delaylog,dev=40000008 on Wed Apr 21 07:29:50 2010
/usr on /dev/vg00/lvol7 ioerror=mwdisable,delaylog,dev=40000007 on Wed Apr 21 07:29:50 2010
/tmp on /dev/vg00/lvol4 ioerror=mwdisable,delaylog,dev=40000004 on Wed Apr 21 07:29:50 2010
/opt on /dev/vg00/lvol6 ioerror=mwdisable,delaylog,dev=40000006 on Wed Apr 21 07:29:50 2010
/home on /dev/vg00/lvol5 ioerror=mwdisable,delaylog,dev=40000005 on Wed Apr 21 07:29:50 2010
/net on -hosts ignore,indirect,nosuid,soft,nobrowse,dev=1 on Wed Apr 21 07:30:26 2010
/var/opt/drd/mnts/sysimage_001 on /dev/drd00/lvol3 ioerror=nodisable,delaylog,dev=40010003 on Wed Apr 21 08:19:46 2010
/var/opt/drd/mnts/sysimage_001/stand on /dev/drd00/lvol1 ioerror=nodisable,delaylog,dev=40010001 on Wed Apr 21 08:19:46 2010
/var/opt/drd/mnts/sysimage_001/tmp on /dev/drd00/lvol4 ioerror=nodisable,delaylog,dev=40010004 on Wed Apr 21 08:19:46 2010
/var/opt/drd/mnts/sysimage_001/home on /dev/drd00/lvol5 ioerror=nodisable,delaylog,dev=40010005 on Wed Apr 21 08:19:46 2010
/var/opt/drd/mnts/sysimage_001/opt on /dev/drd00/lvol6 ioerror=nodisable,delaylog,dev=40010006 on Wed Apr 21 08:19:46 2010
/var/opt/drd/mnts/sysimage_001/usr on /dev/drd00/lvol7 ioerror=nodisable,delaylog,dev=40010007 on Wed Apr 21 08:19:46 2010
/var/opt/drd/mnts/sysimage_001/var on /dev/drd00/lvol8 ioerror=nodisable,delaylog,dev=40010008 on Wed Apr 21 08:19:47 2010
[ivm-v2]/ #
{% endhighlight %}

Had to unmount the filesystems of the image one by one and after almost committing suicide with a rack rail I launched the clone again and without the preview, if I were going to play a stupid role at least it was going to the most stupid one in the world x-)

{% highlight text %}
[ivm-v2]/ # drd clone -x overwrite=true -v -t /dev/dsk/c2t1d0

=======  04/21/10 08:38:22 EDT  BEGIN Clone System Image (user=root)  (jobid=rx260-02)

       * Reading Current System Information
       * Selecting System Image To Clone
       * Selecting Target Disk
       * The disk "/dev/dsk/c2t1d0" contains data which will be overwritten.
       * Selecting Volume Manager For New System Image
       * Analyzing For System Image Cloning
       * Creating New File Systems
ERROR:   Clone file system creation fails.
         - Creating the target file systems fails.
         - Command "/opt/drd/lbin/drdconfigure" fails with the return code 255. The entire output from the command is given below:
         - Start of output from /opt/drd/lbin/drdconfigure:
         -        * Creating LVM physical volume "/dev/rdsk/c2t1d0s2" (0/1/1/0.1.0).
                  * Creating volume group "drd00".
           ERROR:   Command "/sbin/vgcreate -A n -e 4356 -l 255 -p 16 -s 32 /dev/drd00
                    /dev/dsk/c2t1d0s2" failed.

         - End of output from /opt/drd/lbin/drdconfigure
       * Creating New File Systems failed with 1 error.
       * Unmounting New System Image Clone
       * System image: "sysimage_001" on disk "/dev/dsk/c2t1d0"

=======  04/21/10 08:38:46 EDT  END Clone System Image failed with 1 error. (user=root)  (jobid=rx260-02)

[ivm-v2]/ #
{% endhighlight %}

I thought that every possible error was fixed but there it was DRD saying that it failed with a bogus `return code 255`, oh yes very insightful because it's not a 254 or a 256 it is a 255 code and everybody know what it means... Shit! I don't know what it means. Yes it was true, I didn't know what "return code 255" stood for. After doing a small search on ITRC there was only one entry about a similar case, only one. I manage to create a beautiful error, don't you think?

The question is that there was a mismatch between the minor numbers the kernel believed were in use and those really visible in the device files. DRD will always try to use the next free based on the device files and since in my case there was only one in use but the kernel thought there were two in use, one from `vg00` and another one from the failed clone, it failed.

The solution is to cheat the kernel creating a fake group device using the minor number the kernel thinks is in use.

{% highlight text %}
[ivm-v2]/dev # mkdir fake
[ivm-v2]/dev # cd fake
[ivm-v2]/dev/fake # mknod group c 64 0x010000
[ivm-v2]/dev/fake #
{% endhighlight %}

After that I launched DRD and everything went smoothly.

Fortunately everything happened in a test virtual machine and at any step of my frustrating trip through self-generated DRD error I could reset the VM and start over again with a clean system but since the purpose of Dynamic Root Disk is to minimize the downtime of production systems the reboot was not an option, at least no the first in the list.

The credit for the solution goes to [Judit Wathen](http://forums13.itrc.hp.com/service/forums/publicProfile.do?forumId=1&userId=CA1442849&admit=109447627+1271889825841+28353475) from the [Dynamic Root Disk](http://www.docs.hp.com/en/DRD/) Team at HP, continue with your great work :-D

Juanma.
