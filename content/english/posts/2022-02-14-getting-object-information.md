---
title: Getting Object Information
date: 2022-02-14T09:00:00.000+01:00
image: "/images/post/apple-health-powerBi-dashboard.jpg"
author: Rich
layout: post
draft: true
categories:
- sqlserver
tags:
- SQLServer
- T-SQL
- Intermediate
---

In this post I am going to look at how to find information about objects in all of the databases on an database instance.

### The Problem

I want to find all objects in every database sorted by the database name, object type and date which they were created. I could loop through each system view in each database and get the objects one by one, but is there an easier way?

### sys.objects vs sys.all_objects

In each database there are two objects we could use, these reside on the sys schema, sys.objects and sys.all_objects. 
The former contains all user created objects within the scope of the database context while sys.all_objects includes everything from sys.objects plus any system objects that are within the database scope.

```SELECT * FROM sys.objects```

```SELECT * FROM sys.all_objects```

### sys.schema 

sys.schema holds information about all of the schemas registered within the database, we are joining to this DMV to get the schema name for each of the objects. 

### sp_MSforeachdb

In the provided solution you will see reference to sp_MSforeachdb, but what exactly is sp_MSforeachdb and what does it do? 

sp_MSforeachdb is an undocumented Microsoft shipped stored procedure that accepts an nvarchar(2000) command, sp_MSforeachdb will loop every database on the instance which it is run and execute the command provided. 

This is useful for us as we want to get all objects from all databases for which are on the instance.

We could of course write a while loop or a cursor but that would be more lines of code, why do that when there is a solution already available for us to use.

### The Solution

This script will loop all of the databases on the instance in scope. We make use of a temp table to store the results of each database and then select them back out at the end. 

```IF OBJECT_ID('tempdb..#objectsAllDatabases') IS NOT NULL DROP TABLE tempdb..#objectsAllDatabases

DECLARE @SqlQuery varchar(4000)

CREATE TABLE #objectsAllDatabases
(
ID INT IDENTITY(1,1),
DatabaseName varchar(255),
ParentObjectName varchar(255),
ObjectName varchar(255),
ObjectDefinition varchar(MAX),
SchemaName varchar(255),
Type varchar(10),
TypeDescription varchar(255),
CreatedDate DATETIME,
ModifiedDate DATETIME
)

SET @SqlQuery =
'USE ?

INSERT INTO #objectsAllDatabases (DatabaseName,ParentObjectName,ObjectName,ObjectDefinition,SchemaName,Type,TypeDescription,CreatedDate,ModifiedDate)
SELECT
DB_NAME() as DatabaseName,
OBJECT_NAME(p.parent_object_id) ParentObjectName,
p.name as ObjectName,
OBJECT_DEFINITION(object_id),
s.name as SchemaName,
type,
CASE
	WHEN [Type] = ''AF'' THEN ''Aggregate function (CLR)''
	WHEN [Type] = ''C'' THEN ''CHECK constraint''
	WHEN [Type] = ''D'' THEN ''DEFAULT (constraint or stand-alone)''
	WHEN [Type] = ''F'' THEN ''FOREIGN KEY constraint''
	WHEN [Type] = ''FN'' THEN ''SQL scalar function''
	WHEN [Type] = ''FS'' THEN ''Assembly (CLR) scalar-function''
	WHEN [Type] = ''FT'' THEN ''Assembly (CLR) table-valued function''
	WHEN [Type] = ''IF'' THEN ''SQL inline table-valued function''
	WHEN [Type] = ''IT'' THEN ''Internal table''
	WHEN [Type] = ''P'' THEN ''SQL Stored Procedure''
	WHEN [Type] = ''PC'' THEN ''Assembly (CLR) stored-procedure''
	WHEN [Type] = ''PG'' THEN ''Plan guide''
	WHEN [Type] = ''PK'' THEN ''PRIMARY KEY constraint''
	WHEN [Type] = ''R'' THEN ''Rule (old-style, stand-alone)''
	WHEN [Type] = ''RF'' THEN ''Replication-filter-procedure''
	WHEN [Type] = ''S'' THEN ''System base table''
	WHEN [Type] = ''SN'' THEN ''Synonym''
	WHEN [Type] = ''SO'' THEN ''Sequence object''
	WHEN [Type] = ''U'' THEN ''Table (user-defined)''
	WHEN [Type] = ''V'' THEN ''View''
	WHEN [Type] = ''EC'' THEN ''Edge constraint''
	WHEN [Type] = ''SQ'' THEN ''Service queue''
	WHEN [Type] = ''TA'' THEN ''Assembly (CLR) DML trigger''
	WHEN [Type] = ''TF'' THEN ''SQL table-valued-function''
	WHEN [Type] = ''TR'' THEN ''SQL DML trigger''
	WHEN [Type] = ''TT'' THEN ''Table type''
	WHEN [Type] = ''UQ'' THEN ''UNIQUE constraint''
	WHEN [Type] = ''X'' THEN ''Extended stored procedure''
	WHEN [Type] = ''ST'' THEN ''STATS_TREE''
END AS TypeDescription,
create_date,
modify_date

FROM sys.objects p

INNER JOIN sys.schemas s ON p.schema_id = s.schema_id

WHERE
	is_ms_shipped = 0'

EXEC sp_MSforeachdb @SqlQuery

SELECT
ID,
DatabaseName,
SchemaName,
ObjectName,
TypeDescription as ObjectType,
CASE 
	WHEN Type = 'P' THEN CONCAT('DROP PROCEDURE ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ObjectName))
	WHEN Type = 'U' THEN CONCAT('DROP TABLE ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ObjectName))
	WHEN Type = 'V' THEN CONCAT('DROP VIEW ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ObjectName))
	WHEN TYpe = 'PK' THEN CONCAT('ALTER TABLE ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ParentObjectName),' DROP CONSTRAINT ',QUOTENAME(ObjectName))
	WHEN TYpe = 'F' THEN CONCAT('ALTER TABLE ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ParentObjectName),' DROP CONSTRAINT ',QUOTENAME(ObjectName))
	WHEN TYpe = 'D' THEN CONCAT('ALTER TABLE ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ParentObjectName),' DROP CONSTRAINT ',QUOTENAME(ObjectName))
	WHEN TYpe = 'UQ' THEN CONCAT('ALTER TABLE ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ParentObjectName),' DROP CONSTRAINT ',QUOTENAME(ObjectName))
	WHEN TYpe = 'FN' THEN CONCAT('DROP FUNCTION ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ObjectName))
	WHEN TYpe = 'IF' THEN CONCAT('DROP FUNCTION ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ObjectName))
	WHEN TYpe = 'TF' THEN CONCAT('DROP FUNCTION ',QUOTENAME(Databasename),'.',QUOTENAME(SchemaName),'.',QUOTENAME(ObjectName))
END as DropCommand,
ObjectDefinition,
CreatedDate,
ModifiedDate,
CASE 
	WHEN Type = 'P' THEN 'https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-procedures-transact-sql?view=sql-server-ver15'
	WHEN Type = 'U' THEN 'https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?view=sql-server-ver15'
	WHEN Type = 'V' THEN 'https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-views-transact-sql?view=sql-server-ver15'
	WHEN TYpe = 'PK' THEN NULL
	WHEN TYpe = 'F' THEN NULL
	WHEN TYpe = 'D' THEN 'https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-default-constraints-transact-sql?view=sql-server-ver15'
	WHEN TYpe = 'UQ' THEN NULL
	WHEN TYpe = 'FN' THEN NULL
	WHEN TYpe = 'IF' THEN NULL
	WHEN TYpe = 'TF' THEN 'No'
END AS MoreInfo

FROM #objectsAllDatabases ORDER BY DatabaseName,type,CreatedDate
```