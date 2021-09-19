---
title: "SQL Server How to Use Datediff"
date: 2021-09-20T09:00:00+01:00
author: Rich
layout: post
permalink: /how-to-use-dateadd
categories:
  - sqlserver
tags:
  - functions
  - SQL
---

In this post, I want to take a look at datediff, how it works and how I use it as part of my job.  

As per the Microsoft Documentation, "This function returns the count (as a signed integer value) of the specified datepart boundaries crossed between the specified startdate and enddate."

The syntax for **DATEDIFF** looks like this 

```
DATEDIFF ( datepart , startdate , enddate )  
```

Let's break that down, 

- Datepart: this is the part of the date you would like to increment on, for example, the number of days, the number of months or the number of years Microsoft has an [article](https://docs.microsoft.com/en-us/sql/t-sql/functions/datepart-transact-sql?view=sql-server-ver15) that explains all the available dateparts
- StartDate - The date you want to start the calculation from
- EndDate: The date you want to end the calculation at.

So let's say we want to find out how many days there are between the same date today 20 years ago and today, to do that we could write 

```
SELECT DATEDIFF(dd,DATEADD(yy,-20,GETDATE()),GETDATE()) [Days]
```

### How Do I Use DATEDIFF?

This example is getting the start of the previous week by calculating the midnight of the current day then subtracting 6 days

```
SELECT DATEADD(day,-6,DATEADD(wk,DATEDIFF(wk,6,GETDATE()),6))
```
This part of the query

```
DATEADD(wk,DATEDIFF(wk,6,GETDATE()),6)
```
Gets the start of the current day, the start being defined as midnight 

The second part then substracts 6 days from that date

```
DATEADD(day,-6,)
```

Giving us the start of the current week.