---
title: Database Migration Using DbaTools
date: 2020-02-15T10:30:45+01:00
author: Rich
layout: post
permalink: /database-migration-using-dbatools
categories:
  - powershell
tags:
  - PowerShell
  - dba
  - dbatools 
  - Server
  - SQL

---

This post is mainly for me to look back on in the future and see how this was done, so I will keep it short. What I needed was a way to migrate an entire instance from one availability group to another, on two new servers with an entirely new availability group name, the test instance wouldn't have been that much of a problem as there is only around 10 databases on it however the production instance houses many many more databases with various logins and agent jobs for lots of different things that would have taken a really long time to move one by one. 

So, I decided to use DbaTools to do most of the heavy lifting for me and wrote a script to do the migration work, you can see what I came up with below. 

<script src="https://gist.github.com/BonzaOwl/5519b78200df5840b35a0c8422f58367.js"></script>

Hopefully someone else will find it useful, but remember always test before running something like this, additionally as this is a gist, it is a living thing so will be improved and or changed if something doesn't work for me, but if you see something that could be done better I would love to hear how you would have gone about it.