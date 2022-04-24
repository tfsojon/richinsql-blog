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

Information schema's are one of the methods that SQL Server provides to obtain meta data about various different objects within a database of which are detailed below. 

- [CHECK_CONSTRAINTS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/check-constraints-transact-sql?view=sql-server-ver15)
- [COLUMN_DOMAIN_USAGE](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/column-domain-usage-transact-sql?view=sql-server-ver15)
- [COLUMN_PRIVILEGES](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/column-privileges-transact-sql?view=sql-server-ver15)
- [COLUMNS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/columns-transact-sql?view=sql-server-ver15)
- [CONSTRAINT_COLUMN_USAGE](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/constraint-column-usage-transact-sql?view=sql-server-ver15)
- [CONSTRAINT_TABLE_USAGE](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/constraint-table-usage-transact-sql?view=sql-server-ver15)
- [DOMAIN_CONSTRAINTS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/domain-constraints-transact-sql?view=sql-server-ver15)
- [DOMAINS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/domains-transact-sql?view=sql-server-ver15)
- [KEY_COLUMN_USAGE](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/key-column-usage-transact-sql?view=sql-server-ver15)
- [PARAMETERS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/parameters-transact-sql?view=sql-server-ver15)
- [REFERENTIAL_CONSTRAINTS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/referential-constraints-transact-sql?view=sql-server-ver15)
- [ROUTINES](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/routines-transact-sql?view=sql-server-ver15)
- [ROUTINE_COLUMNS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/routine-columns-transact-sql?view=sql-server-ver15)
- [SCHEMATA](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/schemata-transact-sql?view=sql-server-ver15)
- [TABLE_CONSTRAINTS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/table-constraints-transact-sql?view=sql-server-ver15)
- [TABLE_PRIVILEGES](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/table-privileges-transact-sql?view=sql-server-ver15)
- [TABLES](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/tables-transact-sql?view=sql-server-ver15)
- [VIEW_COLUMN_USAGE](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/view-column-usage-transact-sql?view=sql-server-ver15)
- [VIEW_TABLE_USAGE](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/view-table-usage-transact-sql?view=sql-server-ver15)
- [VIEWS](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/views-transact-sql?view=sql-server-ver15)

But how can we use these views to our advantage? In this example we are going to use the StackOverflow database. 

In the StackOverflow database we have a table called Posts, within Posts is a primary key called ID, what if we wanted to find out where that column was used elsewhere in the database? 

This is where the [INFORMATION_SCHEMA](https://docs.microsoft.com/en-us/sql/relational-databases/system-information-schema-views/system-information-schema-views-transact-sql?view=sql-server-ver15) views come in, specifically the **COLUMNS** view. Here we can see all of the columns in each table within the database this is useful for finding out where these columns are used elsewhere.

Lets have a look at the Posts Table and and what columns are in there.

```
SELECT 
*
FROM 
	StackOverflow.INFORMATION_SCHEMA.COLUMNS

WHERE 
    TABLE_NAME = 'Posts'
```

As you can see there is an ID column, this is what we want to find in other tables, within the StackOverflow database the primary key **ID** from the posts table is referenced in other tables as PostID, this we already knew, but what we don't know is which tables it is referenced in specifically.

To locate the tables where the column is used we can use this query. 

```
SELECT 
*
FROM 
	StackOverflow.INFORMATION_SCHEMA.COLUMNS

WHERE 
    COLUMN_NAME = 'Postid'
```

This shows that PostID is used in the following tables;

- Comments
- PostHistory
- PostLinks
- Votes

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
	StackOverflow.INFORMATION_SCHEMA.COLUMNS
ORDER BY 
	TABLE_NAME,
	ORDINAL_POSITION
```