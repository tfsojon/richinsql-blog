---
title: SQL Server Using Comments In Your Code
date: 2022-01-03T09:00:00.000+01:00
image: "/images/post/sql-server-comments-in-your-code.jpg"
author: Rich
layout: post
permalink: "/sql-server-comments-in-your-code"
categories:
- sqlserver
tags:
- SQL
- T-SQL
---

Have you ever written code that when you come to test there was that one column or line of code that you just didn't need? Instead of removing the column or line of code from your query you simply commented it out, am I right? We have all been there. 

The question is which comment type did you use? In T-SQL there are two types of comments which are accepted; 

  - Single dash quotes that start like this -- 
  - Block Quotes which wrap what you want to comment out in these block quotes /* */

To make things worse, in SQL Server Management Studio, if you choose to use the comment button when selecting the line of code to comment out you will be given the single dash version, but that isn't the best way to go about commenting you code, but now your asking why right?

### Single Quotes

The base query for this test is the same for both single quotes and block quotes, just select some data from the sys.databases DMV, but comment using the single quote method the owner_sid column, once you have the code, run it, SSMS won't care that it is single quoted, the data won't be returned which is just what you wanted to happen. 

```
SELECT

name,
database_id,
--owner_sid,
create_date,
compatibility_level,
collation_name

FROM sys.databases
```

Put that same code into some dynamic SQL, this will simulate what would happen should you want to query any of SQL Server's DMV's or system views or even use monitoring tools to find queries for a number of senarios, now run that dynamic SQL in SSMS. 

```
DECLARE @SingleComments NVARCHAR(MAX)
SET @SingleComments = 

'SELECT

name,
database_id,
--owner_sid,
create_date,
compatibility_level,
collation_name

FROM sys.databases'

SELECT @SingleComments

```

You should get something that looks like this;

![](/img/single-quote-result.png)

Copy it, and put it into the query editor

![](/img/single-quote-query.png)

Here lies the problem, SSMS has no idea where that comment that you created with the single quotes actually ends, so everything after owner_sid gets commented out, which to you might not really be a problem but if you have a massive query it might be a bigger issue.

### Block Quotes

Using the same query used above, commenting out owner_sid this time using a block quote, this has a definative start and end point to where the comment should begin and end, denoted by /* and */

Run the query in SSMS, you will first of all get the same results as the first query, owner_sid commented out and no data returned, perfect. 

```
SELECT

name,
database_id,
/*owner_sid,*/
create_date,
compatibility_level,
collation_name

FROM sys.databases
```

Put that code once again, into some dynamic sql and run it.

```
DECLARE @BlockComments NVARCHAR(MAX)
SET @BlockComments = 

'SELECT

name,
database_id,
/*owner_sid,*/
create_date,
compatibility_level,
collation_name

FROM sys.databases'
SELECT @BlockComments
```
You should get something that looks like this;

![](/img/block-quote-result.png)

Copy it out and put it into the query editor again - notice the difference?

![](/img/block-quote-query.png)

This time you will see that SSMS has ended the comment correctly at the end of the owner_sid leaving the rest of the query in tact and not commented.

In summary, always try to use block quotes in your code, it will save a lot of time in the future. 