---
title: Clarion Date Time To SQL DateTime
date: 2021-11-08T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/clarion-date-time"
categories:
- sqlserver
tags:
- SQL
- DateTime
- T-SQL
- Beginner
---

Clairon date is the number of days that have elapsed since December 28, 1800, in SQL Server this could be stored as an INT for example 17/10/2021 would translate to 80647. But how do we get the Clarion date back into Date Time? 

<!--more-->

First, lets create a table to hold some example dates and times

```
CREATE TABLE #d 
(
CDate INT,
CTime INT
)
```

Now that the table is created, we can go ahead and put some data into it, we are just going to insert one single row as that is all that is required for this example. 

```
INSERT INTO #d (CDate,CTime)
VALUES
(80647,2904601)
```

So now that the Clarion dates are into the table, how do we go about getting them back out into a SQL Server Date format? To do that, we are going to make use of the DATEADD function in SQL Server, the below query will return the Date & Time in a SQL Date Time format. 

```
DATEADD(MILLISECOND,(CTime-1)*10,DATEADD(DAY,CDate,'1800-12-28T00:00:00.000'))
```

If we want just the date, we can use the following this will be returned as a DATETIME but the time element will be set to midnight, we could of couse cast this to a DATE should we want just the date. 

```
DateAdd(day, CDate  - 4, '1801-01-01')
```

The final, complete query would look something like this, which would get the DatePlusTime and the Date as two seperate columns. 

```
SELECT 

DATEADD(MILLISECOND,(CTime-1)*10,DATEADD(DAY,CDate,'1800-12-28T00:00:00.000')) as [DatePlusTime],
DateAdd(day, CDate  - 4, '1801-01-01') as [Date] 

FROM #d
```

Don't forget to tidy up after yourself.

```
DROP TABLE #d
```