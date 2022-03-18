---
title: SQL Server Database Mail Format Content As HTML
date: 2024-01-03T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/database-mail-html-format"
categories:
- sqlserver
tags:
- SQL
- T-SQL
- DatabaseMail
- Intermediate
draft: true
---

Have you ever wanted to create an email that can be sent via Database Mail but have a HTML table with your results inside, this could be for a number of reasons but mainly it is because you want the email to look professional. 

In this post, we are going to look at how to add results from T-SQL into a HTML formatted table and send it via database mail. 

### The Final Query

```
BEGIN
	SET NOCOUNT ON;

		DECLARE @Email nvarchar(100)
		DECLARE @Name nvarchar(100)

		CREATE TABLE #tmp
		(
			ID INT IDENTITY(1,1),
			StudentName varchar(50),
			ClassName varchar(50)
		)
		
		INSERT INTO #tmp
		(
		StudentName,
		ClassName
		)
		SELECT

		CONCAT(s.FirstName,' ',s.Surname) as Fullname,
		ClassName

		FROM Hogwarts.dbo.StudentClasses sc 
		
		LEFT OUTER JOIN Hogwarts.dbo.Students s ON s.StudentID = sc.StudentID

		LEFT OUTER JOIN Hogwarts.dbo.Classes c ON sc.ClassID = c.ClassID
		
		/********************************************************
		BUILD THE HTML TABLE TO SEND THE REPORT
		********************************************************/

		BEGIN

		DECLARE @tableHTML nvarchar(MAX), @tableTitle varchar(100),@body nvarchar(MAX)
		
		SET @Name = 'RichInSQL'

		set @tableHTML = '<html><head><style>' +
	   'td {border: solid black 1px;padding-left:5px;padding-right:5px;padding-top:1px;padding-bottom:1px;font-size:11pt;} ' +
	   '</style></head><body>' +
	   'Dear ' + @Name +
	   '<br><br>' + 
	   'Below is a list of students and the classes they are subscribed to' +
	   '<br><br>' +    
	   '<div style="margin-left:50px; font-family:Calibri;"><table cellpadding=0 cellspacing=0 border=0>' +
	   '<tr bgcolor=#EEEDED>' +
	   '<td align=center><font face="calibri" color=Black><b>Student Name</b></font></td>' +    
	   '<td align=center><font face="calibri" color=Black><b>Class Name</b></font></td></tr>' 

		END
		
		--Get the data from the temp table and order it setting the TD as the columns
		select @body =
		(
		   select ROW_NUMBER() over(order by id) % 2 as TRRow,
				   td = StudentName,      
				   td = ClassName   
				   FROM #tmp
		   order by id
		   for XML raw('tr'), elements
		)

		--Set the body of the email ready to send
		set @body = REPLACE(@body, '<td>', '<td align=center><font face="calibri">')
		set @body = REPLACE(@body, '</td>', '</font></td>')
		set @body = REPLACE(@body, '_x0020_', space(1))
		set @body = Replace(@body, '_x003D_', '=')
		set @body = Replace(@body, '<tr><TRRow>0</TRRow>', '<tr bgcolor=#EEEDED>')
		set @body = Replace(@body, '<tr><TRRow>1</TRRow>', '<tr bgcolor=#FFFFFF>')
		set @body = Replace(@body, '<TRRow>0</TRRow>', '')

		--Finish configuring the HTML for the body, closing the table and HTML tags ready for sending.
		set @tableHTML = @tableHTML + @body + '</table></div></body></html>'

		--Apply some styling to the table
		set @tableHTML = '<div style="color:Black; font-size:11pt; font-family:Calibri; width:100px;">' + @tableHTML + '</div>'

		/********************************************************
		SEND THE EMAIL
		********************************************************/
	
		--Set the email with the email address from the #notifywho table
		SET @Email = 'you@yourdomain.com'		

		EXEC msdb.dbo.sp_send_dbmail
				@profile_name = 'MailProfile',
				@from_address = 'Hedwig <hedwig@richinsql.com>',
				@body = @tableHTML,           
				@body_format = 'HTML',
				@subject = 'List of Students & Classes',
				@recipients = @email

END

IF EXISTS (SELECT name FROM tempdb.sys.tables WHERE name like '#tmp%')
BEGIN
	DROP TABLE #tmp
END
```

