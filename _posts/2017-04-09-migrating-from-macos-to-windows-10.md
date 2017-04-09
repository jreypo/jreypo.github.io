---
layout: post
title: Migrating from OS X to Windows 10
date: 2017-04-09
type: post
published: true
status: publish
categories:
- Personal
- Microsoft
- Windows 10
tags:
- Personal
- Microsoft
- Career
- Windows 10
author: juan_manuel_rey
comments: true
---

When I joined Microsoft it was clear to me that I will have to switch to Windows as my main OS. Besides of using Windows 7 on Bootcamp during a couple of weeks I have not used Windows at work since I left HP in 2012 and at home I have always used either Linux, OS X or another BSD operating system.

The more logical path it would have been to get a new MBP and use it in my new job, I love OS X (or macOS like is called now), is a fantastic OS, there are some really great applications for Mac like Omnigraffle or Alfred and also Microsoft is doing great things these days around collaboration on the Mac platform so I will feel at home with my beloved BSD operating system and the Office suite. There was a problem in my reasoning, coming from a 2015 MacBook Pro Retina 15" the latest MacBook Pro didn't look like a worhy spend of my money and to be honest in my mind that laptiop is everything but pro. With that in mind I decided to look at what Microsoft had to offer me. 

# The hardware

Thankfully Microsoft has a great internal laptop offering, most of my colleagues use a Surface device and the ones who don't prefer one of the X1 Lenovo Ultrabooks. Those are great devices, specially if you are looking for a light Ultrabook or a 2-in-1, however I do not mind having a bit of extra weight if I get more power and got used to the 15" MacBook Pro with Retina display so I decided for the [Dell Precision 5510](http://www.dell.com/us/business/p/precision-m5510-workstation/pd) , which is the mobile workstation version of the XPS 15. 

This beast comes with a Core i7-6820HQ processor, 32GB of DDR4 RAM (the processor supports up to 64GB!!! :O ), NVIDIA Quadro M1000M graphics card, 256GB SSD and a beautiful 15.4" 4K InfinityEdge display. I have added a Samsung EVO 960 500GB NVMe disk as main disk for the OS, apps, virtual machines, etc, and left the 256GB SATA SSD as extra space for Dropbox, OneDrive and the likes. And very important, it has ports! Yes it has 2 USB 3.0, SD card slot, full fledge HDMI and a USB-C/Thunderbolt 3 port in all their glory :D

The laptop flies, the form factor and weight is almost the same as the MBPr so I did not noticed any difference. The only issue I have found for now is the trackpad. Apple trackpads are second to none, and although the one in the Dell is very good and gets the job done it does not get to the level of the Apple ones. Anyway when I need more precision, like when I am diagramming on Visio, I got a [Microsoft Bluetooth Designer Mouse](https://www.microsoft.com/accessories/en-us/products/mice/designer-bluetooth-mouse/7n5-00001) which is kind of awesome. 

# The software

Originally I thought about installing Fedora and use a Windows 10 virtual machine for those corner cases where I would need to use Windows, it didn't go well. When you are working in a Microsoft centric environment and you are in a software engineering, sysadmin or another internal technical role, using Linux as your main OS can be challenging but not impossible. However if you are in field customer facing role like mine the burden is almost impossible to bear, I am very dependent of Skype for Business, Outlook, and OneDrive for my job, and like [Scott Lowe has found](http://blog.scottlowe.org/2017/04/03/linux-migration-corp-collab-pt3/) in his series of posts about corporate collaboration in Linux, the alternatives are very limited for those programs in the Linux world. Also I deeply hate Libreoffice, actually being able to use Office was one of the reasons I switched from OpenBSD & Linux to Mac at home. Add to that GNOME scaling in HiDPi displays is everything but great and the choice was clear, to settle on Windows 10. 

I wiped out my Fedora 25 installation and re-install the latest Windows 10 build at the moment. I was pleasantly surprise with Windows 10, the OS is fast and it has some features that sets it apart from the previous Windows versions.

- **Virtual desktops**: In OS X and Linux I was a heavy virtual desktops/workspaces user, is the way I organize my world. One for the mail, one for the terminal, another for coding and tech work, an so on. The only objection I have is the lack of being able to configure an app start in an specific desktop like in OS X, I hope the Windows 10 team implements this feature in the coming releases but in the meantime I am looking for a third party option.
- **Touchpad gestures**: With the Anniversary Update of Windows 10 a lot of new gestures came to the OS, like left-right four finger swipe to change desktop and up swipe to get an expose-like view of your apps and virtual desktops.

In general I was very satisfied with Windows 10. But there was still one really big issue I had to overcome, there is no Unix shell on Windows.

## WSL to the rescue!

As I said the main concern of using Windows for me was the absence of a real Unix shell. I depend on Bash, I honestly do, like many (all?) Unix guys I use the shell for everything, from searching files within my filesystem to use git, vagrant, docker, almost all. During my years at HP I used [Cygwin](https://www.cygwin.com/) and more recently the Bash shell that comes with [Git for Windows](https://git-for-windows.github.io/) on my Windows VMs, those are very useful solutions but in the end they are no more than Win32 versions of the original Unix command so none of them provide a native environment. 

Back in March during [BUILD](http://www.buildwindows.com/) Microsoft announced Windows Subsystem for Linux, a.k.a. "Bash on Ubuntu on Windows" as it is known. WSL is not a VM, or a container or a glorified version of Cygwin, many people think is one of those, WSL is 

- First rule of the WSL club: Use an Insider build. This is a bit less important now that Windows 10 Creators Update is out in the wild, but for months this was the only way to get all the features that make WSL great.
- Second rule of the WSL club: Do not use Windows console for WSl. The Windows team has introduced a lot of enhancements in the Windows console but it doesn't get near to iTerm2 or Terminator. I searched a lot until I found WSLtty, more on this later. 

I am preparing a few posts about how I use WSl as part of my personal workflow, specifically around the small hacks and workarounds I use. In the meantime I suggest you go to the [Windows Command Line Tools for Developers blog](https://blogs.msdn.microsoft.com/commandline/) and keep up to date with Windows Subsystem for Linux, specifically this [post](https://blogs.msdn.microsoft.com/commandline/learn-about-bash-on-windows-subsystem-for-linux/) is very useful with great content to learn the internals of WSL. Also I recommend the following video in [Microsoft Channel 9 from Scott Hanselman during BUILD](https://channel9.msdn.com/Events/Build/2016/C906) conference talking with some of the developers who have created WSL.

<iframe src="https://channel9.msdn.com/Events/Build/2016/C906/player" width="960" height="540" allowFullScreen frameBorder="0"></iframe>

## Apps that will ease the transition

During these months I have been discovering some apps that helped me to have a better user experience in Windows.

- [Wox](https://github.com/Wox-launcher/Wox) - Wox is a launcher for Windows, the only suitable replacement for [Alfred](https://www.alfredapp.com/) I have found.
- [Visual Studio Code](https://code.visualstudio.com/) - On OS X and Linux I was using Atom, although Atom is of course available in Windows ans is still one of the best editors available when I joined Microsoft I decided to embrace VS Code. The experience is superb and the best is that you can set the Bash shell from the Windows Subsystem for Linux as the default terminal.
- [Hyper](https://hyper.is/) - Hyper is a cross-platform terminal emulator written in Javascript, supports tabs, screen splitting, just to name a few features and has a huge plethora of plugins. I use Hyper as my default terminal for Powershell. 
- [WSLtty](https://github.com/mintty/wsltty) - As I mentioned before for WSL I do not use the Windows console, instead I discovered WSLTTY. It is a version of MinTTY, the default terminal for Cygwin and Git Bash, for WSL. It provides a full Unix, xterm-compatible terminal emulation experience, supports 256 colors, has fullscreen mode, theming and many more. It doesn't support tabs but since I do all my work inside a tmux session this not an issue for me.
- [Tweeten](http://tweeten.xyz/) - Like TweetDeck but better, also for Windows 10 it offers a native app which is better IMHO than the Chrome extension.  

I am always looking for useful apps, both in the consumer and the productivity areas so please if you know more put them on the comments. 

Finally, this is how the beast looks now.

WSLtty in fullscreen mode with tmux:

[![](/images/trantor_screen.jpg "Terminal session with tmux")]({{site.url}}/images/trantor_screen.jpg)

The decorations ;-)

[![](/images/trantor_stickers.jpg "My stickers")]({{site.url}}/images/trantor_stickers.jpg)

I hope this post is useful for those folks looking at moving from OS X to Windows 10. Comments are more than welcome.

-- Juanma