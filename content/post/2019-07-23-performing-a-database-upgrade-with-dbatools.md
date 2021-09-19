---
title: Performing a database upgrade with dbatools
date: 2019-07-23T18:00:43+01:00
author: Rich
layout: post
permalink: /performing-a-database-upgrade-with-dbatools
categories:
  - "powershell"
tags:
  - AvailailibtyGroup
  - dba
  - dbatools
  - "powershell"
  - SQL
  - sqlserver
---
I was recently asked to move one of our production databases to a staging server, this was to allow one of our suppliers to update the schema, once the update had been complete I needed to move the database back, our production server was running SQL Server 2012 in a two node availability group, the staging server was running SQL Server 2012 Express. Here is how I made all that happen with just a few lines of PowerShell using the fantastic dbatools.

### Credentials Please

Set your SQL Credentials using Get-Credential, this will live for the life of the session, so if we run it first we can use $cred in the proceeding commands.

```
    $cred = Get-Credential -UserName 'BonzaOwl' -Message 'Password Please'
```

### Who&#8217;s There?

Check to make sure that nobody is connected to the production database

```
    Get-DbaProcess -SqlInstance servera -Database VegetableGarden | Select-Object Host,Login,Program
```

### Time to leave!

If there are still active connections to the production database we can remove them using Stop-DbaProcess

```
    Get-DbaProcess -SqlInstance servera -Database VegetableGarden | Stop-DbaProcess
```

### Remove Availability Group

Now it was time to remove the database from the availability group, this was done before the move.

```
    Remove-DbaAgDatabase -SqlInstance servera -AvailabilityGroup VegetablePatch -Database VegetableGarden -SqlCredential $cred -Confirm
```

### Beam Me Up Scotty

Now, this is where the magic happens, using Copy-DbaDatabase I was able to move the database from the production environment into the staging environment using one line of code, it is however important to note that both instances need to have access to whatever location is specified in -SharedPath if they don&#8217;t the copy will of course, fail.

```
    Copy-DbaDatabase -Source servera -Destination serverb -SourceSqlCredential $cred -DestinationSqlCredential $cred -SharedPath \\Some\\Shared\Path -Database VegetableGarden -BackupRestore -NewName VegetableGarde_Staging
```

### Change The Name

To make sure that the old database couldn&#8217;t be confused when the staging database is moved back, I renamed it but left the database in the instance set to offline, my theory here was that I could use it as part of the rollback plan.

```
    Rename-DbaDatabase -SqlInstance servera -SqlCredential $cred -Database VegetableGarden -DatabaseName "dbatools_&lt;DBN&gt;_&lt;DATE&gt;" -FileGroupName "dbatools_&lt;FGN&gt;" | Set-DbaDbState -Offline
```

### Take Me Home

Once the updates on the staging server were complete, I simply used Copy-DbaDatabase to move the database back to the production server using the same command as before.

```
    Copy-DbaDatabase -Source serverb -Destination servera -SourceSqlCredential $cred -DestinationSqlCredential $cred -SharedPath \\Some\\Shared\Path -Database VegetableGarde_Staging -BackupRestore -NewName VegetableGarden
```

### Add Availability Group

Finally, the database was added back to the Availability Group some touch testing on the database itself was performed before it was handed back to the project team for user acceptance testing.

```
    Add-DbaAgDatabase -SqlInstance servera -AvailabilityGroup VegetablePatch -Database VegetablePatch -SqlCredential $cred
```