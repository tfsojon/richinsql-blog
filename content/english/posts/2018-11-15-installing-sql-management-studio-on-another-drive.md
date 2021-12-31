---
title: Installing SQL Management Studio On Another Drive
date: 2018-11-15T07:14:50+01:00
author: Rich
layout: post
permalink: /installing-sql-management-studio-on-another-drive
image: /wp-content/uploads/2018/11/Installing-SQL-Management-Studio-On-Another-Drive-1200x280.png
categories:
  - powershell
tags:
  - application
  - powershell
  - SQL
  - ssd
  - windows
---

So you have that really fast SSD in your development machine or that machine that is provided by your workplace, windows is installed onto it thus that is where the C: resides, the problem is that SSD is only 120GB in size, windows takes up a large chunk of that and once you install a few tools like Visual Studio, SQL Server Developer and SQL Management Studio that lovely SSD is pretty much full.

I ran into this problem recently, but fortunately the machine that I was working on had a second, larger disk in it, granted it was a spinning disk, but I wanted to install a few of the tools I had previously installed onto the C drive onto that disk, mainly to free up some space.

When I looked into the applications that were taking up all the space on that SSD SQL Management Studio was one of the big hitters, coming in at over 2GB on disk, so I set about removing it. The removal went well the problem occurred when I tried to re-install it, for some reason you can't specify where SQL Management Studio is installed. That could be a problem!

After doing a little bit of research, it turns out that you can you [Symbolic links](https://en.wikipedia.org/wiki/Symbolic_link) to &#8220;trick&#8221; windows into installing SQL Management Studio onto another disk by creating the folders that it expects to be on the C: as kind of virtual re-directs to another location, but how do you do that? Well, I wrote a script that will do all the heavy lifting for you.
