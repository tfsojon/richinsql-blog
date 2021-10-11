---
title: Functions In Views Are Bad
date: 2021-08-30T09:00:00+01:00
author: Rich
layout: post
permalink: /sql-functions-in-views-are-bad
categories:
  - sqlserver
tags:
  - views
  - functions
  - performance
  - SQL
---

One of the things I have spent a lot of time investigating recently is slow running queries, why are some of the queries in the data warehouse slow? They are relivtivly simple, return only a few rows each but take 5-10 minutes to execute. Often they have a function which for example gets the start of the working week for the date passed. 

I set about investigating what was causing this, and it turns out having UDF (User Defined Functions) inside your views is a really bad idea, it forces the view to run serially (single thread)

[Brent Ozar](https://www.brentozar.com/) has a really good [article](https://www.brentozar.com/archive/2017/05/scalar-functions-views-wheres-overhead/) on this, but I wanted to investigate myself and write up my findings. 
 
Lets see this in action!!! For this I am going to use the [StackOverflow database](https://www.brentozar.com/archive/2015/10/how-to-download-the-stack-overflow-database-via-bittorrent/)

So we are going to need a function, I have created a basic on that takes a date and then gives me the day before the date passed in

```
CREATE FUNCTION dbo.fn_GetDate
(
@Date DATETIME
)
RETURNS DATETIME
AS
BEGIN
	DECLARE @Out DATETIME

	SELECT @Out = DATEADD(dd,-1,@Date)

	RETURN @Out
END
```

Next we need a view 

```
CREATE VIEW dbo.LetsGoSerial
AS
SELECT
	*,
	dbo.fn_GetDate(u.CreationDate) as Yesterday
FROM
	dbo.Users u
```

Now let's select the data back out of that view we just created 

```
SELECT 
	id,
	CreationDate,
	Yesterday 
FROM 
	dbo.LetsGoSerial
```

In the query results window you will see that 8,917,507 rows were returned, using BlitzCache lets dive a bit deeper and see what actually happened when that query ran

```
EXEC testing.dbo.sp_BlitzCache @DatabaseName = 'StackOverflow', @ExportToExcel = 1
```

You can see from the results

![](/img/functions-view-blitz.png)

That the function was actually executed for every single row returned even though we only executed the call for the view once, additionally that query was sent serialised.

It is also good to note that it doesn't matter if the query includes the function or not in the output, if it is in the view the same results as above will happen.

Here is the execution plan for the query, the function took up almost 20 seconds of the run time of the query;

![](/img/functions-are-bad-exec-plan.png)

While this might not seem like a big deal on this dataset, running something like this one a complex dataset with many joins or a much large rowset is going to impede performance massively.

My advice, if possible, don't use the functions at all in your queries!