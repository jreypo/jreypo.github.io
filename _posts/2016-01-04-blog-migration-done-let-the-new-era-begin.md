---
layout: post
title: Blog migration done... Let the new era begin!
date: 2016-01-04
type: post
published: true
status: publish
categories:
- Blog
tags:
- Blog
author: juan_manuel_rey
comments: true
image:
   feature: trantor.png
---

I have finally migrated the last post from my old site in **Wordpress.com** to the new one using [**Jekyll**](https://jekyllrb.com) and hosted on [**Github Pages**](https://pages.github.com). The migration has taken more time than I originally though, using Jekyll migration tools and Wordpress exported XML I was able to get all the posts with their corresponding tags, categories and images. At least that's what it looked like, however when I went through several posts I noticed that many of my posts had the different image links but with the same image name, that's because some time ago I used Windows Live Writer and copy pasted captured images from the clipboard instead of using image files. Also Wordpress writing tool allow you to dynamically resize images.

In the end I had to review all the text, modify all the code blocks with Pygments tags, convert all post links in order to use `post_url` tags, re-download and rename many of the pictures from my old site, fix the links to my GitHub Gists and review the Front Matter for every post. In the meantime I also hit an issue with the Jekyll version used by Github Pages (2.4) and timezones, the issue did not allow the site to build. Thanks to my old shell, sed and awk scripting-fo I was able to speed the process.

The good thing is that since I had to manually review all my posts the quality os better than before, I was able to fix some text errors here and there and thanks to the Jekyll bug I decided to add my site repo to [Travis CI](https://travis-ci.org) and now every time I push a commit to Github the site is built and tested by Travis.

There are many Jekyll themes available, some of them quite clean and simple however I was looking for something clean but a bit more appealing, visually speaking, and I choose [**Minimal Mistakes**](http://mmistakes.github.io/minimal-mistakes/) by [Michael Rose](http://twitter.com/mmistakes). The theme includes support for Disqus and Google Analytics and uses Pygments for the syntax highlight.

Jekyll supports tags however Minimal Mistakes lacked a tag section by default, for me this was a must since I like to organize my posts. Fortunately the theme and Jekyll are very hacker friendly so I put my Google-fu at work and found a nice [post](http://dareneiri.github.io/Jekyll-Themes-and-Tags/) by [**Daren Eiri**](http://twitter.com/dareneiri) explaining how to implement it. After the Tag section, adjusting the font to something I liked and other small changes the site was done.

The next challenge for me was to find a way to share my posts on my social network accounts right after I publish them, in Wordpress the integration with Twitter, Facebook, LinkedIn and Google+ comes out of the box and is very easy to use. However Jekyll does not have it and even if there was a plugin, to be honest I do not know if there is one, Github Pages only allow you to use a limited set of plugins. Then I remember a service I have read about many times in the past but have never used, [**IFTTT**](http://ifttt.com). IFTTT, If This Then This, is a service that allow you to create recipes to automate many tasks interacting with many online services APIs, called channels. With IFTTT I have fully automated the sharing of any new post published in Twitter, Facebook and LinkedIn. Unfortunately Google+ is not very friendly and for now I have not been able to find an easy way to automatically share my posts, for now I'll do it manually until I find it.

And that's it. Let the new era begin!

- Juanma
