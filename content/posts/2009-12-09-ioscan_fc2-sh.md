---
title: ioscan_fc2.sh
date: 2009-12-09 00:15:00 +0100
tags:
- hp-ux
- scripting
showComments: true
---

My first post is about one small, but great, piece of software I found some time ago in Olivier's site [Mayoxide](http://www.mayoxide.com/). This script is named `ioscan_fc2.sh` and you can find it in the software/toolbox area.

Basically with this script you can obtain a small and comprehensive report of every agile disk device in a 11.31 system. You can put the script in verbose mode with the -v switch and obtain more detailed info, you can also query for a single disk (`-D`) or a single LUN (`-H`). A few examples will be better to show how it works.

I find it very useful to set an alias for this script on the root's profile of every server in which I have the script.

```text
[root@ignite] / # alias ifc
alias ifc='/usr/local/scripts/ioscan_fc2.sh'
[root@ignite] / #
```

Show every disk:

```text
[root@prod01] ~ # ifc | grep rdisk
/dev/rdisk/disk1     0x600508e0000000004911c7e407303805 ONLINE     64000/0xfa00/0x0     136     round_robin  8
/dev/rdisk/disk28    0x600508b40006cb7000006000094b0000 ONLINE     64000/0xfa00/0x2f    2       least_cmd_load 8
/dev/rdisk/disk29    0x600508b40006cb700000600009340000 ONLINE     64000/0xfa00/0x30    6       least_cmd_load 8
/dev/rdisk/disk30    0x600508b40006cb700000600009370000 ONLINE     64000/0xfa00/0x31    4       least_cmd_load 8
/dev/rdisk/disk31    0x600508b40006cb7000006000093e0000 ONLINE     64000/0xfa00/0x32    52      least_cmd_load 8
/dev/rdisk/disk32    0x600508b40006cb700000600009480000 ONLINE     64000/0xfa00/0x33    2       least_cmd_load 8
/dev/rdisk/disk37    0x600508b40006cb700000600009a90000 ONLINE     64000/0xfa00/0x34    4       least_cmd_load 8
/dev/rdisk/disk38    0x600508b40006cb700000600009a30000 ONLINE     64000/0xfa00/0x35    7       least_cmd_load 8
/dev/rdisk/disk43    0x600508b40006cb700000600009520000 ONLINE     64000/0xfa00/0x36    4       least_cmd_load 8
/dev/rdisk/disk52    0x600508b40006cb7000006000095a0000 ONLINE     64000/0xfa00/0x37    10      least_cmd_load 8
/dev/rdisk/disk53    0x600508b40006cb700000600009570000 ONLINE     64000/0xfa00/0x38    10      least_cmd_load 8
/dev/rdisk/disk60    0x600508b40006cb700000600009660000 ONLINE     64000/0xfa00/0x39    2       least_cmd_load 8
/dev/rdisk/disk61    0x600508b40006cb700000600009610000 ONLINE     64000/0xfa00/0x3a    3       least_cmd_load 8
/dev/rdisk/disk62    0x600508b40006cb700000600009690000 ONLINE     64000/0xfa00/0x3b    76      least_cmd_load 8
/dev/rdisk/disk77    0x600508b40006cb700000600009750000 ONLINE     64000/0xfa00/0x3c    1       least_cmd_load 8
/dev/rdisk/disk78    0x600508b40006cb700000600009700000 ONLINE     64000/0xfa00/0x3d    1       least_cmd_load 8
/dev/rdisk/disk95    0x600508b40006cb7000006000097d0000 ONLINE     64000/0xfa00/0x3e    2       least_cmd_load 8
/dev/rdisk/disk96    0x600508b40006cb7000006000097a0000 ONLINE     64000/0xfa00/0x3f    72      least_cmd_load 8
/dev/rdisk/disk97    0x600508b40006cb700000600009800000 ONLINE     64000/0xfa00/0x40    3       least_cmd_load 8
/dev/rdisk/disk98    0x600508b40006cb700000600009890000 ONLINE     64000/0xfa00/0x41    20      least_cmd_load 8
/dev/rdisk/disk107   0x600508b40006cb7000006000098c0000 ONLINE     64000/0xfa00/0x42    1       least_cmd_load 8
/dev/rdisk/disk108   0x600508b40006cb7000006000098f0000 ONLINE     64000/0xfa00/0x43    1       least_cmd_load 8
/dev/rdisk/disk114   0x600508b40006cb700000600009be0000 ONLINE     64000/0xfa00/0x45    3       least_cmd_load 8
/dev/rdisk/disk119   0x600508b40006cb700000600009de0000 ONLINE     64000/0xfa00/0x46    3       least_cmd_load 8
/dev/rdisk/disk122   0x600508b40006cb700000600009e40000 ONLINE     64000/0xfa00/0x47    32      least_cmd_load 8
/dev/rdisk/disk127   0x600508b40006cb700000600009ef0000 ONLINE     64000/0xfa00/0x48    11      least_cmd_load 8
[root@prod01] ~ #
```

Single device query:

```text
[root@prod01] ~ # ifc -D /dev/rdisk/disk127

disk                 wwid                               state      lun_hw_path          size_gb load_bal     max_q_depth
/dev/rdisk/disk127   0x600508b40006cb700000600009ef0000 ONLINE     64000/0xfa00/0x48    11      least_cmd_load 8
 0/3/0/0/0/0.0x50001fe15010bf4a.0x4019000000000000 ACTIVE     (LUN # 25, Flat Space Addressing)
 0/3/0/0/0/0.0x50001fe15010bf4e.0x4019000000000000 STANDBY    (LUN # 25, Flat Space Addressing)
 0/3/0/0/0/1.0x50001fe15010bf4b.0x4019000000000000 ACTIVE     (LUN # 25, Flat Space Addressing)
 0/3/0/0/0/1.0x50001fe15010bf4f.0x4019000000000000 STANDBY    (LUN # 25, Flat Space Addressing)
[root@prod01] ~ #
```

Verbose mode:

```text
[root@ignite] / # ifc -v

disk                 wwid                               state      lun_hw_path          size_gb load_bal     max_q_depth
/dev/rdisk/disk3     0x600508e000000000ecc83792ea772803 ONLINE     64000/0xfa00/0x0     135     round_robin  8
 0/2/1/0.0x32877ea9237c8ec.0x0 ACTIVE     (LUN # 0, Peripheral Addressing)
scope                                                   vg_holder
"/escsi/esdisk/0x0/HP      /IR Volume       /HP01"      /dev/vg00                               

disk                 wwid                               state      lun_hw_path          size_gb load_bal     max_q_depth
/dev/rdisk/disk10    0x600508b40006cb700000600008bb0000 ONLINE     64000/0xfa00/0x29    300     least_cmd_load 8
 0/3/0/0/0/0.0x50001fe15010bf4a.0x4001000000000000 ACTIVE     (LUN # 1, Flat Space Addressing)
 0/3/0/0/0/0.0x50001fe15010bf4e.0x4001000000000000 STANDBY    (LUN # 1, Flat Space Addressing)
 0/3/0/0/0/1.0x50001fe15010bf4b.0x4001000000000000 ACTIVE     (LUN # 1, Flat Space Addressing)
 0/3/0/0/0/1.0x50001fe15010bf4f.0x4001000000000000 STANDBY    (LUN # 1, Flat Space Addressing)
scope                                                   vg_holder
"/escsi/esdisk/0x0/HP      /HSV210          /6110"      /dev/vgignite
[root@ignite] / #
```

As you can see, with `ioscan_fc2.sh` (aliased as `ifc` in my examples), a lot of useful information about the storage stack of an HP-UX can be obtained. Yes someone can say that the info is obtained with HP-UX commands and that's it is completely true but this script gets the info in a more elegant way and more important, at least for me, in one command.

Olivier has done a great work with this script, I recommend his [blog](http://omasse.blogspot.com/) and his software site, they are a must for every HP-UX System Administrator.

With his permission here it is a modified version that shows the VG
holder with the verbose switch:

```bash
#!/bin/sh
#
# ioscan_fc2.sh
#
# Gives out a comprehensive report of all agile disk devices on on HP-UX 11iv3 system
#
# N.B. This is still beta. I don't have enough 11.31 servers available to test
#      the script to its full extent.
#
#
# (c) 2008 Olivier S. Masse, omasse ~at~ mayoxide ~dot~ com
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
#
#
version="@(#) v0.22 2009/02/09"

verbose=0

function usage
{
 typeset myself=$(basename $0)
 echo
 echo "ioscan_fc2 ${version}"
 echo
 echo "Usage: ${myself} [-v] [-H lun_hwpath | -D agile_dsf ...]"
 echo
 echo "Examples:"
 echo "   ${myself}"
 echo "   ${myself} -H 64000/0xfa00/0xa"
 echo "   ${myself} -D /dev/disk/disk73"
 echo
 exit 1
}

if [ ! "$(uname -r)" = "B.11.31" ]
then
 echo "Tested only on B.11.31, sorry."
 exit 1
fi

if [ "$1" ]
then
 case "$1" in
 -v) shift; verbose=1;;
 -H) shift; [ "$1" = "" ] &amp;&amp; usage || desired_hwpath=${1};;
 -D) shift; [ "$1" = "" ] &amp;&amp; usage || desired_disk=${1};;
 *)  usage;;
 esac
fi

scsimgr_cmd="scsimgr get_attr all_lun -a device_file -a hw_path -a state -a capacity -a block_size -a wwid -a load_bal_policy -a max_q_depth -p"
[ ! "${desired_hwpath}" = "" ] &amp;&amp; scsimgr_cmd="scsimgr get_attr -H ${desired_hwpath} -a device_file -a hw_path -a state -a capacity -a block_size -a wwid -a load_bal_policy -a max_q_depth -p"
[ ! "${desired_disk}" = "" ] &amp;&amp; scsimgr_cmd="scsimgr get_attr -D ${desired_disk} -a device_file -a hw_path -a state -a capacity -a block_size -a wwid -a load_bal_policy -a max_q_depth -p"

eval ${scsimgr_cmd} | grep rdisk | while IFS=":" read device_file hw_path state capacity block_size wwid load_bal_policy max_q_depth
do
 if [ "${capacity}" = "" ]   # capacity is nul if device was unpresented
 then
 size_gb="???"
 else
 echo "crap" | awk '{printf("%i\n", '"${capacity}"' * '"${block_size}"' / 1024 / 1024 / 1024);}' | read size_gb
 fi
 echo
 printf "%-20s %-34s %-10s %-20s %-7s %-12s %-12s\n" disk wwid state lun_hw_path size_gb load_bal max_q_depth
 printf "%-20s %-34s %-10s %-20s %-7s %-12s %-12s\n" "${device_file}" "${wwid}" "${state}" "${hw_path}" "${size_gb}" "${load_bal_policy}" "${max_q_depth}"

 ioscan -kFm hwpath -H ${hw_path} | while IFS=":" read crap lunpath crap
 do
 scsimgr get_attr -H ${lunpath} -a state -p | read lunpath_state
 scsimgr get_attr -H ${lunpath} -a lunid | grep "current =" | read crap crap lunpath_lun
 printf "%55s %-10s %-30s\n" "${lunpath}" "${lunpath_state}" "${lunpath_lun}"
 done | sort -k 1,1

 if [ ${verbose} -eq 1 ]
 then
 printf "%-55s %-40s\n" scope vg_holder
 scsimgr ddr_name -D ${device_file} rev | tail -1 | read ddr_name
 [ "${ddr_name}" = "" ] &amp;&amp; ddr_name="(unknown)"

 echo ${device_file} | sed 's/rdisk/disk/g' | read cooked_device_file
 if [ -c ${device_file}_p2 ]
 then
 pvdisplay ${cooked_device_file}_p2 2&gt;/dev/null | awk '/VG Name/ {print $3}' | read vg
 else
 pvdisplay ${cooked_device_file} 2&gt;/dev/null | awk '/VG Name/ {print $3}' | read vg
 fi
 [ "${vg}" = "" ] &amp;&amp; vg="(unknown)"
 printf "%-55s %-40s\n" "${ddr_name}" "${vg}"
 fi
done
```

See you next time.

Juanma.
