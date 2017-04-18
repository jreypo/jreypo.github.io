---
layout: post
title: Linux LVM commands in HP-UX
date: 2010-02-16
type: post
published: true
status: publish
categories:
- HP-UX
- Linux
tags:
- 11iv3
- commands
- HP-UX
- Linux LVM
- LVM
- lvs
- pvs
- scripts
- vgs
author: juan_manuel_rey
comments: true
---

Some of the features I always liked about the Linux LVM2 implementation are the `lvs`, `vgs` and `pvs` commands. With these simple commands a short list of the LVs, VGs and PVs active on the system can be obtained.

```
www04:~ # vgs
VG            #PV #LV #SN Attr   VSize  VFree
vgwww01         1   6   0 wz--n- 39.75G   6.25G
vgbbdd          3   3   0 wz--n- 94.25G      0
vgsys           1   6   0 wz--n- 34.74G   1.25G
www04:~ #
```

In HP-UX there is nothing similar available, well since HP-UX 11.31 and LVM2 `-F` option has been added to produce a formatted, more script friendly list but is not very human readable.

```
[root@nfscl02] ~ # vgdisplay -vF vg00
vg_name=/dev/vg00:vg_write_access=read,write:vg_status=available:max_lv=255:cur_lv=9:open_lv=9:max_pv=16:cur_pv=1:act_pv=1:max_pe_per_pv=4353:vgda=2:pe_size=32:total_pe=4322:alloc_pe=1539:free_pe=2783:total_pvg=0:total_spare_pvs=0:total_spare_pvs_in_use=0:vg_version=1.0:vg_max_size=2228736m:vg_max_extents=69648
lv_name=/dev/vg00/lvol1:lv_status=available,syncd:lv_size=1856:current_le=58:allocated_pe=58:used_pv=1
lv_name=/dev/vg00/lvol2:lv_status=available,syncd:lv_size=8192:current_le=256:allocated_pe=256:used_pv=1
lv_name=/dev/vg00/lvol3:lv_status=available,syncd:lv_size=1504:current_le=47:allocated_pe=47:used_pv=1
lv_name=/dev/vg00/lvol4:lv_status=available,syncd:lv_size=512:current_le=16:allocated_pe=16:used_pv=1
lv_name=/dev/vg00/lvol5:lv_status=available,syncd:lv_size=8192:current_le=256:allocated_pe=256:used_pv=1
lv_name=/dev/vg00/lvol6:lv_status=available,syncd:lv_size=5024:current_le=157:allocated_pe=157:used_pv=1
lv_name=/dev/vg00/lvol7:lv_status=available,syncd:lv_size=5024:current_le=157:allocated_pe=157:used_pv=1
lv_name=/dev/vg00/lvol8:lv_status=available,syncd:lv_size=8704:current_le=272:allocated_pe=272:used_pv=1
lv_name=/dev/vg00/lv_crash:lv_status=available,syncd:lv_size=10240:current_le=320:allocated_pe=320:used_pv=1
pv_name=/dev/disk/disk1_p2:pv_status=available:total_pe=4322:free_pe=2783:autoswitch=On:proactive_polling=On
[root@nfscl02] ~ #
```

Because of this I decided to write three scripts to emulate the behavior of `vgs`, `lvs` and `pvs` on my HP-UX servers. This scripts take advantage of the mentioned LVM2 `-F` switch so they will not work on HP-UX 11.23 or any other previous versions. If anyones recognize the scripting style is because I grab some parts of the code from Olivier's [ioscan\_fc2.sh](http://jreypo.wordpress.com/2009/12/09/ioscan_fc2-sh/) and adapted them to my needs so credit goes to him also :-)

-   **VGS:** List the volume group on the `/etc/lvmtab` file, if the server is part of a cluster the volume groups active on other nodes will be showed as deactivated. With the `-v` switch single VGs can be queried.

```
root@cldpp01:~# ./vgs.sh
VG         PVs   LVs   Status               Version  VGSize Free
vg00       1     9     available            1.0      135G   77G
vgdpp      1     1     available,exclusive  1.0      49G    0G
vgidbbck               deactivated
root@cldpp01:~# ./vgs.sh -v vgdpp
VG         PVs   LVs   Status               Version  VGSize Free
vgdpp      1     1     available,exclusive  1.0      49G    0G
root@cldpp01:~#
```

The code:

```bash
#!/sbin/sh
#
# vgs.sh - script to emulate the Linux LVM command vgs in HP-UX 11iv3
#
# (C) 2010 - Juan Manuel Rey (juanmanuel.reyportal@gmail.com)
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
#

version="v0.1 2010/02/15"

function usage
{
        echo
        echo "VGS for HP-UX ${version}"
        echo
        echo "Usage: vgs [-v vg_name]"
        echo
        exit 1
}

if [ ! "$(uname -r)" = "B.11.31" ]
then
        echo "VGS for HP-UX only works on HP-UX 11iv3"
        exit 1
fi

if [ "$1" ]
then
        case "$1" in
        -v) shift; [  "$1" = "" ] && usage || vg_name=${1};;
        *)  usage;;
        esac
fi

vg_display="vgdisplay -F"
[ ! "${vg_name}" = "" ] && vg_display="vgdisplay -F ${vg_name}"

printf "%-10s %-5s %-5s %-20s %-8s %-6s %-5s\n" VG PVs LVs Status Version VGSize Free

eval ${vg_display} | while IFS=":" read vgdisplay
do
        echo ${vgdisplay} | cut -d ":" -f 2 | cut -d "=" -f 2 | read status
        if [ "${status}" = "deactivated" ]
        then
                status=deactivated
                vg_size=""
                vg_free=""
        else
                echo ${vgdisplay} | cut -d ":" -f 3 | cut -d "=" -f 2 | read status
                echo ${vgdisplay} | cut -d ":" -f 13 | cut -d "=" -f 2 | read total_pe
                echo ${vgdisplay} | cut -d ":" -f 12 | cut -d "=" -f 2 | read pe_size
                echo ${vgdisplay} | cut -d ":" -f 15 | cut -d "=" -f 2 | read free_pe
                vg_size="`/usr/bin/expr $total_pe \* $pe_size / 1024`G"
                vg_free="`/usr/bin/expr $free_pe \* $pe_size / 1024`G"
        fi
        echo ${vgdisplay} | cut -d ":" -f 1 | cut -d "=" -f 2 | cut -d "/" -f 3 | read vg_name
        echo ${vgdisplay} | cut -d ":" -f 8 | cut -d "=" -f 2 | read pvs
        echo ${vgdisplay} | cut -d ":" -f 5 | cut -d "=" -f 2 | read lvs
        echo ${vgdisplay} | cut -d ":" -f 19 | cut -d "=" -f 2 | read version
        printf "%-10s %-5s %-5s %-20s %-8s %-6s %-5s\n" "${vg_name}" "${pvs}" "${lvs}" "${status}" "${version}" "${vg_size}" "${vg_free}"
done
```


-   **LVS:** Like its Linux counterpart shows a list with every active logical volume. As in `vgs.sh` with the `-v` switch you can ask the list of a specific volume group.

```
root@asoka:/# ./lvs.sh -v vg00
LV                             VG           Status            LVSize Permissions Mirrors Stripes  Allocation
lvol1                          vg00         available,syncd   1G     read,write        0       0  strict,contiguous
lvol2                          vg00         available,syncd   8G     read,write        0       0  strict,contiguous
lvol3                          vg00         available,syncd   1G     read,write        0       0  strict,contiguous
lvol4                          vg00         available,syncd   0G     read,write        0       0  strict
lvol5                          vg00         available,syncd   20G    read,write        0       0  strict
lvol6                          vg00         available,syncd   1G     read,write        0       0  strict
lvol7                          vg00         available,syncd   5G     read,write        0       0  strict
lvol8                          vg00         available,syncd   20G    read,write        0       0  strict
lv_crash                       vg00         available,syncd   9G     read,write        0       0  strict
root@asoka:/#
```

The code:

```bash
#!/sbin/sh
#
# lvs.sh - script to emulate the Linux LVM command lvs in HP-UX 11iv3
#
# (C) 2010 - Juan Manuel Rey (juanmanuel.reyportal@gmail.com)
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
#

version="v0.1 2010/02/15"

function usage
{
        echo
        echo "LVS for HP-UX ${version}"
        echo
        echo "Usage: lvs [-v vg_name]"
        echo
                exit 1
}

if [ ! "$(uname -r)" = "B.11.31" ]
then
        echo "LVS for HP-UX only works on HP-UX 11iv3"
        exit 1
fi

if [ "$1" ]
then
        case "$1" in
        -v) shift; [  "$1" = "" ] && usage || vg_name=${1};;
        *)  usage;;
        esac
fi

lv_list="vgdisplay -vF | grep lv_name"
[ ! "${vg_name}" = "" ] && lv_list="vgdisplay -vF ${vg_name} | grep lv_name"

printf "%-30s %-12s %-17s %-6s %-10s %-7s %-8s %-8s %-7s\n" LV VG Status LVSize Permissions Mirrors Stripes Allocation

eval ${lv_list} | while IFS=":" read lvlist
do
        echo ${lvlist} | cut -d ":" -f 1 | cut -d "/" -f 4 | read lv_name
        echo ${lvlist} | cut -d ":" -f 1 | cut -d "=" -f 2 | read lv_long_name
        lvdisplay -F ${lv_long_name} | cut -d ":" -f 2 | cut -d "/" -f 3 | read vg_name
        lvdisplay -F ${lv_long_name} | cut -d ":" -f 4 | cut -d "=" -f 2 | read lv_status
        lvdisplay -F ${lv_long_name} | cut -d ":" -f 3 | cut -d "=" -f 2 | read lv_perm
        lvdisplay -F ${lv_long_name} | cut -d ":" -f 5 | cut -d "=" -f 2 | read lv_mirrors
        lvdisplay -F ${lv_long_name} | cut -d ":" -f 11 | cut -d "=" -f 2 | read lv_stripes
        lvdisplay -F ${lv_long_name} | cut -d ":" -f 14 | cut -d "=" -f 2 | read lv_allocation
        lvdisplay -F ${lv_long_name} | cut -d ":" -f 8 | cut -d "=" -f 2 | read size_megs
        lv_size="`/usr/bin/expr $size_megs / 1024`G"

        printf "%-30s %-12s %-17s %-6s %-17s %-7s %-2s %-5s\n" "${lv_name}" "${vg_name}" "${lv_status}" "${lv_size}" "${lv_perm}" "${lv_mirrors}" "${lv_stripes}" "${lv_allocation}"
done
```

-   **PVS:** And now the last one. List the activated physical volumes, if a VGs is not active on the current node its PVs wouldn't be shown. Like in `pvs.sh` and `vgs.sh` there is a `-v` switch.

```
root@oracle:~# ./pvs.sh
PV                   VG         Status               PVSize Free
/dev/disk/disk1_p2   vg00       available            135G   48G  
/dev/disk/disk28     vgora      available            1G     0G   
/dev/disk/disk29     vgora      available            5G     0G   
/dev/disk/disk30     vgora      available            3G     0G   
/dev/disk/disk31     vgora      available            51G    1G   
/dev/disk/disk32     vgora      available            1G     0G   
/dev/disk/disk37     vgora      available            3G     0G   
/dev/disk/disk38     vgora      available            6G     0G   /dev/disk/disk43     vgoracli   available            3G     0G   
/dev/disk/disk119    vgoracli   available            2G     0G  
root@oracle:~#
```

And the code:

```bash
#!/sbin/sh
#
# pvs.sh - script to emulate the Linux LVM command pvs in HP-UX 11iv3
#
# (C) 2010 - Juan Manuel Rey (juanmanuel.reyportal@gmail.com)
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
#

version="v0.1 2010/02/15"

function usage
{
        echo
        echo "PVS for HP-UX ${version}"
        echo
        echo "Usage: pvs [-v vg_name]"
        echo
        exit 1
}

if [ ! "$(uname -r)" = "B.11.31" ]
then
        echo "PVS for HP-UX only works on HP-UX 11iv3"
        exit 1
fi

if [ "$1" ]
then
        case "$1" in
        -v) shift; [  "$1" = "" ] && usage || vg_name=${1};;
        *)  usage;;
        esac
fi

pv_list="vgdisplay -vF | grep disk"
[ ! "${vg_name}" = "" ] && pv_list="vgdisplay -vF ${vg_name} | grep disk"

printf "%-20s %-10s %-20s %-6s %-5s\n" PV VG Status PVSize Free

eval ${pv_list} | while IFS=":" read pvlist
do
        echo ${pvlist} | cut -d ":" -f 1 | cut -d "=" -f 2 | read pv_name
        pvdisplay -F ${pv_name} | cut -d ":" -f 2 | cut -d "=" -f 2 | cut -d "/" -f 3 | read vg_name
        pvdisplay -F ${pv_name} | cut -d ":" -f 3 | cut -d "=" -f 2 | read status
        pvdisplay -F ${pv_name} | cut -d ":" -f 8 | cut -d "=" -f 2 | read total_pe
        pvdisplay -F ${pv_name} | cut -d ":" -f 7 | cut -d "=" -f 2 | read pe_size
        pvdisplay -F ${pv_name} | cut -d ":" -f 9 | cut -d "=" -f 2 | read free_pe
        pv_size="`/usr/bin/expr $total_pe \* $pe_size / 1024`G"
        pv_free="`/usr/bin/expr $free_pe \* $pe_size / 1024`G"
        printf "%-20s %-10s %-20s %-6s %-5s\n" "${pv_name}" "${vg_name}" "${status}" "${pv_size}" "${pv_free}"
done
```

These scripts are available also on my [HP-UX Scripts](https://github.com/jreypo/hpux-scripts) Github repository. As always any comments, corrections and/or suggestions are welcome.

Juanma.
