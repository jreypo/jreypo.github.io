---
layout: post
title: Howto identify the EMC LUN ID in HP-UX 11iv3 – CLARiiON arrays
date: 2010-06-14
type: post
published: true
status: publish
categories:
- EMC
- HP-UX
- Storage
- Sysadmin
tags:
- 11iv3
- CLARiiON
- EMC
- HP-UX
- inquiry
- Storage
author: juan_manuel_rey
comments: true
---

In my previous [post]({% post_url 2010-05-10-howto-identify-the-emc-lun-id-in-hp-ux-11iv3 %}) about EMC storage I showed a procedure to identify the ID of LUN presented to an 11iv3 host without Powerpath, which is not recommended to be installed on 11.31. I stated that I only did test the procedure with Symmetrix DMX arrays.

**Jean Mesquida**, a reader of this blog and a friend, tried the same procedure with **CLARiiON** cabinets and discovered that it didn't work because the serial number attribute was the same for every disk. After that he performed some tests and provided a similar solution using to the EMC tool inquiry.

The inquiry utility can be downloaded in the following URL:

<ftp://ftp.emc.com/pub/symm3000/inquiry/>

Following are his results, **all credit of this post goes to him** I'm just publishing his work here. I also want to thank my friend Jesus at EMC who confirmed Jean's procedure. Many thanks to both of you, without people like you this blog wouldn't be possible :-)

And now the procedure:

Launch an `inq` against the CLARiiON array.

{% highlight text %}
[hpux-server]root:/ #/usr/local/bin/inq -clariion
Inquiry utility, Version V7.3-891 (Rev 2.0)      (SIL Version V6.5.2.0 (Edit Level 891)
Copyright (C) by EMC Corporation, all rights reserved.
For help type inq -h.

................................................................

-------------------------------------------------------------------------------------------------
DEVICE              :VEND    :PROD            :REV   :SER NUM    :CAP(kb)      :VLU :CLUN:State
-------------------------------------------------------------------------------------------------
...
/dev/rdsk/c7t0d1    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :    83886080:   1:  2a:ASSIGNED
/dev/rdsk/c7t0d2    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :    83886080:   2:  2b:ASSIGNED
/dev/rdsk/c7t0d3    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   3:  2c:ASSIGNED
/dev/rdsk/c7t0d4    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   4:  2d:ASSIGNED
/dev/rdsk/c7t0d5    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   5:  2e:ASSIGNED
/dev/rdsk/c7t0d6    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   6:  2f:ASSIGNED
/dev/rdsk/c7t0d7    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :   419430400:   7:  30:ASSIGNED
/dev/rdsk/c8t0d1    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :    83886080:   1:  2a:ASSIGNED
/dev/rdsk/c8t0d2    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :    83886080:   2:  2b:ASSIGNED
/dev/rdsk/c8t0d3    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   3:  2c:ASSIGNED
/dev/rdsk/c8t0d4    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   4:  2d:ASSIGNED
/dev/rdsk/c8t0d5    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   5:  2e:ASSIGNED
/dev/rdsk/c8t0d6    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   6:  2f:ASSIGNED
/dev/rdsk/c8t0d7    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :   419430400:   7:  30:ASSIGNED
/dev/rdsk/c9t0d1    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :    83886080:   1:  2a:ASSIGNED
/dev/rdsk/c9t0d2    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :    83886080:   2:  2b:ASSIGNED
/dev/rdsk/c9t0d3    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   3:  2c:ASSIGNED
/dev/rdsk/c9t0d4    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   4:  2d:ASSIGNED
/dev/rdsk/c9t0d5    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   5:  2e:ASSIGNED
/dev/rdsk/c9t0d6    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   6:  2f:ASSIGNED
/dev/rdsk/c9t0d7    :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :   419430400:   7:  30:ASSIGNED
/dev/rdsk/c10t0d1   :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :    83886080:   1:  2a:ASSIGNED
/dev/rdsk/c10t0d2   :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :    83886080:   2:  2b:ASSIGNED
/dev/rdsk/c10t0d3   :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   3:  2c:ASSIGNED
/dev/rdsk/c10t0d4   :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   4:  2d:ASSIGNED
/dev/rdsk/c10t0d5   :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   5:  2e:ASSIGNED
/dev/rdsk/c10t0d6   :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :     5242880:   6:  2f:ASSIGNED
/dev/rdsk/c10t0d7   :DGC     :CX4-240WDR5     :HP03  :Ch2 CONT   :   419430400:   7:  30:ASSIGNED
...
{% endhighlight %}

We are going to use the last disk (`c10t0d7`). Take a look at the CLUN column, this column gives the LUN ID (`0x30=48` for instance) on the Clariion array. So is Jean's understanding, and I fully agree with him, that `c10t0d7` disk match the 48 LUN on this CLARiiON array.

Finally, and to be more accurate, I modified the title of the other post to reflect that is only for Symmetrix arrays.

Juanma.
