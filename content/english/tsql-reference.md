---
title: "T-SQL Reference"
date: 2021-10-30T09:00:00.000+01:00
draft: false
---

This page serves as an ever growing resource of T-SQL code and examples which show how to use the statements provided. 

**Last Updated** 30/10/2021 11:56am

## Contents

* [Create Database](#Create-Database)
* [Drop Database](#drop-database)
* [Create Table](#create-table)
* [Datatypes](#datatypes)
* [Constraints](#constraints)
* [Alter Table](#alter-table)
* [Drop Table](#drop-table)
* [Select Rows](#select-rows)
* [Select Distinct Rows](#select-distinct-rows)
* [Joins](#joins)
* [Aggregate Functions](#aggregate-functions)
* [Conditions](#conditions)
* [Insert Rows](#insert-rows)
* [Update Rows](#update-rows)
* [Delete Rows](#delete-rows)
* [Truncate Table](#truncate-table)


## Create Database

Create database if a database doesn't already exist

```
IF NOT EXISTS(SELECT * FROM sys.databases WHERE name = 'database_name')
    BEGIN
        CREATE DATABASE [database_name]
    END
```
Create a database, if the database already exists an error will be thrown.

```
CREATE DATABASE [database_name]
```

## Drop Database

Drop Database if the Database exists

```
IF EXISTS(SELECT * FROM sys.databases WHERE name = 'database_name')
    BEGIN
        DROP DATABASE [database_name]
    END
```

Drop Database if the database doesn't exist an error will be thrown

```
DROP DATABASE [database_name]
```

## Create Table

Create a table if the table doesn't already exist.

```
IF NOT EXISTS(SELECT *
          FROM   [schema].[table_name])
  CREATE TABLE [database_name].[schema].[table_name]
(
    [column_a] [data_type] [constraints] DEFAULT [value],
    [column_b] [data_type] [constraints]

);
```

Create a table, if the table already exists an error will be thrown.

```
CREATE TABLE [database_name].[schema].[table_name]
(
    [column_a] [data_type] [constraints] DEFAULT [value],
    [column_b] [data_type] [constraints]

);
```

Create Table Example

```
CREATE TABLE [database_name].[schema].[table_name]
(
    id INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    customer_name varchar(10) NOT NULL,
    active BIT DEFAULT 1

);
```
## Datatypes

|Datatype|Description|
|---|---|
|bigint| |
|bit| |
|decimal| |
|int| |
|money| |
|numeric| |
|smallint| |
|smallmoney| |
|tinyint| |
|float| |
|real| |
|date| |
|datetime2| |
|datetime| |
|datetimeoffset| |
|smalldatetime| |
|time| |
|char| |
|varchar| |
|text| |
|nchar| |
|nvarchar| |
|ntext| |
|binary| |
|varbinary| |
|image| |
|cursor| |
|rowversion| |
|hierarchyid| |
|uniqueidentifier| |
|sql_variant| |
|xml| |


<!-- ## Constraints

|Constraint|Description|
|---|---|
|| | -->

## Alter Table

```
ALTER TABLE [schema].[table_name]
    ADD [column_name] [data_type] [NULL/NOT NULL]
    ALTER COLUMN [column_name] [data_type] [NULL/NOT NULL]
    DROP COLUMN [column_name]
```

Example Table Adjustment (addition)

```
ALTER TABLE [schema].[table_name] ADD [username] varchar(50) NOT NULL
```

Example Table Adjustment (removal)

```
ALTER TABLE [schema].[table_name] DROP [username]
```

Example Table Adjustment (change)

```
ALTER TABLE [schema].[table_name] ALTER COLUMN [username] varchar(60) NOT NULL
```

## Drop Table

Drop table if the table already exists in the database

```
IF EXISTS(SELECT *
          FROM  [schema].[table_name])
  DROP TABLE [database_name].[schema].[table_name]
```

Drop Table, this will throw an error if the table doesn't exist.

```
DROP TABLE [database_name].[schema].[table_name]
```

## Select Rows

```
SELECT 
    [column_a], 
    [column_b],
    AggregateFunction(column_c) AS [alias]
FROM
    [database_name].[schema].[table_name]
WHERE
    Condition
    AND Condition
    OR Condition
    NOT Condition
    IN Condition
GROUP BY 
    [column_a]
HAVING
    Condition
ORDER BY 
    [column_a] ASC/DESC
```

## Select Distinct Rows

```
SELECT DISTINCT 
    [column_a]
FROM
    [database_name].[schema].[table_name]
```

## Joins
|Join|Description|
|---|---|
|Inner Join| |
|Left {outer} Join| |
|Right {outer} Join | |
|Full {outer} Join| |
|Cross Join| |

## Aggregate Functions

| Function|Description|
|---|---|
|APPROX_COUNT_DISTINCT|This function returns the approximate number of unique non-null values in a group.|
|AVG| This function returns the average of the values in a group. It ignores null values.|
|CHECKSUM_AGG|This function returns the checksum of the values in a group. CHECKSUM_AGG ignores null values. The OVER clause can follow CHECKSUM_AGG.|
|COUNT|This function returns the number of items found in a group.|
|COUNT_BIG|This function returns the number of items found in a group.|
|GROUPING| Indicates whether a specified column expression in a GROUP BY list is aggregated or not.|
|GROUPING_ID|Is a function that computes the level of grouping. GROUPING_ID can be used only in the SELECT <select> list, HAVING, or ORDER BY clauses when GROUP BY is specified.|
|MAX|Returns the maximum value in the expression.|
|MIN|Returns the minimum value in the expression. May be followed by the OVER clause.|
|STDEV|Returns the statistical standard deviation of all values in the specified expression.|
|STDEVP|Returns the statistical standard deviation for the population for all values in the specified expression.|
|STRING_AGG|Concatenates the values of string expressions and places separator values between them. The separator is not added at the end of string.|
|SUM|Returns the sum of all the values, or only the DISTINCT values, in the expression. SUM can be used with numeric columns only. Null values are ignored.|
|VAR|Returns the statistical variance of all values in the specified expression. May be followed by the OVER clause.|   
|VARP|Returns the statistical variance for the population for all values in the specified expression.|    

<!-- ## Conditions -->

## Insert Rows

Insert values into a table using passed values

```
INSERT INTO [database_name].[schema].[table_name] (column_a, column_b)
VALUES
('string_value',1)
```
Insert values into a table using a select statement called from another table

```
INSERT INTO [database_name].[schema].[table_name] (column_a, column_b)
SELECT
    column_a,
    column_b
FROM 
    [database_name].[schema].[table_name]
```

## Update Rows

Update a column in a table using a provided value

```
UPDATE [database_name].[schema].[table_name]
SET 
    [column_a] = 'value'
WHERE
    Condition
```

Update a table using a value from another table

```
UPDATE table_a
SET 
    [table_a].[column_a] = [table_b].[column_a]

FROM 
    [database_name].[schema].[table_name] [table_a]

INNER JOIN [database_name].[schema].[table_name] [table_b]
    ON [table_a].[key] = [table_b].[key]

WHERE
    Condition
```

## Delete Rows

Delete rows from a database, when a where condition is not provided all rows will be deleted, if there is an IDENTITY column the IDENTITY will not be reset, this operation also writes to the log

```
DELETE FROM [database_name].[schema].[table_name]
    WHERE 
        Condition
```

## Truncate Table

Truncate a table, this is simillar to the Delete Rows operation except it will delete everything from the table and not write to the log file. 

```
TRUNCATE TABLE [database_name].[schema].[table_name]
```
