---
title: What are CTEs & How Do They Work?
date: 2021-10-18T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/what-are-cte"
categories:
- sqlserver
tags:
- SQL
- T-SQL
draft: true

---

A [CTE](https://docs.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql?view=sql-server-ver15) or Common Table Expression specifies a tempoary named result set for the current session, a CTE can reference itself or it can be referenced in a single SELECT, INSERT, UPDATE, DELETE or MERGE statement. A CTE cannot be referenced more than once per session though. 

Let's have a look at how to write one and then we can break it down.

This CTE is using the StackOverflow database to get a count of users grouped by the CreationDate. Lets break it down a little bit. 

```
;WITH CTE_Users (SomeDate, Total)
AS
(
	SELECT		 
		CAST([CreationDate]as DATE), 
		COUNT(1)
	FROM
		[dbo].[Users]
	GROUP BY CAST(CreationDate as DATE)
)
```
When writing a CTE you must always start with the WITH command if this is after any other T-SQL in that query the WITH keyword must be prefixed with a semicolon (;). After the WITH keyword you can specify a name for the CTE this can be anything you want, it is then possible to specify the column names that you would like the CTE to return, these names do not have to reflect the column names of the columns in the SELECT which is housed within the CTE, think of them like aliased columns but they are specified in order of the defined columns in the SELECT. 

```
;WITH CTE_Users (SomeDate, Total) 
AS
```
Once you have specified your columns you must then use the AS keyword followed by and open and close bracket ()

```
AS
(

)
```
Inside the brackets is where you write the query to return the results you would like to use in the CTE, I have written a simple example to get me a count of all users from the users table in StackOverflow grouped by the date they created their accounts. 

```
(
	SELECT		 
		CAST([CreationDate]as DATE), 
		COUNT(1)
	FROM
		[dbo].[Users]
	GROUP BY CAST(CreationDate as DATE)
)
```
Now that the CTE is fully written we can use it in the final query, you would write your query as if you were calling any other SQL object and in the from portion of the query put the name of your CTE. 

```
SELECT 
	* 
FROM 
	CTE_Users 
WHERE 
	Total > 1 
ORDER BY 
	SomeDate
```
The final query looks something like this.

```
;WITH CTE_Users (SomeDate, Total)
AS
(
	SELECT		 
		CAST([CreationDate]as DATE), 
		COUNT(1)
	FROM
		[dbo].[Users]
	GROUP BY CAST(CreationDate as DATE)
)

SELECT 
	* 
FROM 
	CTE_Users 
WHERE 
	Total > 1 
ORDER BY 
	SomeDate
```
 It is possible to use multiple CTE's in the same query, to do this you would put a comma after the closing bracket of the first CTE and specify a new CTE as shown below ***(not that you would write your query like this)***

```
;WITH CTE_Users (ID,SomeDate)
AS
(
	SELECT		 
		ID,
		[CreationDate] 
	FROM
		[dbo].[Users]
	WHERE CAST(CreationDate as DATE) = '2010-03-14'
),
CTE_Users_LastAccessed (ID,CreationDate) AS
(
	SELECT		 
		ID,
		[CreationDate]
	FROM
		[dbo].[Users]
)

SELECT 
	u.id,u.SomeDate,la.CreationDate 
FROM 
	CTE_Users u

	INNER JOIN CTE_Users_LastAccessed la 
		ON u.ID = la.ID
```





