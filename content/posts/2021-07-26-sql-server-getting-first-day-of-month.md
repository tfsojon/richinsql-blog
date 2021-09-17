---
title: SQL Server Getting First Day Of Month
date: 2021-07-26T09:00:00+01:00
author: Rich
layout: post
permalink: /sql-server-getting-first-day-of-month
categories:
  - sqlserver
tags:
  - SQL
---

So you want to get the first day of the month in SQL server? using DATEADD it is as simple as using the below line of code. 

```
    DATEADD(month,DATEDIFF(month,0,GETDATE()),0)
```

You can replace GETDATE() with any date you wish, for example if you are running a reporting query and need to know what the first date of the month was when an order was placed, simply switch out GETDATE() with your order date column. 

Another example below 

```
DATEADD(month,DATEDIFF(month,0,'20210329'),0)
```

I have also created a scalar function which can be used with any valid date to get the start of that date's month

```
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE FUNCTION fn_StartOfMonth
    (
      @Date DATETIME
    )
    RETURNS DATETIME
    AS
    BEGIN
      DECLARE @MonthStart DATETIME

      SELECT @MonthStart = (SELECT DATEADD(month,DATEDIFF(month,0,GETDATE()),0))

      RETURN @MonthStart

    END
```

Which can then be called using the following syntax

```
    SELECT dbo.fn_StartOfMonth(GETDATE())
```