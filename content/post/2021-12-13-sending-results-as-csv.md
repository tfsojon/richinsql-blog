---
title: Sending Query Results As CSV Using Database Mail
date: 2021-12-13T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/sending-results-as-csv"
categories:
- sqlserver
tags:
- SQL
- T-SQL
- SQLServerAgent
draft:
    true
---

Sometimes we have the need to send results output from our queries to someone via email maybe as a CSV. We can do that with SQL Server Agent but first we need to write the code.

This post is going to assume the following; 

* You have SQL Mail setup and configured. 
* You know the name of the SQL Mail profile you wish to use.

**Scenario:** Dumbledore wants to know how many students are taking a certain class in his school, this list might change from week to week so he wants the list emailed to him weekly on a Monday morning.

If you want the final query, you can go to the end of this post but if you want to learn how to put it together follow along. 

### The Final Query

This is the final query that we are going to end up with at the end of this post. 

```
DECLARE 
       @sub VARCHAR(100),
       @qry VARCHAR(1000),
       @msg VARCHAR(250),
       @query NVARCHAR(MAX),
       @query_attachment_filename NVARCHAR(2000)

SET @sub = 'Hogwarts Student Class List'
SET @msg = 'Below is a list of students and their class.'
SET @query_attachment_filename = 'Hogwarts_Student_Class_List.csv'

CREATE TABLE ##students
(
       [StudentID] [varchar](50) NULL,
       [StudentName] [varchar](250) NULL,
       [Class] [varchar](50) NULL
) 

INSERT INTO ##students
SELECT

FROM 
       Hogwarts.dbo.Students

SELECT @query = 'select * from ##students'

EXEC msdb.dbo.sp_send_dbmail
       @profile_name = 'Administrator',
       @recipients = 'joe.bloggs@domain.com',
       @copy_recipients = 'jane.doe@domain.com',
       @body = @msg,
       @subject = @sub,
       @query = @query,
       @query_attachment_filename = @query_attachment_filename,
       @attach_query_result_as_file = 1,
       @query_result_header = 1,
       @query_result_width = 1000,
       @query_result_separator = '         ' ,
       @query_result_no_padding = 1

DROP TABLE ##students
```

### The Query 

First we need to write a query to get our data, that is just going to get all the students and the classes they are enrolled into

```
```

### Variables 

We need to create some variables that we will use later in our query to store some values. 

**@sub VARCHAR(100)** - To the store the subject of the email
**@msg VARCHAR(250)** - The body of the email 
**@query NVARCHAR(MAX)** - The query to generate the output 
**@query_attachment_filename NVARCHAR(2000)** - The filename of the attachment

### Populate 

We now need to set the values for these variables 

SET @sub = 'Hogwarts Student Class List'
SET @msg = 'Below is a list of students and their class.'
SET @query_attachment_filename = 'Hogwarts_Student_Class_List.csv'

### Table 

We need to create a table to store our results into, as you can see there are two hashes (##) before the table name. This means that this table is a global temp table which are temp tables accessible accross connections. 

```
CREATE TABLE ##students
(
       [StudentID] [varchar](50) NULL,
       [StudentName] [varchar](250) NULL,
       [Class] [varchar](50) NULL
)
```

### The Query Variable

We need to put the query into a variable, this is becuase we need to pass it into sp_send_dbmail at the end of the procedure 

```SELECT @query = 'select * from ##students'```

### Send The Email 

This section is where we actually send the email, [sp_send_dbmail](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql?view=sql-server-ver15#arguments) which accepts a wide variety of [parameters](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql?view=sql-server-ver15#arguments) the ones we are going to use for this purpose are detailed in the table below. 

```
EXEC msdb.dbo.sp_send_dbmail
       @profile_name = 'Administrator',
       @recipients = 'joe.bloggs@domain.com',
       @copy_recipients = 'jane.doe@domain.com',
       @body = @msg,
       @subject = @sub,
       @query = @query,
       @query_attachment_filename = @query_attachment_filename,
       @attach_query_result_as_file = 1,
       @query_result_header = 1,
       @query_result_width = 1000,
       @query_result_separator = '         ' ,
       @query_result_no_padding = 1
```

|Variable|Description|
|---|---|
|@profile_name|Is the name of the profile to send the message from. |
|@recipients|Is a semicolon-delimited list of e-mail addresses to send the message to.|
|@copy_recipients|Is a semicolon-delimited list of e-mail addresses to carbon copy the message to.|
|@body|Is the body of the e-mail message.|
|@subject|Is the subject of the e-mail message.|
|@query|Is a query to execute. The results of the query can be attached as a file, or included in the body of the e-mail message.|
|@query_attachment_filename|Specifies the file name to use for the result set of the query attachment.|
|@attach_query_result_as_file|Specifies whether the result set of the query is returned as an attached file.|
|@query_result_header|Specifies whether the query results include column headers.|
|@query_result_width|Is the line width, in characters, to use for formatting the results of the query.|
|@query_result_separator|Is the character used to separate columns in the query output.|
|@query_result_no_padding|he type is bit. The default is 0. When you set to 1, the query results are not padded, possibly reducing the file size.|

More details can be [found here](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql?view=sql-server-ver15#arguments)