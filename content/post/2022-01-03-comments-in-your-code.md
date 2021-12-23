---
title: Comments In Your Code
date: 2022-01-03T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/comments-in-your-code"
categories:
- sqlserver
tags:
- sqlserver
- t-sql
draft:
    true
---

You have more than likely written code before but when you have come to test it there was a column that you didn't need, instead of removing the column or line of code you simply commented it out, am I right?

The question is which comment type did you use? In T-SQL there are two types of comment which are accepted 
    * Single dash quotes that start like this -- 
    * Block Quotes which wrap what you want to comment out in these block quotes /* */

To make things worse, in SQL Server Management Studio, if you select the comment button when selecting the code you will get the single dash version but that isn't the best way to go about commenting you code, but now your asking why right?

### Single Quotes

The base query for this test is the same, just select some data from the sys.databases DMV but we are going to single quote the owner_sid, run it, SSMS won't care that it is single quoted, the data won't be returned which I just what you wanted to happen. 

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

Put that same code into some dynamic SQL, this will simulate what would happen should you want to query any of SQL Server's DMV's or system views or even use monitoring tools, now run that in SSMS. 

```
DECLARE @CommentMe NVARCHAR(MAX)
SET @CommentMe = 

'SELECT

name,
database_id,
--owner_sid,
create_date,
compatibility_level,
collation_name

FROM sys.databases'

SELECT @CommentMe

```
![]()

Here lies the problem, SSMS has no idea where that comment that you created with the single quotes actually ends, so everything after owner_sid gets commented out. 

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

Put that code once again, into some dynamic sql and run the coe again

```
DECLARE @BlockComments NVARCHAR(MAX)
SET @BlockCommentMe = 

'SELECT

name,
database_id,
/*owner_sid,*/
create_date,
compatibility_level,
collation_name

FROM sys.databases'
SELECT @BlockCommentMe
```

![]()

This time you will notice that SSMS has ended the comment at the end of the owner_sid line leaving the rest of the query in tact.

You might be thinking this isn't that much of an issue but when you have large queries trying to work out where the start and end of the comment should be can tricky. 

In summary, always use block quotes in your code, it will save a lot of time in the future. 