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

You have more than likely wrote code before but when you have come to test it there wa a column that you didn't need, instead of removing the column or line of code you simply commented it out, am I right?

The question is which comment type did you use? In T-SQL there are two types of comment which are accepted 
    * Single dash quotes that start like this -- 
    * Block Quotes which wrap what you want to comment out in these block quotes /* */

In SSMS, if you select the comment button when selecting code you will get the single dash version but that isn't the best way to go about commenting you code, but now your asking why right?

```
DECLARE 
	@CommentMe nvarchar(MAX),
	@BlockCommentMe nvarchar(MAX)
```

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

```
SET @CommentMe = 

'SELECT

name,
database_id,
--owner_sid,
create_date,
compatibility_level,
collation_name

FROM sys.databases'
```

```
SELECT @CommentMe
```

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

```
SET @BlockCommentMe = 

'SELECT

name,
database_id,
/*owner_sid,*/
create_date,
compatibility_level,
collation_name

FROM sys.databases'
```

```
SELECT @BlockCommentMe
```