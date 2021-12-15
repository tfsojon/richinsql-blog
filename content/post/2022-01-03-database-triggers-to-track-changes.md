---
title: SQL Server Database Trigger To Track Changes
date: 2022-01-03T09:00:00.000+01:00
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

First, we are going to create a database to store an audit of all of the changes.

```
CREATE DATABASE ChangeTracking
```
Next we will create another database as a demo to show how changes to objects can be tracked. 
If you already have a database you would like to track changes for, you can skip this step. 

```
CREATE DATABASE TestDatabase
```
Switch context into the TestDatabase

```
USE TestDatabase
```

Create a test table, as above if you already have some objects that you would like to track changes for, you can skip this step.

```
CREATE DATABASE dbo.TestTable 
(
	ID INT IDENTITY(1,1)
)
```

Switch database context to the ChangeTracking database, this is where all of the changes are going to be recorded. 

```
USE ChangeTracking
```

A table to hold the changes is needed so create that using the below, it is important to note that if you add any new columns they match what you expect to be inserted, if they don't it could cause the trigger to fail and in turn your changes will also fail to commit. 

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

Switch database context into the database that the changes need to be tracked on, in our example, this is TestDatabase. 

```
USE TestDatabase
```

Once the database context has been changed, creating a database trigger is the next task. 
This database trigger is going to track the following changes; 

* New Stored Procedures
* Changes to stored procedures 
* Dropped Stored Procedures
* Schema Changes
* Table Changes

```
ALTER TRIGGER AuditTrigger
    ON DATABASE
    FOR CREATE_PROCEDURE, ALTER_PROCEDURE, DROP_PROCEDURE, ALTER_SCHEMA, ALTER_TABLE
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE
        @EventData XML = EVENTDATA();

    INSERT ChangeTracking.dbo.AuditEvents
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

Now that the trigger is created, we can now make a change to our test table to see if it works.

```
ALTER TABLE dbo.TestTable ADD DateAdded DATETIME DEFAULT DATETIME
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
	[ChangeTracking].[dbo].[AuditEvents]
WHERE
	EventType = 'ALTER_TABLE'
```

If everything worked as expected, you should have a row for the change that you just performed. 

This demo can be expanded to suit your own needs, but as mentioned at the beginning, make sure that database triggers are tested in depth before adding into production as they can cause things to break where you wouldn't want them to break if testing is skipped. 