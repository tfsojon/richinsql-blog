---
title: SQL Server Strings And Numbers
date: 2021-09-27T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/strings-and-numbers"
categories:
- sqlserver
tags:
- SQL
- Datatypes

---
Something new happened this week, well not new in the sense like buying a new car but new to me, I had one of the analysts come to me and ask why was their query excluding certain numeric results...let's have a look at what was going on.

First create a table to hold a list of ages, for this we want the table to contain an nvarchar value - I know numbers should be in their correct datatype, bear with me becuase this is what I was working with and it plays into the demo. 

```
CREATE TABLE #t 
(Age nvarchar(3))
```
Now that we have that table we need to put some data into it, we are going to insert a list of ages up to 18 - you could insert as many as you want but this is what I am going to use here. 

```
INSERT INTO #t (Age)
VALUES
('1'),
('2'),
('3'),
('4'),
('5'),
('6'),
('7'),
('8'),
('9'),
('10'),
('11'),
('12'),
('13'),
('14'),
('15'),
('16'),
('17'),
('18')
```

Now, lets select from that table, but here we are going to wrap our INT value in single quotes so basically passing in a string 

```
SELECT age FROM #t where age <= '18'
```

![](/img/excluded-ages.png)

As you can see a large portion of the numbers that we would expect to be returned are missing, specifically anything after 1 and before 10, now let's perform the same query, but this time, remove the single quotation marks telling SQL Server that we want to query a integar value. 

```
SELECT * FROM #t where age <= 18
```

![](/img/execplan-age.png)

As you can see, we have a warning "Type conversion in expression (CONVERT_IMPLICIT(int,[#t].[Age],0)) may affect "CardinalityEstimate" in query plan choice" the column age has been converted to an INT implicitly

![](/img/all-ages.png)

Now, all of the results we expect to be returned are actually returned. This is good news but what is going on? I didn't know the answer either so I went over the SQL Slack to ask and found my answer.

When we pass in a numeric value wrapped in single quotation marks, we are telling SQL Server we want to analyse a string so it is going to take that string and find everything in order of the numbers passed, our example of <=18 actually looks like <=1 then <=8

It is important when working with datatypes to tell SQL Server specificlly what you are trying to do the even better soloution would be to set the data type on the table to a tinyint or int which would remove this problem all together.