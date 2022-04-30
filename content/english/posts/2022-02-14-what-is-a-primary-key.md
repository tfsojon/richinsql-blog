---
title: What is a primary key
date: 2022-02-28T09:00:00.000+01:00
image: "/images/post/apple-health-powerBi-dashboard.jpg"
author: Rich
layout: post
categories:
- sqlserver
draft: true
tags:
- SQLServer
- T-SQL
- Advanced
---

In Database Design you will often see tables with a **PRIMARY KEY** these can be seen in the database schema like the below 

```
[TABLE NAME] (PK,[DATATYPE] [DATATYPE_LENGTH],[NULL|NOT NULL])
```

But what exactly is a **PRIMARY KEY** and when should it be used? 

### Primary Key

A **PRIMARY KEY** is a field within the database that uniquely identifies each row/record. 

Primary keys can only contain unique values, you cannot store NULL values in a **PRIMARY KEY** column and each table in a database can only have one **PRIMARY KEY**.

If a table has a **PRIMARY KEY** defined on any of the columns then you cannot have two records with the same **PRIMARY KEY** value. 

### Creating a primary key 

You can create a primary key a couple of ways. 

The first way is to create the key at the same time you create the table, this is shown below. 

```
CREATE TABLE dbo.People
(
    ID INT NOT NULL PRIMARY KEY,
    Firstname varchar(255) NOT NULL,
    Surname varchar(255),
    Active BIT DEFAULT 1
)
```

Another way to create a **PRIMARY KEY** would be to create the table

```
CREATE TABLE dbo.People
(
    ID INT NOT NULL,
    Firstname varchar(255) NOT NULL,
    Surname varchar(255),
    Active BIT DEFAULT 1
)
```

Then create the primary key after, this method would be used if for example the table already existed. 

```
ALTER TABLE dbo.People ADD CONSTRAINT PK_People_ID PRIMARY KEY (ID)
```

We can also add a **PRIMARY KEY** to multiple columns, to this we would use the following syntax

```
CREATE TABLE dbo.People
(
    ID INT NOT NULL,
    Firstname varchar(255) NOT NULL,
    Surname varchar(255),
    Active BIT DEFAULT 1
)

ALTER TABLE dbo.People ADD CONSTRAINT PK_People_ID_Firstname PRIMARY KEY (ID,Firstname)
```

### Unique Values Are A Must

As mentioned above, all values on the **PRIMARY KEY** column must be unique, so what happens if we try and add a row with the same **PRIMARY KEY** multiple times? Let's find out...

In this example, we are going to try inserting the same ID into the table more than once. 

```
INSERT INTO dbo.People (ID,Firstname,Surname)
VALUES
(1,'John','Jones')
(1,'Abbey','Jones')
```

You will be presented with an error message that reads something like 

```
Msg 2627, Level 14, State 1, Line 11
Violation of PRIMARY KEY constraint 'PK_People_ID'. Cannot insert duplicate key in object 'dbo.People'. The duplicate key value is (1).
The statement has been terminated.
```

The statement will be terminated and none of the values inserted into the table. 

If we change our insert and make the values unique 

```
INSERT INTO dbo.People (ID,Firstname,Surname)
VALUES
(1,'John','Jones'),
(2,'Abbey','Jones')
```

We get a very different response from SQL Server and our values get successfully inserted into the table. 

```
(2 rows affected)

Completion time: 2022-04-30T19:06:48.8708207+01:00
```

### Dropping a primary key

To remove a primary key from a table you can use this syntax 

```
ALTER TABLE dbo.People DROP CONSTRAINT PK_People_ID_Firstname
```

The primary key will be removed from the table with immediate effect.