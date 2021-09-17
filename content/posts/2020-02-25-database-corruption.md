---
title: Corrupting databases on purpose with dbatools
date: 2020-02-25T22:00:00+00:00
author: Rich
layout: post
permalink: /corrupting-databases-on-purpose-with-dbatools
categories:
  - powershell
tags:
  - Powershell
  - SQL
---

So here we are going to look at corrupting a database and how we can go about restoring the individual page to that database 

We don't want to go corrupting a production database, please don't do that, for this we will create a demonstration database called CorruptionTest

<pre>
    <code class="sql">
		CREATE DATABASE CorruptionTest;
	</code>
</pre>

Now that we have the database created, we need to pop some data in it, so we will create a table 

<pre>
    <code class="sql">
		USE CorruptionTest;

        CREATE TABLE dbo.People
        (
        PersonID INT IDENTITY(1,1),
        FirstName nvarchar(100),
        Surname nvarchar(100),
        Sex CHAR(1)
        );
	</code>
</pre>

Once the table is created, insert a few rows

<pre>
    <code class="sql">
		USE CorruptionTest;

        INSERT INTO dbo.People (FirstName,Surname,Sex)
        VALUES
        ('Chris','Jones','M'),
        ('Claire','Roberts','F'),
        ('Robert','Mcinnerly','M'),
        ('Dave','Smith','M'),
        ('Sally','Johnson','F'),
        ('Sandra','Williams','F'),
        ('Olivia','Brown','F'),
        ('Brian','Davis','M'),
        ('Billy','Miller','M'),
        ('Hayley','Wilson','F');
	</code>
</pre>

Now, let's make sure that the data exists in the table

<pre>
    <code class="sql">
		USE CorruptionTest;
        GO
        SELECT * FROM dbo.People;
	</code>
</pre>

To be able to restore the page, we first need to take a backup of the database

<pre>
    <code class="sql">
		BACKUP DATABASE CorruptionTest
            TO DISK = 'C:\Program Files\Microsoft SQL Server\MSSQL14.MSSQLSERVER\MSSQL\Backup\CorruptionTestCorruptDemo.bak'
            WITH FORMAT;
	</code>
</pre>

Now, load up PowerShell in Administrator mode and paste the following code into the window and let it run.

<pre>
    <code class="powershell">
    $fn = Import-Module dbatools -PassThru
		& $fn {Invoke-DbaDbCorruption -SqlInstance localhost -database CorruptionTest}
	</code>
</pre>

Once the corruption has taken place we can use the following code to find out which page within the database has been corrupted. 

<pre>
    <code class="powershell">
    $fn = Import-Module dbatools -PassThru 
    & $fn {Get-DbaSuspectPage -SqlInstance localhost -Database CorruptionTest}
	</code>
</pre>

If you would like to make sure that the table is corrupt, you can re-run the select and you should be presented with an error message that looks some thing like the below 

<pre>
    <code class="sql">
		USE CorruptionTest;
        GO
        SELECT * FROM dbo.People;
	</code>
</pre>


<pre>
    <code class="sql">
Msg 824, Level 24, State 2, Line 3
SQL Server detected a logical consistency-based I/O error: incorrect checksum (expected: 0x6f10206a; actual: 0x6f10446a). It occurred during a read of page (1:224) in database ID 21 at offset 0x000000001c0000 in file 'C:\Program Files\Microsoft SQL Server\MSSQL14.MSSQLSERVER\MSSQL\DATA\CorruptionTest.mdf'.  Additional messages in the SQL Server error log or operating system error log may provide more detail. This is a severe error condition that threatens database integrity and must be corrected immediately. Complete a full database consistency check (DBCC CHECKDB). This error can be caused by many factors; for more information, see SQL Server Books Online.

	</code>
</pre>

As you can see from the error page 1:224 is corrupt, which is the same as what PowerShell outputted to us when we ran the second command, to fix this we need to restore the page that has been corrupt. 

If you haven't already, load up SQL Management Studio (SSMS) and connect to the instance that CorruptionTest resides, in my case 'localhost'

Expand the databases folder in the item tree

![Database Corruption 1](/img/corruption-1.png)

Right click on the CorruptionTest database, from the menu that appears, select Tasks and choose Restore, from the fly out menu select page

![Database Corruption 2](/img/corruption-2.png)

From the window that appears make sure that the File ID is set to 1 and the Page ID is set to 224, the backup file also needs to point to the backup file that we took earlier, which in this case was C:\Program Files\Microsoft SQL Server\MSSQL14.MSSQLSERVER\MSSQL\Backup\CorruptionTestCorruptDemo.bak

![Database Corruption 3](/img/corruption-3.png)

Once you are happy, you can press OK and the page will be restored.

![Database Corruption 4](/img/corruption-4.png)

Re-running the SELECT that we did earlier should now return all the data that we are expecting. 

<pre>
    <code class="sql">
		USE CorruptionTest;
        GO
        SELECT * FROM dbo.People;
	</code>
</pre>