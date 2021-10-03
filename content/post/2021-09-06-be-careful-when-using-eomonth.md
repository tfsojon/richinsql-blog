---
title: "Be careful when using Eomonth()"
date: 2021-09-06T09:00:00+01:00
author: Rich
layout: post
permalink: /be-careful-when-using-eomonth
categories:
  - sqlserver
tags:
  - functions
  - SQL
summary: In SQL Server 2012 Microsoft added a function called EOMONTH, this returns the end of the month of the date specified. In this post we are going to have a look at how this works. 
---

In SQL Server 2012 Microsoft added a function called EOMONTH, this returns the end of the month of the date specified. 

Lets have a look at what is returned when we call this function, 

```
SELECT 
	EOMONTH(GETDATE()) [eomonth],
	CAST(EOMONTH(GETDATE())as datetime) [datetime],
	CAST(EOMONTH(GETDATE())as datetime2) [datetime2]
```

![](/img/eomonth-1.png)


As you can see the date of the last day of the month is returned but if we cast this to datetime it shows this is actually returned as end of the month at midnight of that day, so any transactions that happens during the last day of the month would be excluded. Let's demo it. 

I am going to create table to hold some dates in date time format

```
CREATE TABLE #t
( 
	ID INT IDENTITY(1,1),
	Dates DATETIME
)
```

Here I am inserting some dates then inserting a row manually at the end of the last day of the month but past midnight, this would replicate a potential transaction taking place on that day.

```
INSERT INTO #t (Dates)
VALUES
(GETDATE()),
(DATEADD(dd,-1,'20210831')),
(DATEADD(dd,-2,'20210831')),
(DATEADD(dd,-3,'20210831')),
(DATEADD(dd,-4,'20210831')),
(DATEADD(dd,-5,'20210831')),
('2021-08-31 07:27:02.920')
```

We just need to make sure that the rows all went in correnty so lets select them out 

![](/img/eomonth-2.png)

As you can see above all 7 rows were inserted including our row for the end of the month. 

Now what happens if we select all rows from our table where Dates is less than or equal to the end of the month

```
SELECT * FROM #t WHERE Dates <= EOMONTH(GETDATE())
```

![](/img/eomonth-4.png)

As you can see, the row where the date is the last day of the month has been excluded becuase the time is AFTER midnight which is what EOMONTH() returns. 

To get around this you would need to cast or convert all of your dates in the where clause to date 

```
SELECT * FROM #t WHERE CAST(Dates as DATE) <= EOMONTH(GETDATE())
```
Which as below, returns all 7 rows correctly

![](/img/eomonth-2.png)





