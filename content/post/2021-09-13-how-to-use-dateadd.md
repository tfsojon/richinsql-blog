---
title: "SQL Server How to Use Dateadd"
date: 2021-09-13T09:00:00+01:00
author: Rich
layout: post
permalink: "/how-to-use-dateadd"
categories:
  - sqlserver
tags:
  - functions
  - SQL
---

When I first starting learning T-SQL things like DATEADD took a really long time for my brain to pick up, I wanted to share how they work and how I use them in this post so that others can benefit. 

<!--more-->

As per the Microsoft Documentation, "This function adds a specified number value (as a signed integer) to a specified datepart of an input date value, and then returns that modified value."

The syntax for **DATEADD** looks like this 

```
DATEADD (datepart , number , date )  
```

Let's break that down, 

- Datepart: this is the part of the date you would like to increment on, for example, the number of days, the number of months or the number of years Microsoft has an [article](https://docs.microsoft.com/en-us/sql/t-sql/functions/datepart-transact-sql?view=sql-server-ver15) that explains all the available dateparts
- Number: This is the number you want to increment by, you can also substract using DATEADD
- Date: This is the date you want to modify, you can pass in a date from an existing query, for example order date or you can use GETDATE() to use the current date.

### 7 Days From Today

Let's say we want to get 7 days from today, to do that we would use the following syntax 

```
SELECT DATEADD(dd,7,GETDATE())
```

### 7 Days Prior To Today

We can also get 7 days before today, to do this we can pass in -7 into the number portion of the function

```
SELECT DATEADD(dd,-7,GETDATE())
```

### 1 Month From Today

Getting one month from today

```
SELECT DATEADD(mm,1,GETDATE())
```

### 1 Year From Today

Getting one year from today

```
SELECT DATEADD(yy,1,GETDATE())
```

**Note:** DATEADD returns DATETIME so if you need time precision in your results take this into account. 

### How Do I Use DATEADD?

In my job, I use DATEADD all the time to calculate starts and ends of periods, for example, in my post [SQL Server Getting Beginning Of Previous Week](http://localhost:1313/posts/2021-08-02-sql-server-start-of-previous-week/) I show how to calculate the beginning of the previous week, this uses DATEADD.

