---
title: SQL Server Resetting Identity Seed
date: 2022-01-10T09:00:00.000+01:00
image: "/images/post/sql-server-resetting-identity-seed.jpg"
author: Rich
layout: post
permalink: "/sql-server-resetting-identity-seed"
categories:
- sqlserver
tags:
- SQL
- T-SQL
- Beginner
---

In this post we are going to look at re-seeding the identity column on one of our tables, sometimes there is a requirement to delete all rows from a table but we can't use the [truncate](https://docs.microsoft.com/en-us/sql/t-sql/statements/truncate-table-transact-sql?view=sql-server-ver15) method which would re-seed the identity column for us. 

So that nothing gets broken in production, we will create a test table to demonstrate how re-seeding the identity column works.

If you already have a test table that you can use, make sure that it has an IDENTITY column for this demo, if you don't have a test table, you can use the one below. 

```
CREATE TABLE dbo.TestTable
(
	ID INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
	SomeOtherRef INT
)
```

Next, insert some values into that table so that the seed will increment. 

```
INSERT INTO dbo.TestTable (SomeOtherRef)
VALUES
(1),
(2),
(3),
(4),
(5),
(6),
(7),
(8),
(9),
(10)
```
Once the data is inserted we can now check to see what the seed on this table currently is. 

```
DBCC CHECKIDENT ('dbo.TestTable');
GO
```

![](/img/sql-server-resetting-identity-seed-1.png)

Delete all the values from our test table. Doing this won't re-seed the identity column.

```
DELETE FROM dbo.TestTable
```

So if you were to run the CHECKIDENT command again the seed value should still be set to 10. 

```
DBCC CHECKIDENT ('dbo.TestTable');
GO
```

![](/img/sql-server-resetting-identity-seed-1.png)

To re-seed the IDENTITY column we can use the [DBCC CHECKIDENT](https://docs.microsoft.com/en-us/sql/t-sql/database-console-commands/dbcc-checkident-transact-sql?view=sql-server-ver15) function passing in our table name followed by the RESEED command then the value we would like the seed to start at.

```
DBCC CHECKIDENT ('dbo.TestTable', RESEED, 0);
GO
```

![](/img/sql-server-resetting-identity-seed-2.png)

Checking the seed value now shows that the IDENTITY column is now set to 0 

```
DBCC CHECKIDENT ('dbo.TestTable');
GO
```
