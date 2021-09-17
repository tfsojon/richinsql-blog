---
title: SQL Server 2014 Service Pack Error
date: 2020-02-29T08:00:00+00:00
author: Rich
layout: post
permalink: /SQL-Server-2014-Service-Pack-Error
categories:
  - sqlserver
tags:
  - Powershell
  - SQL
---

I had an issue on one node in the new SQL 2014 cluster that had just been built S-SQL06 specifically where [KB4470220](https://support.microsoft.com/en-us/help/4470220/cumulative-update-1-for-sql-server-2014-sp3) & [KB4532095](https://support.microsoft.com/en-us/help/4532095/description-of-the-security-update-for-sql-server-2014-sp3-gdr-feb) wouldn’t install, they kept throwing an error with code 0x80070643 which when googled didn’t relate to the actual problem, I did some digging and found out what was causing the problem and thought I would share my findings in case anyone runs into this same problem in the future.

![Database Corruption 1](/img/SQL-Service-Pack-Install-Error-1.jpg)

As I couldn’t get the updates to install from within windows I manually downloaded both of these KB items however for some reason the Windows Update Catalog wouldn’t work so I broke out powershell and pulled them using the following cmdlet, you will need [kbupdate](https://github.com/potatoqualitee/kbupdate) installed for this to work

```
Save-KbUpdate -Architecture x64 -Path C:\temp -Name KB4470220, 4532095
```

Once I had them downloaded I tried installing each update individually on the server and almost immediately was presented with the following error

*“The user data directory in the registry is not valid”*

![Database Corruption 1](/img/SQL-Service-Pack-Install-Error-2.png)

A quick google search pointed me to Dave Pinal’s [article](https://blog.sqlauthority.com/2015/05/16/sql-server-service-pack-error-the-user-data-directory-in-the-registry-is-not-valid-verify-defaultdata-key-under-the-instance-hive-points-to-a-valid-directory/) on the same issue.  

In the article Dave points out that the default Data or Log directory in the SQL Server Engine configuration are potentially set to a directory that doesn't exist on the disk, I thought that was a bit weird as I was almost sure I had configured the directories correctly when running the post installation tasks so as Dave points out I fired up the registry and navigated to the following HIVE 

**Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL12.MSSQLSERVER\MSSQLServer**

Now that I had the registry open, I wanted to check against each node in the cluster to see what the difference was, SQL05 was good, the installation had already successfully installed there so that is where I started.

## SQL05

![Database Corruption 1](/img/SQL-Service-Pack-Install-Error-3.jpg)

I checked the data and log defaults in SQL and they were correct

![Database Corruption 1](/img/SQL-Service-Pack-Install-Error-4.jpg)

This path is VALID and physically exists so the update installed on this server as expected

## SQL06

![Database Corruption 1](/img/SQL-Service-Pack-Install-Error-5.jpg)

I checked the data and log defaults in SQL and they were correct

![Database Corruption 1](/img/SQL-Service-Pack-Install-Error-6.png)

However, as you can see there is a mismatch, this node has been configured to have the data and log files on teh root of **S:\\** and **L:\\** however **S:\Data** and **L:\Data** don’t actually exist at all so I created the Data directory on both S:\ & L:\ drives on SQL06 which made the physical path S:\Data and L:\Data available and tried the updates again

Lone behold – they installed first time

![Database Corruption 1](/img/SQL-Service-Pack-Install-Error-7.jpg)