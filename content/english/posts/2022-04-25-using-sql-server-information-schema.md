---
title: Using SQL Server's Information Schema
date: 2022-04-25T09:00:00.000+01:00
author: Rich
layout: post
categories:
- sqlserver
draft: false
tags:
- SQLServer
- T-SQL
- Intermediate
---

Information schema's are one of the methods that SQL Server provides to obtain meta data. 

These views can be used to obtain information about a variety of different objects within a SQL Server Database, of which are detailed below. 

CHECK_CONSTRAINTS
COLUMN_DOMAIN_USAGE
COLUMN_PRIVILEGES
COLUMNS
CONSTRAINT_COLUMN_USAGE
CONSTRAINT_TABLE_USAGE
DOMAIN_CONSTRAINTS
DOMAINS
KEY_COLUMN_USAGE
PARAMETERS
REFERENTIAL_CONSTRAINTS
ROUTINES
ROUTINE_COLUMNS
SCHEMATA
TABLE_CONSTRAINTS
TABLE_PRIVILEGES
TABLES
VIEW_COLUMN_USAGE
VIEW_TABLE_USAGE
VIEWS

How can we use these views to our advantage? In this example we are going to use the StackOverflow database. 

In this database we have a table called Posts, within Posts is a primary key called ID, what if we wanted to find out where that column was used elsewhere in the database? 

This is where the INFORMATION_SCHEMA views come in, specifically the COLUMNS view. Here we can see all of the columns and where they are used elsewhere in the database. 

```
SELECT 
*
FROM 
	StackOverflow2013.INFORMATION_SCHEMA.COLUMNS

WHERE 
    TABLE_NAME = 'Posts'
```

In the StackOverflow database the primary key **ID** from the posts table is referenced as PostID elsewhere, we know this much, but what we don't know is which tables it is referenced in specifically.

To locate the tables where the columns are used we can use this query. 

```
SELECT 
*
FROM 
	StackOverflow2013.INFORMATION_SCHEMA.COLUMNS

WHERE 
    COLUMN_NAME = 'Postid'

ORDER BY 
	TABLE_NAME,
	ORDINAL_POSITION
```

Here is a more detailed query from the COLUMNS view that you can adapt to your own requirements. 

```
SELECT 

TABLE_CATALOG as [Database],
TABLE_SCHEMA as [Schema],
TABLE_NAME as [Table_Name],
COLUMN_NAME as [Column_Name],
ORDINAL_POSITION as [Col_Position_In_Table],
CONCAT(QUOTENAME(TABLE_CATALOG),'.',QUOTENAME(TABLE_SCHEMA),'.',QUOTENAME(TABLE_NAME)) as [Full_Schema_Path],
CONCAT('SELECT * FROM ',QUOTENAME(TABLE_CATALOG),'.',QUOTENAME(TABLE_SCHEMA),'.',QUOTENAME(TABLE_NAME)) as [Select_All_Query],
CONCAT('SELECT ', COLUMN_NAME, ' FROM ',QUOTENAME(TABLE_CATALOG),'.',QUOTENAME(TABLE_SCHEMA),'.',QUOTENAME(TABLE_NAME)) as [Select_Column_Query],
LOWER(IS_NULLABLE) AS [Is_Nullable],
DATA_TYPE as Data_Type,
CASE 
	WHEN CHARACTER_MAXIMUM_LENGTH = '-1' THEN 'MAX'
	WHEN CHARACTER_MAXIMUM_LENGTH IS NULL THEN 'N/A'
	ELSE CAST(CHARACTER_MAXIMUM_LENGTH as varchar)
END AS Character_Maximum_Length
FROM 
	StackOverflow2013.INFORMATION_SCHEMA.COLUMNS
ORDER BY 
	TABLE_NAME,
	ORDINAL_POSITION
```