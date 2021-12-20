---
title: SQL Server Database Trigger To Track Changes
date: 2021-12-20T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/database-triggers-to-track-changes"
categories:
- sqlserver
tags:
- SQL
- T-SQL
- Triggers
---

Have you ever had a situation where a change was made to a stored procedure, table or schema in a database you manage, you asked around the team and everyone said it wasn't them? In this article we are going to take a look at Database Triggers and how we can use them to track changes.

### What is a DDL Trigger?

A DDL trigger is much like a table trigger, these triggers fire in response to a variety of Data Definition Language (DDL) events. 
Primarially these corrospond with T-SQL statements that start with the keywords CREATE,ALTER,DROP,GRANT,DENY,REVOKE or UPDATE STATISTICS. 

DDL Triggers can be used when you want to do the following 

* Record changes to a database
* Prevent changes to a database 
* Record changes to a stored procedure

### Creating the trigger

First, we are going to create a database to store an audit of all of the changes.

```
CREATE DATABASE ComeFindMe
```
Next we will create another database as a demo to show how changes to objects can be tracked. 
If you already have a database you would like to track changes for, you can skip this step, but make a note of that database name, as you will need it later.  

```
CREATE DATABASE CatchMeIfYouCan
```
Switch context into the CatchMeIfYouCan database

```
USE CatchMeIfYouCan
```

Create a test table, as above if you already have some objects that you would like to track changes for, you can skip this step, just make a note of the object you are going to use for testing. 

```
CREATE DATABASE dbo.TimesAreChanging 
(
	ID INT IDENTITY(1,1)
)
```

Switch database context again back to the ComeFindMe database, this is where all of the changes are going to be recorded. 

```
USE ComeFindMe
```

A table to hold the changes is needed so create that using the below, it is important to note that if you add any new columns they must match what you expect to be inserted, if they don't it could cause the trigger to fail and in turn your changes will also fail to commit. 

```
CREATE TABLE [dbo].[AuditEvents]
(
	[EventDate] [datetime] NOT NULL,
	[EventType] [nvarchar](64) NULL,
	[EventDDL] [nvarchar](max) NULL,
	[EventXML] [xml] NULL,
	[DatabaseName] [nvarchar](255) NULL,
	[SchemaName] [nvarchar](255) NULL,
	[ObjectName] [nvarchar](255) NULL,
	[LoginName] [nvarchar](255) NULL
)
```

Switch database context once more, this time into the database that the changes need to be tracked on, in our example, this is the CatchMeIfYouCan database. 

```
USE CatchMeIfYouCan
```

Once the database context has been changed, creating a database trigger is the next task. 
This database trigger will fire when the following changes are made; 

* New Stored Procedures are created
* Changes are made to existing stored procedures 
* Stored Procedures are dropped
* A Schema is changed
* A Table is changed

```
ALTER TRIGGER AuditTrigger
    ON DATABASE
    FOR CREATE_PROCEDURE, ALTER_PROCEDURE, DROP_PROCEDURE, ALTER_SCHEMA, ALTER_TABLE
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE
        @EventData XML = EVENTDATA();

    INSERT ComeFindMe.dbo.AuditEvents
    (
        EventType,
        EventDDL,
        EventXML,
        DatabaseName,
        SchemaName,
        ObjectName,
        LoginName
    )
    SELECT
        @EventData.value('(/EVENT_INSTANCE/EventType)[1]',   'NVARCHAR(100)'), 
        @EventData.value('(/EVENT_INSTANCE/TSQLCommand)[1]', 'NVARCHAR(MAX)'),
        @EventData,
        DB_NAME(),
        @EventData.value('(/EVENT_INSTANCE/SchemaName)[1]',  'NVARCHAR(255)'), 
        @EventData.value('(/EVENT_INSTANCE/ObjectName)[1]',  'NVARCHAR(255)'),
        SUSER_SNAME();
END
```

Now that the trigger is created, we can make a change to our test table. Remember, this is the one you made a note of before? You are doing this to test that the trigger is actually working.

```
ALTER TABLE dbo.TimesAreChanging ADD DateAdded DATETIME DEFAULT DATETIME
```

Once the change have been to the table we can query the ChangeTracking database to see if the changes were successfully recorded.

```
SELECT 
	[EventDate]
    ,[EventType]
    ,[EventDDL]
    ,[EventXML]
    ,[DatabaseName]
    ,[SchemaName]
    ,[ObjectName]
    ,[LoginName]
FROM 
	[ComeFindMe].[dbo].[AuditEvents]
WHERE
	EventType = 'ALTER_TABLE'
```

If everything worked as expected, you should have a row for the change that you just performed. 

This demo can be expanded to suit your own needs, but as mentioned at the beginning, make sure that database triggers are tested in depth before adding into production as they can cause things to break where you wouldn't want them to break if testing is skipped. 