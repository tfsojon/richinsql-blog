---
title: What's the difference between CAST & CONVERT
date: 2022-04-08T09:00:00.000+01:00
author: Rich
layout: post
categories:
- sqlserver
draft: false
tags:
- SQLServer
- T-SQL
- Beginner
---

In SQL Server there are two options for converting data from one data type to another in your T-SQL query, these are [CONVERT & CAST](https://docs.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql?redirectedfrom=MSDN&view=sql-server-ver15) but what is the difference between the two and how do you use them in your queries? 

### CONVERT

Firstly CONVERT, let's take a look at this function. CONVERT is SQL Server Specific it is also far more flexible than CAST. But what do I mean when I say flexible? Here is an example, let's say we have a DATETIME column that has today's date stored inside it, in it's natural form this would look something like **2022-04-04 07:41:22.297** but what if we want just the date portion? To do that we can use CONVERT and specify the style we want returned.

```SELECT CONVERT(varchar,GETDATE(),112)```

As you can see, we are able to pass the data type we want to convert, the data itself and also the style we want the data to be returned in. It is also possible to just get the time from a DATETIME column using CONVERT to do that we can specify a varchar length and the style.

```SELECT CONVERT(varchar(8),GETDATE(),108)``` 

For more information on DATETIME styles, check the [Microsoft Documentation](https://docs.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql?redirectedfrom=MSDN&view=sql-server-ver15#date-and-time-styles)

### CAST

CAST allows you to convert values from one datatype to another but you can't ask for a specific style to be returned which makes this function less flexibile than CONVERT for example lets take our GETDATE() example again, we want just the date portion of the returned value, to do that we would use this syntax 

```SELECT CAST(GETDATE() AS DATE)```

But if we wanted the ISO specific date format of yyyymmdd that isn't what is returned, instead we get yyyy-mm-dd

### Drawbacks

One thing to be mindful of when using CAST or CONVERT is data types that require precision to be maintained can have that precision lost when IMPLICIT conversion takes place. An example of this would be if you had a column DECIMAL(10,6) in that column you were storing 10.50000 if we cast that to FLOAT you loose the proceeding zeros. 

Here is an example

```
DECLARE @d DECIMAL(10,5)

SET @d = 10.5

SELECT CAST(@d as float)
```

This is shown in detail in the implict conversion chart provided from the Microsoft Documentation

![](https://docs.microsoft.com/en-us/sql/t-sql/data-types/media/lrdatahd.png?view=sql-server-ver15)



