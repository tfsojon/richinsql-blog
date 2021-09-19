---
title: Getting started with DBATools
date: 2019-03-21T16:00:06+01:00
author: Rich
layout: post
permalink: /getting-started-with-dba-tools
categories:
  - powershell
tags:
  - dba
  - powershell
---
I have never really got into PowerShell it is just something that I have never needed to use, however recently I have been reading more and more about the great things that can be done with DBATools so thought I would give the module a go and see what it could do.

### First thing is first

I needed to find out what version of PowerShell I am using so the correct method of installation for the module could be used. To do that I need to execute the following function;

```
    $PSVersionTable.PSVersion
```

This returns the currently installed version of PowerShell

![](/img/PSVersion.png)

As you can see my system has version 5 installed, so I can use one of two installation methods.

  * I can use PowerShellGet to download the module

```
    Save-Module -Name dbatools -Path C:\Temp
```

  * I can invoke a web request to download the module

```
    Invoke-WebRequest -Uri powershellgallery.com/api/v2/package/dbatools -OutFile C:\Temp\dbatools.zip
```

The web request method is a little bit slower but if PowerShell 5 isn&#8217;t available you will have to use it because PowerShellGet wasn&#8217;t introduced until version 5. That is it, there is nothing else to it the module is now available to use.

When installing the module, I was presented with the following, as the NuGet Provider is required, selecting yes is the only option to allow the installation to proceed

![](/img/PSNuGet.png)

&nbsp;

### Now what?

Now that I have the module installed there are so many functions that can be used, it is worth taking a look at the [Command Index](https://dbatools.io/commands/) or run this command to get a list of available commands right inside PowerShell

```
    Find-dbacommand
```

![](/img/DBACommand.png)

As you can see, that returns a list of all the available commands that are usable within DBATools. But what if I want to find something more specific, let&#8217;s say I need to perform a backup but don&#8217;t know the command? Well, that is built into DBATools too, simply run this command.

```
    Find-DbaCommand -tag backup
```

![](/img/DBACommand-BackupTag.png)

As you can see that will return all the available backup related commands that can be performed in DBATools.

### Let&#8217;s Backup

Now that I know what commands I have available, let&#8217;s have a look at how to use one of them, I want to back up a database on my SQL Server but don&#8217;t want to have to load up SSMS, we can do that in DBATools.

```
    Backup-dbadatabase -Sqlinstance localhost -Database StackOverflow2010 -BackupDirectory C:\Backups -BackupFileName Stackoverflow.bak
```

When we run this command, DBATools will back up our StackOverflow database to the directory we have specified with the filename we have specified.

![](/img/BackupDatabase.png)

&nbsp;

![](/img/BackupDatabase-Done.png)

So that is cool, but that is going to break my log chain! DBATools can handle that too just specify the CopyOnly parameter.

```
    Backup-dbadatabase -Sqlinstance localhost -Database StackOverflow2010 -BackupDirectory C:\Backups -BackupFileName Stackoverflow.bak -CopyOnly
```

### Last Backup

So now we know how to backup databases from DBATools how can we find out when the database was last backed up? Sure we can.

```
    Get-DbaLastBackup -SqlInstance localhost | out-gridview
```

![](/img/LastBackupAll.png)

This is going to show us the last backup information for ALL databases on the instance, but what we want to just find out about one database, well just like the backup command, specify the database.

```
    Get-DbaLastBackup -SqlInstance localhost -Database StackOverflow2010 | out-gridview
```

![](/img/LastBackupSO.png)

### Check it once, check it twice

So now I know how to backup my databases, I can find out when the last backup was taken what about checking the backup, well you guessed it, DBATools can help with that too.

```
    Test-DBALastBackup -sqlinstance localhost | out-gridview
```

![](/img/LastBackupCheck.png)

If I have a look in SSMS during the restore I will be able to see that the restore is actually taking place

![](/img/LastBackupCheck-SSMS.png)

&nbsp;

Just like the backup command, if I want to test a specific database I can specify that using the **-database** flag

```
    Test-DBALastBackup -sqlinstance localhost -Database StackOverflow2010 | out-gridview
```

### Tip Of The Iceberg

This is just the tip of the iceberg, there is so much more that can be done with DBATools, not just Backups, it is worth checking out the [DBATools](https://www.dbatools.io) website which has a wealth of information on what the module can do and how you can use it.

<https://docs.dbatools.io/>

This is a great repository of commands with a detailed explanation of each and how they work, so if you wanted to find out how Test-DBALastBackup actually works you could look it up [here](https://docs.dbatools.io/#Test-DbaLastBackup).

### Let&#8217;s Upgrade

DBATools is constantly being developed so it is worth knowing how to go about keeping it updated. As DBATools was downloaded from the PowerShell Gallery the following command can be used to perform an update of the module, when running this command it is worth noting that the PowerShell command line has to be run as administrator.

```
    Update-Module dbatools
```

### More Help

DBATools has a great community over on the [SQL Community slack](https://dbatools.io/slack/) it is a great place to get more help and advice when using the module, the developers are active there and are always on hand to help.