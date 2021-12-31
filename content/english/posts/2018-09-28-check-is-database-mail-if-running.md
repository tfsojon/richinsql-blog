---
title: Check If Database Mail Is Running
date: 2018-09-28T07:46:00+01:00
author: Rich
layout: post
permalink: /check-is-database-mail-if-running
image: /wp-content/uploads/2018/09/Check-If-Database-Mail-Is-Running-1200x280.png
categories:
  - sqlserver
tags:
  - database
  - dba
  - mail
  - SQL
  - SQLAgent
---

So, recently I ran into a problem where I wasn't getting any alerts for any of my SQL Agent Jobs, it didn't take very long to get to the bottom of the problem but upon investigating it turned out that DBMail was actually in a STOPPED state, which was weird, this particular server had been rebooted the day before but SQL Agent was running so why was DBMail stopped? The error log didn't prove too helpful on this one so I used;

```
    Exec msdb.dbo.sysmail_start_sp
```

To get it started again but what if it happened again so I set about trying to figure out if there was a way I could automate checking the state of DBMail and if it was stopped get it running again.

On my travels to finding a solution I ran into this snippet of code written by Cody Konior [Check if Database Mail is running](https://www.codykonior.com/2015/06/02/check-if-database-mail-is-running/) that totally saved the day, it did exactly what I wanted so I tweaked it a little, wrapped it in a stored procedure, popped it in my DBA_Tasks database and set an Agent Job to run it once a day.

Of course full credit to Cody for this one, I just amended his suggestion to fit my needs but wanted to share my experience with running into this problem with DBMail and how I went about fixing it.
