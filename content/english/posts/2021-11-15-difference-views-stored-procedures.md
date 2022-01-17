---
title: Differences between views and stored procedures
date: 2021-11-15T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/difference-views-stored-procedures"
categories:
- sqlserver
tags:
- SQL
- T-SQL
- Beginner
---

In Microsoft SQL Server it is possible to create views and also stored procedures, but what are the differences between the two?

<!--more-->

## Views 

A view is a filtered layer ontop of a table, you could think of a view like a virtual table with a set of filters added. Unless a view is indexed the view does not exist as a stored set of data. 

Views are often used to provide a trimmed down version of a dataset to a subset of users, as it is possible to assign distinct permission on views it is then possible to restrict users access to the underlying data so that they can only access the data provided by the view through that particular view even though the view is calling the data from another source. 

### Types of views

* Indexed Views
    - An indexed view is a view that has been materialized. This means the view definition has been computed and the resulting data stored just like a table. [[1]](https://docs.microsoft.com/en-us/sql/relational-databases/views/views?view=sql-server-ver15)
* Paritioned Views
    - A partitioned view joins horizontally partitioned data from a set of member tables across one or more servers. This makes the data appear as if from one table. [[2]](https://docs.microsoft.com/en-us/sql/relational-databases/views/views?view=sql-server-ver15)
* System Views
    - System views expose catalog metadata. You can use system views to return information about the instance of SQL Server or the objects defined in the instance. [[3]](https://docs.microsoft.com/en-us/sql/relational-databases/views/views?view=sql-server-ver15)

### Example View

```
CREATE VIEW dbo.vw_SalesData 

AS

SELECT
    c.Customer_ID,
    c.Customer_Name,
    c.Customer_Primary_Contact_Method,
    c.Customer_Address,
    s.Order_Line_1,
    s.Order_Line_2,
    s.Order_Line_3,
    s.Basket_Sub_Total,
    s.Shipping_Total,
    s.Order_Total
FROM
    dbo.Sales s

LEFT OUTER JOIN dbo.Customers c ON
    s.Customer_ID = c.ID
```

## Stored Procedures

Stored Procedures in Microsoft SQL Server are totally different to views, while you can just like a view write your T-SQL statements to return a dataset, stored procedure are a series of T-SQL statements or a reference to a Microsoft .NET framework common runtime language (CLR) 

Stored procedures can also;

* Contain Parameters
* Perform UPDATES, INSERTS & DELETES
* Call other stored procedures
* Return values to the calling application or procedure

Stored procedures have a number of key benefits 

* Stronger Security
* Reuse of code
    - No need to re-write the same programmable procedures over and over again
* Easier to maintain
    - Changes to the database only need to be reflected in the procedures, the application layer can remain unchanged.
* Improved Performance
    - A procedure compiles the first time it is executed and creates an execution plan that is reused for subsequent executions [[4]](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/stored-procedures-database-engine?view=sql-server-ver15)


### Types of stored procedures

* User-Defined
    - A user-defined procedure can be created in a user-defined database or in all system databases except the Resource database. [[5]](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/stored-procedures-database-engine?view=sql-server-ver15)
* Temporary
    - Temporary procedures are a form of user-defined procedures. The temporary procedures are like a permanent procedure, except temporary procedures are stored in tempdb. [[6]](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/stored-procedures-database-engine?view=sql-server-ver15)
* System
    - System procedures are included with SQL Server. They are physically stored in the internal, hidden Resource database and logically appear in the sys schema of every system- and user-defined database. [[7]](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/stored-procedures-database-engine?view=sql-server-ver15)
* Extended User-Defined
    - Extended procedures enable creating external routines in a programming language such as C. [[8]](https://docs.microsoft.com/en-us/sql/relational-databases/stored-procedures/stored-procedures-database-engine?view=sql-server-ver15)

### Example Stored Procedure

```
CREATE PROCEDURE dbo.p_SalesData 

@Customer_ID INT,
@Order_Total DECIMAL(16,3)


AS

SET NOCOUNT ON;

SELECT
    c.Customer_ID,
    c.Customer_Name,
    c.Customer_Primary_Contact_Method,
    c.Customer_Address,
    s.Order_Line_1,
    s.Order_Line_2,
    s.Order_Line_3,
    s.Basket_Sub_Total,
    s.Shipping_Total,
    @Order_Total = s.Order_Total
FROM
    dbo.Sales s

LEFT OUTER JOIN dbo.Customers c ON
    s.Customer_ID = c.ID

WHERE 
    c.Customer_ID = @Customer_ID

```

## So, what is the difference?

To recap, a view is like a layer ontop of a table, it is only possible to return a dataset from the Database, it isn't possible to ask the view to return you specific results that would change based on critria of the calling application, for example, if you wanted all orders for a specific customer using a view that wouldn't be possible unless the customer_ID was specifically hard coded into the views where clause. 

A stored procedure however can accept incoming parameters, using our example above it would be possible to pass the customer_id into the store procedure and get the details of that order returned. 

In the example stored procedure above we are passing in a customer_ID and then getting the order_total returned so when the stored procedure is executed the order_total for the provided customer_id would be returned to the application layer or user calling that procedure, that isn't possible with a view. 