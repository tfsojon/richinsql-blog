---
title: Inserting data from a select
date: 2019-07-01T16:00:03+01:00
author: Rich
layout: post
permalink: /inserting-data-from-a-select
categories:
  - sqlserver
tags:
  - SQL
  - sqlserver
  - t-sql
---

In SQL Server it is possible to insert data from one table into another using a select within the insert statement, I recently ran into an issue where a stored procedure has been written that didn't specify the columns on the insert side of the statement and simply specified

```
        select * from t
  ```

I want to demonstrate why this might cause a problem.

### Build the tables

t1 which will hold all of the demo data to begin with

```
        CREATE TABLE #t1 
        (
            ID INT IDENTITY(1,1) ,
            OwnerName nvarchar(10),
            Dog BIT,
            Cat BIT,
            Sex CHAR(1),
            AdoptionDate DATETIME DEFAULT GETDATE(),
            PreviousOwner nvarchar(10)
        )
  ```

t2, similar to t1 but this will hold all of the data after the Insert, previous owner is however omitted _(for this demo this is important)_

```
CREATE TABLE #t2 
(
    OwnerName nvarchar(10),
    Dog BIT,
    Cat BIT,
    Sex CHAR(1),
    AdoptionDate DATETIME DEFAULT GETDATE()
)
```

#### Insert all the data

t1 is populated with the dummy data &#8211; I made all of this data up.

<pre>
    <code class="sql">
        INSERT INTO #t1 (OwnerName,Dog,Cat,Sex,PreviousOwner)
        VALUES
        ('John',1,0,'M','Jane'),
        ('Sally',0,1,'F','Mike'),
        ('Hayley',1,0,'F','Simon'),
        ('Becky',0,1,'F','Kelly'),
        ('David',1,0,'M','Julie'),
        ('Nathan',0,1,'M','Lee'),
        ('James',1,0,'M','Bob'),
        ('Victoria',0,1,'F','Judith'),
        ('Katie',1,0,'F','Imogen'),
        ('Barry',0,1,'M','Tilly')
  ```

#### Insert on the select

When I created t1 I purposely didn't create the previous owner column, remember? This script is going to attempt to Insert the data into t2 based on the Select from t1, the columns are specified on the Insert, however the Select is going to return all of the columns from t1 and attempt to insert them into t2

```
    BEGIN TRAN

    INSERT INTO #t2 (OwnerName,Dog,Cat,Sex,AdoptionDate)
    SELECT * FROM #t1

IF @@ERROR = 0 

    BEGIN
        COMMIT TRAN
        SELECT * FROM #t2
    END
ELSE
    BEGIN  
        ROLLBACK TRAN
    END
  ```

As expected, SQL Server has thrown an error;

```
        The select list for the INSERT statement contains more items than the insert list. The number of SELECT values must match the number of INSERT columns.
  ```

![Inserting From Select](/img/Insert-from-select.png)

To fix this, the columns need to be specified on both the Insert & Select which will tell SQL Server &#8220;Hey, I have these columns where I would like to get data from, and here are the columns I would like to put that data&#8221;

This is demonstrated in the below;

```
        BEGIN TRAN

        INSERT INTO #t2 (OwnerName,Dog,Cat,Sex,AdoptionDate)
        SELECT 
            OwnerName
            ,Dog
            ,Cat
            ,Sex
            ,AdoptionDate FROM #t1

    IF @@ERROR = 0 
        BEGIN
            COMMIT TRAN
            SELECT * FROM #t2
        END
    ELSE
        BEGIN  
            ROLLBACK TRAN
    END
```

Just as we would expect, the data was successfully inserted.

![Inserting From Select](/img/Insert-from-select2.png)

#### Drop them tables

Just to keep things nice and tidy drop them temp tables.

```
        DROP TABLE #t1,#t2
  ```

#### To Conclude

If you have a table that you would like to Select data from and Insert that data into another table it is important to specify the columns on both sides of the statement (Select & Insert) to ensure that SQL server knows exactly where that data needs to be Inserted into otherwise it will have selected more data than there is room for it to Insert.
