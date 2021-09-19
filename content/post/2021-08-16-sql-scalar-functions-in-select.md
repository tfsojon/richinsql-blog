---
title: SQL Server - Using Scalar Functions Inside A Select
date: 2021-08-16T09:00:00+01:00
author: Rich
layout: post
permalink: /sql-scalar-functions-in-select
categories:
  - sqlserver
tags:
  - functions
  - SQL
---

Sometimes Scalar functions are useful, other times however they are not but they do have their uses, sometimes I need to use them inside a select to get a result based on data returned from the database I am querying, but when people read the code back they are not sure how the function is being used or what it means.

For this example, I am going to make use of the 2010 StackOverflow database, within it, I have created a scalar function that will calculate the age of the user
s account based on the date that the account was created. 

The base query is something like this;

```
    SELECT 
        DisplayName,
        --Age,
        CreationDate,
        LastAccessDate,
        Location,
        Reputation,
        UpVotes

    FROM 
        dbo.Users
```

![](assets/img/query-without-function.png)

I have already created the function to calculate the age, for reference, that looks like this - I am not going to explain how that works in this post; 

```
        SET ANSI_NULLS ON
        GO
        SET QUOTED_IDENTIFIER ON
        GO
        CREATE FUNCTION dbo.fn_AccountAge 
        (
            @Date DATETIME
        )
        RETURNS INT
        AS
        BEGIN
            DECLARE @Age INT

            DECLARE @Now datetime
            SET @Now = GETDATE()

            SET @Age = (SELECT
                (CONVERT(int,CONVERT(char(8),@Now,112))-CONVERT(char(8),@Date,112))/10000)

            RETURN @Age

        END
  ```

So how do we call that function inside the select shown in the first query? You can simply call the function like a normal column just like this;

```
    SELECT 
        DisplayName,
        dbo.fn_AccountAge(CreationDate) as Age,
        CreationDate,
        LastAccessDate,
        Location,
        Reputation,
        UpVotes

    FROM 
        dbo.Users
```

![](assets/img/query-with-function.png)

* dbo - When using functions in your code the schema is required, in our case we have created the function on the dbo schema so can specify this.
* fn_AccountAge() - This is the function name 
* () - The brackets in the function name are when the parameters defined within the function schema go, in our case we have specified that a DATETIME value must be passed, we have only requested one input parameter so that is all the function will accept.