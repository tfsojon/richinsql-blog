---
title: Sending Query Results As CSV Using Database Mail
date: 2021-12-13T09:00:00.000+01:00
image: "images/post/Sending-Query-Results-As-CSV-Using-Database-Mail.jpg"
author: Rich
layout: post
permalink: "/sending-results-as-csv"
categories:
- sqlserver
tags:
- SQL
- T-SQL
- DatabaseMail
featured: true
---

Sometimes we have the need to send results which are output from our queries to someone via email, maybe as a CSV. We can do that with SQL Server Agent & Database Mail but first we need to write the code.

This post is going to assume the following; 

* You have SQL Mail setup and configured. If you don't check out [this post](/posts/2021-12-06-configuring-sqlserver-databasemail)
* You know the name of the SQL Mail profile you wish to use.
* You have permission to msdb and Database Mail

**Scenario:** Dumbledore wants to know how many students are taking a which classes in his school, this list might change from week to week so he wants the list emailed to him weekly on a Monday morning.

If you just want the final query, you can get that below but if you want to learn how to put it together follow along. 

### The Final Query

This is the final query that we are going to end up with at the end of this post. 

```
DECLARE 
       @sub VARCHAR(100),
       @msg VARCHAR(250),
       @query NVARCHAR(MAX),
       @query_attachment_filename NVARCHAR(2000),
       @column1name varchar(50)

SET @sub = 'Hogwarts Student Class List'
SET @msg = 'Attached is a list of students and the classes they are enrolled in.'
SET @query_attachment_filename = 'Hogwarts_Student_Class_List.csv'

CREATE TABLE ##students
(
       [StudentID] [varchar](50) NULL,
       [StudentName] [varchar](250) NULL,
       [Class] [varchar](50) NULL
) 

INSERT INTO ##students
SELECT
s.ID,
CONCAT(s.Firstname,' ',s.Surname) as StudentName,
c.ClassName

FROM 
	Hogwarts.dbo.StudentClasses SC

	LEFT OUTER JOIN Hogwarts.dbo.Students S
		ON sc.StudentId = s.Id

	LEFT OUTER JOIN Hogwarts.dbo.Classes C
		ON sc.ClassID = c.ID

ORDER BY 
	s.ID,
	c.ID

SELECT @query = 'set NOCOUNT ON; SELECT TOP 10 c.StudentName ' + @column1name + ' ,c.Class FROM ##students c'

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

First write the query to get our data. It just needs to get all the students and the classes they are enrolled into, standard T-SQL is all we need here, nothing fancy.

```
SELECT
s.ID,
CONCAT(s.Firstname,' ',s.Surname) as StudentName,
c.ClassName

FROM 
	Hogwarts.dbo.StudentClasses SC

	LEFT OUTER JOIN Hogwarts.dbo.Students S
		ON sc.StudentId = s.Id

	LEFT OUTER JOIN Hogwarts.dbo.Classes C
		ON sc.ClassID = c.ID

ORDER BY 
	s.ID,
	c.ID
```

### Variables 

The procedure has some variables which assist in it's function these are used later in the procedure to store some required values. 

|Variable|Description|
|---|---|
|@column1name varchar(50)|This variable will store our excel instructions|
|@sub VARCHAR(100)|To the store the subject of the email|
|@msg VARCHAR(250)|The body of the email |
|@query NVARCHAR(MAX)|The query to generate the output|
|@query_attachment_filename NVARCHAR(2000)|The filename of the attachment|

### Populate Them Vars

Now the variables need to be populated with the values required for the procedure to function 

```
SET @sub = 'Hogwarts Student Class List'
SET @msg = 'Below is a list of students and their class.'
SET @query_attachment_filename = 'Hogwarts_Student_Class_List.csv'
```

### The Excel Workaround 

If the procedure was run as is it would put each row into one column in the CSV and when opened in Excel that would all be in one single column, which isn't what someone receiving the file would want, this is because Excel doesn't understand what the file is and what the columns mean. 
To fix this we need to tell Excel what the file we are sending is, in our case a CSV. To do that we need to pass ```sep=,``` which tells Excel that the seperator to create the columns is a comma. 

```SET @Column1Name = '[sep=,' + CHAR(13) + CHAR(10) + 'StudentName]'```

### Table 

A table to store the results is required, as you can see there are two hashes (##) before the table name. This means that this table is a global temp table which are temp tables accessible across connections. 

```
CREATE TABLE ##students
(
       [StudentID] [varchar](50) NULL,
       [StudentName] [varchar](250) NULL,
       [Class] [varchar](50) NULL
)
```

### The Query Variable

The query that will be used to generate the CSV needs to go into a variable, this is so it can be passed it into sp_send_dbmail at the end of the procedure, sp_send_dbmail will then create and attach the results to the CSV using the parameters provided. 

Setting NOCOUNT to ON also prevents the total number of rows that were returned from the query being included in the output file, that isn't something that we want in the CSV so it is excluded.

```SELECT @query = 'set NOCOUNT ON; SELECT s.StudentName ' + @column1name + ' ,s.Class FROM ##students s'```

### Send The Email 

Now we are ready to actually send the email, to do this we use [sp_send_dbmail](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql?view=sql-server-ver15#arguments) which accepts a wide variety of [parameters](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql?view=sql-server-ver15#arguments) the ones used in this procedure are detailed in the table below, but you can add more or remove the ones you don't need as required. 

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
       @query_result_separator = ',' ,
       @query_result_no_padding = 1
```

|Variable|Description|
|---|---|
|@profile_name|Is the name of the profile to send the message from.|
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

More details on that parameters that sp_send_dbmail accepts can be [found here](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql?view=sql-server-ver15#arguments)

Now that everything is in place, send the email by executing the procedure or putting the code into a SQL Agent Job and you should get an email to the address specified in the @recipients variable with the CSV file attached.

![](/img/csv-result-email.png)

Open up the attachment and the results should look something like this - row two isn't ideal but the output works 

![](/img/csv-excel-result.png)