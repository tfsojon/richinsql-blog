---
title: CHAR Limitations
date: 2021-07-19T09:00:00+01:00
author: Rich
layout: post
permalink: /sql-server-char-limitations
categories:
  - sqlserver
tags:
  - SQL
  - Beginner
---

Recently we had an issue raised that a user was getting an error thrown in the application front end saying that **The varchar value (*) could not be converted to datatype int**. We set about looking at the code to try and figure out where the * was coming from, after a little while of searching we couldn't find the * anywhere, it wasn't being passed into the application, nobody had accidently typed it - so what was going on? 

This is where I started running some tests, the application was sending in the value *10* as an INT we knew that, the stored procedure was coded to receive the value also as an INT but what had happened was someone had defined a param inside the stored procedure which was sitting inside some dynamic SQL, the parameter that 10 was then being set to was CHAR(1).

There was a reference table that 10 was being set from by the application, before the support request was raised there was only values 1-9 in that table, so what happens if we change the 10 that the application was passing to 9 do we still get *? Let's try it.

```
    DECLARE @i INT, @c CHAR(1)
    SET @i = 9
    SET @c = @i

    SELECT @c 
```

![SQL Management Studio Output Truncation](/img/sql-server-char-limitations-4.png)

No we don't. So why is this is causing a problem? Well let's run some code and have a look. 

```
    DECLARE @i INT, @c CHAR(1)
    SET @i = 10
    SET @c = @i

    SELECT @c 
```

![SQL Management Studio Output Truncation](/img/sql-server-char-limitations-1.png)

If we run the above SQL Server is going to output the value (*) but why? Well there is truncation occuring as per the [Microsoft Docs](https://docs.microsoft.com/en-us/sql/t-sql/functions/cast-and-convert-transact-sql?view=sql-server-ver15#truncating-and-rounding-results) "When converting character or binary expressions (binary, char, nchar, nvarchar, varbinary, or varchar) to an expression of a different data type, the conversion operation could truncate the output data, only partially display the output data, or return an error." this is illustrated in the table within the docs

![SQL Management Studio Output Truncation](/img/sql-server-char-limitations-5.png)

This is exactly what was happening, if we re-run the above code but set the CHAR parameter to say 10 to account for any INT value will that fix our problem?

```
    DECLARE @i INT, @c CHAR(10)
    SET @i = 10
    SET @c = @i

    SELECT @c 
```

![SQL Management Studio Output Without Truncation](/img/sql-server-char-limitations-2.png)

Well, not really, while the (*) has gone away what is now happening is SQL Server is adding 8 blank CHARS after our value, we can see this if we copy the output from SQL management studio into something like notepad++

![SQL Management Studio Output Without Truncation](/img/sql-server-char-limitations-3.png)

As you can see here, there is 8 additional blank chars after our value of 10. 

The correct fix is of course to change the dynamic SQL in our stored procedure to INT rather than CHAR but it was an interesting lesson in CAST & CONVERT of datatypes and what can happen in certain circumstances, certainly something that whoever had wrote the code before me hadn't considered. 