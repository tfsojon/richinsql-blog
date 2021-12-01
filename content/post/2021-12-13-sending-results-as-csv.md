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

```
DECLARE 
       @sub VARCHAR(100),
       @qry VARCHAR(1000),
       @msg VARCHAR(250),
       @query NVARCHAR(MAX),
       @query_attachment_filename NVARCHAR(2000),
       @datestart DATE,
       @dateend DATE,
       @Month varchar(20),
       @Year varchar(20)

SET @sub = 'Email Subject'
SET @msg = 'Body Of the email can go here.'

CREATE TABLE ##students(
       [PatientNo] [varchar](50) NULL,
       [Complications] [varchar](250) NULL,
       [Ward] [varchar](50) NULL,
       [Date] [varchar](50) NULL,
       [Seen By Doctor] [varchar](50) NULL
) ON [PRIMARY]


SET @datestart = (Select dateadd(d,-(day(dateadd(m,-1,getdate()-2))),dateadd(m,-1,getdate()-1)))
SET @dateend = (Select dateadd(d,-(day(getdate())),getdate()))
SET @Month = (SELECT left(datename(month, dateadd(MONTH, -1, getdate())), 10))
SET @Year = (SELECT DATEPART(yy,GETDATE()))

INSERT INTO ##students
SELECT TOP 500

FROM 
       Hogwarts.dbo.Students

SELECT @query = 'select * from ##temp'

SELECT @query_attachment_filename = 'File_Name_Of_Attachment'+' '++@Month++' '++@Year+'.csv'

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





