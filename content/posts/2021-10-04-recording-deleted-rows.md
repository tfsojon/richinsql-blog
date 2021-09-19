---
title: SQL Server Recording Deleted Rows
date: 2021-10-04T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/recording-deleted-rows"
categories:
- sqlserver
tags:
- SQL
- Triggers
draft: true

---
Recently, we had a user delete a number of rows from a production database by mistake, it transpired that they were unsure that they had even made the mistake, this led me to think about how we could audit this. 

This is what I came up with;

To demonstrate, I am going to create an orders table, not very wide, but will be fine for this demo.

```
CREATE TABLE Orders
(
ID INT IDENTITY (1,1),
Part varchar(100),
OrderDate DATETIME DEFAULT GETDATE()
)
```

Lets put some data into this table, again, for the demo we don't need that many rows.

```
INSERT INTO Orders (Part)
VALUES
('Part 1'),
('Part 2'),
('Part 3'),
('Part 4'),
('Part 5'),
('Part 6'),
('Part 7'),
('Part 8'),
('Part 9'),
('Part 10')
```

Now what we are going to create is a carbon copy of the orders table schema but add some columns onto the end to record who deleted the rows and when they were deleted 

```
CREATE TABLE Orphend_Orders
(
ID INT,
Part varchar(100),
OrderDate DATETIME,
DateRemoved DATETIME  DEFAULT GETDATE(),
UserName varchar(100)
)
```

Now on the orders table, we can create a trigger that will fire on delete of any row, be warned that this could potentially slow your delete action down some what but the trade off is worth it in this situation.

```
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

ALTER TRIGGER dbo.DeletedOrders
   ON  dbo.Orders
   AFTER DELETE
AS 
BEGIN

	SET NOCOUNT ON;

    INSERT INTO dbo.Orphend_Orders ([ID], [Part], [OrderDate], [DateRemoved],UserName)
	SELECT 
		ID,
		Part,
		OrderDate,
		GETDATE(),
		/* The user who deleted the rows, if the rows are deleted by a web application this
		will more than likely collect the application pool context */
		SUSER_SNAME()
	FROM 
		deleted
END
```

What this trigger is doing is inserting into the Orphend_Orders table all the data from the orders table that has been deleted, you will see in the from section of the query we reference a table called deleted, this is where SQL server stores the deleted data before it is committed. 

One other thing to be aware of when using SUSER_SNAME() if a web application or application with it's own context into SQL Server it will record that username and not the actual person who deleted the data from the web application, this is meant for delete operations that take place directly in SSMS.

Now that we have all that setup, we can delete some rows from the orders table and see if it works. 

```
DELETE FROM Orders WHERE ID IN (1,2)
```

![](/img/deleted-rows.png)

As you can see below, the two deleted rows are now recorded in the orphened orders table. You could take this further and add a SQL Agent job to check that table for a count of rows between a certain set of dates and email a list of people when rows have been removed.