---
title: SYSTEM_USER and it's limitations
date: 2019-03-06T18:50:03+01:00
author: Rich
layout: post
permalink: /system_user-and-its-limitations
categories:
  - sqlserver
tags:
  - dba
  - security
  - SQL
  - sqlserver
---

Recently I came across a stored procedure I had written a few years ago, when I first started as a developer I was given a task where I needed to write a stored procedure that allowed users to update an underlying table but also track the changes, what data had been amended and who amended it, to make it work I made use of an audit table to capture them details around which users were doing what. To achieve this I had selected to use [SYSTEM_USER](https://docs.microsoft.com/en-us/sql/t-sql/functions/system-user-transact-sql?view=sql-server-2017) not knowing at the time it's limitations.

I ran into a scenario a little while ago where the underlying data had been updated but the audit table suggested that a user had updated the data who said they didn't, SYSTEM_USER was raised as a potential problem and not knowing it could record another user's context I set about investigating how this works.

So like all things, I built out a test to see how this could have potentially happened. First thing is first, I need a stored procedure that I could use, my instance of SQL has a database called **DBA_Tasks** in here I put a stored procedure called LoginTest

```
    CREATE PROCEDURE LoginTest
    
    AS
    
    SELECT SYSTEM_USER AS [Executing User];
    
    SELECT ORIGINAL_LOGIN() AS [User Logged In]
    
    SELECT RIGHT(ORIGINAL_LOGIN(),LEN(ORIGINAL_LOGIN()) -16) AS [Username Without Domain]

    GO
```

What this will do is return me the following;

1. The executing user
2. The user who is logged into the computer
3. The username without the computer name

Okay so let's find out what we get back from that stored procedure if I just execute it as me

```
    EXEC LoginTest
```

I get exactly what I am expecting, my username is in both the SYSTEM_USER and ORIGINAL_LOGIN() function returns.

![](/img/SystemUser_Result1.png)

Now I need another user in that database to test against, someone that isn't me. I am going to map this user to the DBA_Tasks database and give them db_owner permissions

```
  USE [master]
  GO
  CREATE LOGIN [DbaTasksTest] WITH PASSWORD=N'Abracadabra', DEFAULT_DATABASE=[DBA_Tasks], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
  GO
  USE [DBA_Tasks]
  GO
  CREATE USER [DbaTasksTest] FOR LOGIN [DbaTasksTest]
  GO
  USE [DBA_Tasks]
  GO
  ALTER ROLE [db_owner] ADD MEMBER [DbaTasksTest]
  GO
```

Now that we have another user, let's try executing that stored procedure as them;

```
  EXECUTE AS USER = 'DbaTasksTest'
  
  EXECUTE LoginTest
  
  REVERT
```

I am still logged into the computer as me but what results am I going to get back from my test stored procedure?

![](/img/SystemUser_Result2.png)

As you can see, SYSTEM_USER has returned the principal that we ran the stored procedure as, if we are trying to capture WHO ran that stored procedure this obviously wouldn't be sufficient, ORIGINAL_LOGIN() however has given us the username of the principal logged into the machine and where the query was being executed which in this case would have been correct.

Reading through the [documentation](https://docs.microsoft.com/en-us/sql/t-sql/functions/system-user-transact-sql?view=sql-server-2017) for SYSTEM_USER we see the following;

```
    SYSTEM_USER
```

returns the name of the currently executing context. If the EXECUTE AS statement has been used to switch context, SYSTEM_USER returns the name of the impersonated context.

Of course we had given the login db_owner to the DBATasks database in the first test, so they could essentially do anything that they pleased within that database, but what if the user has just select permissions against the database?

```
    USE [DBA_Tasks]
    GO
    ALTER ROLE [db_datareader] ADD MEMBER [DbaTasksTest]
    GO
    USE [DBA_Tasks]
    GO
    ALTER ROLE [db_owner] DROP MEMBER [DbaTasksTest]
    GO
```

Let's try executing the stored procedure now

```
    EXECUTE AS USER = 'DbaTasksTest'

    EXECUTE LoginTest

    REVERT
```

As you can see from the result returned, we are not allowed to do that

```
    Msg 15517, Level 16, State 1, Line 17
    Cannot execute as the database principal because the principal "DbaTasksTest" does not exist, this type of principal cannot be impersonated, or you do not have permission.
```

Even if we give the user db_datawriter permissions we still can't impersonate another user, the principal needs explicit impersonate permissions, db_owner or be a sysadmin for this to work.

Wanna see? Ok let's demonstrate that;

```
    USE [Master]

    GRANT IMPERSONATE ON LOGIN::[DbaTasksTest] to [DbaTasksTest]
```

```
    USE [DBA_Tasks]
    GO
    GRANT EXECUTE TO [DbaTasksTest]
    GO
```

Our user still only has datawriter, datareader and execute permissions to the DBATasks database but let's try executing our stored procedure as DBATasksTest now

```
    EXECUTE AS USER = 'DbaTasksTest'
  
    EXECUTE LoginTest
  
    REVERT
```

There we have it, we are now executing that stored procedure as someone else.

![](/img/SystemUser_Result2.png)

We can check if a login has IMPERSONATE permissions by running the following TSQL query against the instance, 1 or 0 will be returned depending on the permission set.

```
    SELECT HAS_PERMS_BY_NAME('Ps', 'LOGIN', 'IMPERSONATE');
```

It is clear that SYSTEM_USER wouldn't be suitable for the use case we are proposing, current user will return the windows principal even when EXECUTE AS has been executed before the stored procedure was called, of course as shown in our tests if the individual that is running the stored procedure has full access to the underlying table none of this really matters as they could simply run an update to rid their name from existence from that table anyway.
