---
title: Whats eating up tempdb
date: 2019-02-20T16:02:38+01:00
author: Rich
layout: post
permalink: /big-tempdb
categories:
  - sqlserver
tags:
  - dba
  - extendedevents
  - SQL
  - sqlserver
  - tempdb
---

So, I have a problem, TempDB is blowing up, I don&#8217;t know what is causing it but I need to figure out why it is happening and getting so big in such a short space of time.

### The Pretext

Honestly, I had no where to start when it came to tracking down what might be causing TempDB to get so big, I knew that I needed something that would track changes but everything I could think of was either not very elegant or added a massive overhead to the instance.

I set about doing some googling and stumbled across a great post over on [Brent Ozar](https://www.brentozar.com) called [Tracking tempdb growth using Extended Events](https://www.brentozar.com/archive/2015/12/tracking-tempdb-growth-using-extended-events/) written by Eric Darling, the only problem with this solution was that it made use of custom Extended Events and I had ever used them before, apart from the ones that come shipped with SQL.

**_I have permission from Brent to use the code from the original article in this blog posting._**

### The Servers

This instance in which this problem resides is part of a two node availability group and has a TempDB configuration with 4 data files which are all set to grow in 256mb increments\* and one log file set to grow in 128mb increments. The starting size of each files is 1GB and the TempDB configuration is the same on both nodes.

- I know that TempDB should be sized for the instance on an individual basis and be configured to fill the disk across the data files, however that isn&#8217;t possible with this instance.

### The Problem

Over the period of one week the TempDB datafile will grow from 4GB spread over 4 files to 25GB spread over 4 files almost over night, while this may not be a problem I want to find out what is causing the rapid growth this is for both inquisitive and preventative reasons, I don&#8217;t want to leave it like this if there something that could be fixed causing this.

### But What Is TempDB

The TempDB is a global system database available to all users, it is a temporary workspace for storing temp tables, work tables that hold intermediate results during a sorting or query processing and materialised static cursors which improves the overall performance of the SQL Server.

SQL Server will record information in the TempDB that will also aid in transaction roll back, it won&#8217;t record any information that will aid in database recovery though, TempDB is re-created each time SQL Server is restarted with a clean copy of the database, TempDB will be created based on model and will reset to it&#8217;s last configured size.

I suppose you can think of TempDB as a massive bucket, everything and anything can get thrown in there, temp tables from all of them queries, cursors, information relating to sort operations lots and lots of different system and user objects. Microsoft provide a useful [document](https://docs.microsoft.com/en-us/sql/relational-databases/databases/tempdb-database?view=sql-server-2017) which explains more about the TempDB database.

### **Let&#8217;s Do This**

To get to the bottom of this, I am going to make use of [Extended Events](https://docs.microsoft.com/en-us/sql/relational-databases/extended-events/extended-events?view=sql-server-2017) this will work for me as the instance with the &#8220;problem&#8221; is running SQL Server 2012 and what is outlined below should work for anything 2012+.

**The Extended Event**

The Extended Event is exactly the same as the one provided in the Brent Ozar post, this is because I had no idea how Extended Events worked or how to go about writing my own Extended Event, what could be captured or what I might need to capture to track down the problem with the growth of TempDB.

The T-SQL provided in the aforementioned post captured everything I needed to diagnose the problem.


```
		CREATE EVENT SESSION [TempDBTest] ON SERVER
		ADD EVENT [sqlserver].[database_file_size_change] 
		(
			ACTION (
				[sqlserver].[session_id], 
				[sqlserver].[database_id], 
				[sqlserver].[client_hostname],
				[sqlserver].[sql_text]
				
				)

			WHERE (
				[database_id] = (2) 
				AND [session_id] &gt; (50))),

		ADD EVENT [sqlserver].[databases_log_file_used_size_changed]
		(
			ACTION (
				[sqlserver].[session_id], 
				[sqlserver].[database_id], 
				[sqlserver].[client_hostname],
				[sqlserver].[sql_text] )

			WHERE (
				[database_id] = (2)))

		ADD TARGET [package0].[asynchronous_file_target]
		(
			SET 
				filename = N'C:\TempDbTest\tempdbtest.xel',
				metadatafile = N'C:\TempDbTest\tempdbtext.xem',
				max_file_size = (10),
				max_rollover_files = 10 )

		WITH 
		(
			MAX_MEMORY = 4096 KB,
			EVENT_RETENTION_MODE = ALLOW_SINGLE_EVENT_LOSS,
			MAX_DISPATCH_LATENCY = 1 SECONDS,
			MAX_EVENT_SiZE = 0 KB,
			MEMORY_PARTITION_MODE = NONE,
			TRACK_CAUSALITY = ON,
			STARTUP_STATE = ON );

		GO
	```

That look&#8217;s pretty complicated right? When I first saw the code it looked pretty crazy to me too so lets see if we can break it down a little bit and see what it is doing.

First off, we need to tell SQL Server that we are going to create a new **event session** and give it a name, in this example I have just used **TempDBTest** after we have done that we need to tell SQL Server that we want it to be created on the server where the T-SQL is being executed.

```
		CREATE EVENT SESSION [TempDBTest] ON SERVER
	```

Next up is the event we want to track my requirement is to track when a database file _(in this case TempDB)_ changes in size.

Inside the event you can see there is an action, this tells the event what it should record when the event is triggered. Finally we have a where clause, as TempDB will _<span style="text-decoration: underline;"><strong>ALWAYS</strong></span>_ have database ID 2 we can simply specify that here so this particular Extended Event will only record database file changes inside database id 2.


```
		ADD EVENT [sqlserver].[database_file_size_change] 
		(
			ACTION (
				[sqlserver].[session_id], 
				[sqlserver].[database_id], 
				[sqlserver].[client_hostname],
				[sqlserver].[sql_text]
				
				)

			WHERE (
				[database_id] = (2)))
	```

The same event is repeated for the log file


```
		ADD EVENT [sqlserver].[databases_log_file_used_size_changed]
		(
			ACTION (
				[sqlserver].[session_id], 
				[sqlserver].[database_id], 
				[sqlserver].[client_hostname],
				[sqlserver].[sql_text] )

			WHERE (
				[database_id] = (2)))
	```

Now that we have our events and actions specified we need to tell the Extended Event what to do with this information once it has it, where do we want it to go? That is handled in the Action.

Filename tells SQL Server where to put that main Extended Event log file, you can specify any location, I used C:\TempDBTest


```
		filename = N'C:\TempDbTest\tempdbtest.xel',
	```

Next up we have the meta data file and where we want that to be stored, pretty much the same as the Extended Event log file, choose a location and specify it.


```
		metadatafile = N'C:\TempDbTest\tempdbtext.xem',
	```

**Max File Size (MB)** &#8211; This tells SQL Server how big the Extended Log File should get before it rolls it over into a new file

**Max Rollover Files** &#8211; This tells SQL Server how many files it should make before it dosen&#8217;t make anymore, useful if you don&#8217;t want the disk to get filled up with lots of files, should you leave that extended event sessions run over the weekend by mistake, but we wouldn&#8217;t do that&#8230;..would we?!


```
		max_file_size = (10),
		max_rollover_files = 10
	```

Now we are happy with our Extended Event and have everything we want captured specified let&#8217;s create it. Once it has been created you can check that it is available by doing the following;

1. Expand the name of the SQL Server
2. Expand the Management folder
3. Expand The Extended Events Folder
4. Check that the Extended Event you specified exists

![](/img/TempDB_ExtendedEvent_Session.png)

If your Extended Event exists you can go ahead and Start it, this will start capturing information right away.

```
		ALTER EVENT SESSION [TempDBTest] ON SERVER STATE = START;
	```

You can use the same T-SQL with **Start** replaced with **Stop** when you would like to stop the Extended Event which will stop any further data from being captured.

```
		ALTER EVENT SESSION [TempDBTest] ON SERVER STATE = STOP;
	```

You can also delete the Extended Event using the following T-SQL, this however _**<span style="text-decoration: underline;">DOES NOT</span>**_ delete the XEL & XEM files that were created while the session was running

```
		DROP EVENT SESSION [TempDBTest] ON SERVER;
	```

**Let&#8217;s Shred**

Now that we have some data in the output file(s) which should look like (below) in the location you specified;

![The output files as shown on disk](/img/tempdb_event_log.png)

We can go ahead and have a look at the results. The Extended Event Log File(s) are essentially just XML so we can shed them in SQL Server Management Studio just like we would XML to get the information from them into a column based results table.

![Taken from my pre-analysis testing.](/img/TempDB_ExtendedEvent_Shred_ExampleOut.png)

This shred is also the same as the code in the [Brent Ozar post](https://www.brentozar.com/archive/2015/12/tracking-tempdb-growth-using-extended-events/), as I used the Extended Event provided in that post it made sense to use the code to shred that data too, I did make some changes to datatypes but other than that it is the same code.

```
		SELECT
			[eventdata].[event_data].[value]('(event/action[@name="session_id"]/value)[1]','INT') AS [SessionID],
			[eventdata].[event_data].[value]('(event/action[@name="client_hostname"]/value)[1]','VARCHAR(100)') AS [ClientHostName],
			COALESCE(DB_NAME([eventdata].[event_data].[value]('(event/data[@name="database_id"]/value)[1]','BIGINT')),'Temp') AS 'SourceDB',
			DB_NAME([eventdata].[event_data].[value]('(event/action[@name="database_id"]/value)[1]','BIGINT')) AS [GrowthDB],
			[eventdata].[event_data].[value]('(event/data[@name="file_name"]/value)[1]','VARCHAR(200)') AS [GrowthFile],
			[eventdata].[event_data].[value]('(event/data[@name="file_type"]/text)[1]','VARCHAR(200)') AS [DBFileType],
			[eventdata].[event_data].[value]('(event/@name)[1]','VARCHAR(MAX)') AS [EventName],
			[eventdata].[event_data].[value]('(event/data[@name="size_change_kb"]/value)[1]','BIGINT') AS [SizeChangedKb],
			[eventdata].[event_data].[value]('(event/data[@name="total_size_kb"]/value)[1]','BIGINT') AS [TotalSizeKb],
			[eventdata].[event_data].[value]('(event/data[@name="duration"]/value)[1]','BIGINT') AS [DurationInMS],
			[eventdata].[event_data].[value]('(event/@timestamp)[1]','VARCHAR(MAX)') AS [GrowthTime],
			[eventdata].[event_data].[value]('(event/action[@name="sql_text"]/value)[1]','VARCHAR(MAX)') AS [QueryText]
		FROM 
			(
			SELECT 
				CAST([event_data] as XML) AS [TargetData] 
			FROM
				[sys].[fn_xe_file_target_read_file]('C:\TempDbTest\tempdbtest*.xel', NULL,NULL,NULL)
			) AS [eventdata] ([event_data])
			WHERE
				[eventdata].[event_data].[value]('(event/@name)[1]','VARCHAR(100)') = 'database_file_size_change'
				OR [eventdata].[event_data].[value]('(event/@name)[1]','VARCHAR(100)') = 'databases_log_file_used_size_changed'
		ORDER BY [GrowthTime] ASC;
	```

### Let&#8217;s Test

**Note:** It is probably not a good idea to go running the following code on that production server you have sitting in the corner, probably best to use it on your test/development instance.

Now we have that out of the way, I needed a way to test the Extended Event so I could see the results, see what was going on and be able to explain it.

The Brent Ozar post had a query that used the StackOverflow database which forced the TempDB database to grow un-naturally, I needed something like that so I came up with the below.

```
		USE [tempdb]

		SET NOCOUNT ON

		IF (OBJECT_ID('tempdb..#names')) IS NOT NULL
		DROP TABLE #Names

		CREATE TABLE #Names
		(
		Forename varchar(100),
		Surname varchar(100)
		)

		DECLARE @RoundWeGo INT
		SET @RoundWeGo = 100

		DECLARE @Dizzy INT
		SET @Dizzy = 1

		DBCC SHRINKFILE('tempdev',1)
		DBCC SHRINKFILE('temp2',1)
		DBCC SHRINKFILE('temp3',1)
		DBCC SHRINKFILE('temp4',1)
		DBCC SHRINKFILE('templog',1)

		WHILE @Dizzy &lt;= @RoundWeGo

		BEGIN

			INSERT INTO #Names (Forename)
			SELECT TOP 1500
			Forename
			FROM 
			MPI_Demo.SourceData.Forenames 

			UPDATE #Names

			SET Surname = S.Surname

			FROM MPI_Demo.SourceData.Surnames S
			WHERE Forename IS NOT NULL

			RAISERROR ('How many spins are we up to %i ',0,1,@Dizzy) WITH NOWAIT

			SET @Dizzy = @Dizzy + 1

		END
	```

#### What&#8217;s all this?

First up we need somewhere to store the data, as we want TempDB to bloat we will make use of a Temp table

```
		CREATE TABLE #Names
		(
		Forename varchar(100),
		Surname varchar(100)
		)
```

**@RoundWeGo** &#8211; This variable is going to define how many times we want to loop the data, I went a bit over the top and specified 100 just to be sure things were going to grow,

**@Dizzy** &#8211; This is our counter, how many times have we been around already?

```
		DECLARE @RoundWeGo INT
		SET @RoundWeGo = 100

		DECLARE @Dizzy INT
		SET @Dizzy = 1
```

We need to shrink them TempDB data files to something small that we know is going to grow when we throw lots of data at it, don&#8217;t go shrinking these files in production of course, this is simply for testing.

```
		DBCC SHRINKFILE('tempdev',1)
		DBCC SHRINKFILE('temp2',1)
		DBCC SHRINKFILE('temp3',1)
		DBCC SHRINKFILE('temp4',1)
		DBCC SHRINKFILE('templog',1)
```

Having this raiserror in the loop will show how many times the loop has been round, it is just good for monitoring what is going on from the messages tab in SSMS

```
		RAISERROR ('How many spins are we up to %i ',0,1,@Dizzy) WITH NOWAIT
```

See&#8230;it just prints out how many times the loop has been around

![](/img/TempDB_ExtendedEvent_Dizzy_Monitor.png)

The rest of the T-SQL will get the **top 1500** Forenames from my names table and insert them into our defined Temp table, once it is has done the select it will update them rows with a Surname from the same names table, this will happen 100 times giving us something like this in our Extended Event log file when we shred the data.

![](/img/TempDB_ExtendedEvent_Shred_DemoOut.png)

#### What Does All That Mean?

Blimey, that is a lot of information to process, what does it all mean though?

- **Session ID** &#8211; The session ID (SPID) assigned to the user who has executed the query that caused the TempDB file to grow
- **Client Host Name** &#8211; The hostname of the machine where the offending query originated.
- **SourceDB** &#8211; The source database where the growth came from
- **GrowthDB** &#8211; The database that grew
- **Growth File** &#8211; The file in the database that grew
- **DB File Type** &#8211; The file type, log or data that grew
- **Event Type** &#8211; The event that took place, in this case it will be database_file_size_change as we are not capturing anything else
- **Size Changed KB** &#8211; The size that the database changed by (in KB)
- **Total Size KB** &#8211; The new size of the file
- **Duration In MS** &#8211; How long the growth event took to complete
- **Growth Time** &#8211; The time that the growth event took place
- **Query Text** &#8211; The offending query

One thing that I learnt along away is that in the **SizeChangedKB** column you may see -65592 (negative numbers) this is a shrink event, if you shrink a data file on the database you are monitoring using an extended event shrink events are recorded when the data file changes size, these events are recorded as negative numbers.

As you can see the query we used for testing caused every single file in TempDB to grow, the size in which that database grew is recorded and the new size of that file is shown.

![](/img/TempDB_ExtendedEvent_Shred_Grow.png)

The offending SQL query is also shown, so you are able to easily track down what query caused the growth to happen.

![](/img/TempDB_ExtendedEvent_Shred_Offending_Query.png)

You are then able to track down on your SQL Server what T-SQL query is causing your TempDB to grow, although we know that TempDB should really be configured to fill the disk in which it resides but as we mentioned earlier this isn&#8217;t possible in every configuration so it is important to understand that not all growth events are bad but this will allow you to see what is causing the growth of the database so that you can monitor and take action if required.

### What I Have Learnt

This project has taught me a lot about Extended Events, how powerful they are, how they can be created and how the data can be viewed right inside SQL Server Management Studio by shredding the XML into something you can easily read.

The above has also taught me that TempDB is a place where lots of different things, which cannot be covered in one post take place, the database can grow uncontrollably if one query spills or writes to a Temp table outside of the expected behavior of the developer.

This lesson has also taught me that TempDB should always where possible be sized for the instance in which it is running, it should be given a dedicated disk and allowed to fill that disk from day one.
