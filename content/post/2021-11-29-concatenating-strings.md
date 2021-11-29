---
title: Concatenating Strings
date: 2021-11-29T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/concatenating-strings"
categories:
- sqlserver
tags:
- SQL
- T-SQL

---

There are a number of ways to concatenate strings in SQL server and each will give you a different result, in this post we are going to explore what those methods are and how they are different. 

Through this post I am going to use the Hogwarts data that I have used in a previous post to demonstrate. This is what the data stored in that table looks like; 

![](/img/concat-table.png)

*Note that some of the Firstname's and some of the Surname's are blank.*

### Using the + Operator

The first and probably most common method of concatenating a string is to use the + operator, you can do this by specifying [columna] + [columnb] the issue with this method is if either of them columns are NULL the entire string will be returned as NULL.

```
SELECT

    Name,
    Firstname,
    Surname,
    Firstname + Surname as Fullname

FROM Hogwarts.[dbo].[Students] 
```

Executing that query gives the following results; 

![](/img/concat-1.png)

As you can see, Ron & Katie both have NULL fullnames but their Firstname contained a value, this is because using the + operator will return NULL for the entire string when concatenating columns that contain at least one NULL. 

### The CONCAT Function

Let's have a look at the other method of bringing these columns together. 

```
SELECT

    Name,
    Firstname,
    Surname,
    CONCAT(Firstname,' ',Surname) as Fullname

FROM Hogwarts.[dbo].[Students] 
```

Here we are using the [CONCAT function](https://docs.microsoft.com/en-us/sql/t-sql/functions/concat-transact-sql?view=sql-server-ver15), what this will do is concatenate two or more strings into one single string but if any of the values passed are NULL they will simply be omitted and a value returned regardless. 

If we run the query from above, we will see that Ron & Katie both have Fullname values returned now with just Ron & Katie as their surnames are blank. 

![](/img/concat-2.png)

The CONCAT function also has a child function called [CONCAT_WS](https://docs.microsoft.com/en-us/sql/t-sql/functions/concat-ws-transact-sql?view=sql-server-ver15) which is available in SQL Server 2017+ this function will add a provided separator between each value. An example of how to use this is shown below;

```
SELECT

    Name,
    Firstname,
    Surname,
    CONCAT_WS(' ',Firstname,Surname,Firstname,Surname)

FROM Hogwarts.[dbo].[Students] 
```

The difference with this function is that you only need to pass your separating value once and it is added between each value, whereas the CONCAT function requires the separating value to be added between each value else it will create one long string of the values provided. 

![](/img/concat-3.png)