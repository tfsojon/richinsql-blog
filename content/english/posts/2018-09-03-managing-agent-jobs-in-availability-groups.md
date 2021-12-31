---
title: Managing Agent Jobs in Availability Groups
date: 2018-09-03T10:00:45+01:00
author: Rich
layout: post
permalink: /managing-agent-jobs-in-availability-groups
image: /wp-content/uploads/2018/09/Managing-Agent-Jobs-in-Availability-Groups-1200x280.png
categories:
  - sqlserver
tags:
  - AvailailibtyGroup
  - Server
  - SQL
  - SQLAgent
  - StoredProcedure
---

Availability groups and SQL Server Agent Jobs, how do you go about managing them? In our environment we have two availability group members, for the purposes of this post we will call them S1 and S2 one is always primary and the other is the read-only secondary if the availability group fails over the roles will change, but you knew that right?

So **S1** is the primary and **S2** is the secondary, I had a number of SQL Server Agent jobs that existed on both members, the job would run on both members but always fail on the secondary, as my jobs are setup to alert when the job fails we were getting alerted that the job had failed on S2 but really it hadn't it just wasn't able to run on S2 because that was in a read-only state.

### So what can we do?

We could write a check into each job to see if it was the primary every time it runs, setting this at Step 1 so if Step 1 failed the job would exit but this would become messy, I wanted something we could manage centrally.

### Break out the Utility Database

Our availability group has a Utility Database, this is where all the DBA stored procedures and views that we use are stored, you may have one called something similar, or you may not have one at all but it is useful to create one if not, it keeps everything nice and tidy.

The Utility database is synchronized as part of the Availability Group so I set to looking for a solution that would allow me to use this database to manage my jobs.

### The Setup

**_This stored procedure uses a while loop &#8211; there may be a better way to do this but this way works really well for me._**

Inside the Utility Database, we needed to create a table to store the names of the SQL Server Agent Jobs we wanted to disable on the secondary.

```
		CREATE TABLE [dbo].[ExcludedJobs]
		(
		[ID] [int] IDENTITY(1,1) NOT NULL PRIMARY KEY,
		[Job_name] [nvarchar](128) NOT NULL,
		[Active] [bit] NULL
		)
```

Now that we have our table we can populate it with some of the jobs we would like to manage, the active column on the table means that that job will be included in the check.

```
		INSERT INTO [DBA_Tasks].[dbo].[ExcludedJobs] (Job_Name, Active)
		VALUES
		('Job Name Here', 1)
```

Once you have populated the table with all of the jobs you would like to include in the check, we will need to create the stored procedure that will enable/disable the jobs depending on the member status. I have included an example below.

![](https://www.codenameowl.com/wp-content/uploads/2018/09/Availability_Groups_And_Server_Agents_10-300x98.jpg")

#### What Is Going On?

Alright, let me try and explain what that stored procedure is doing, I will break it down into manageable chunks

```
		USE msdb

		-- Change DBA_Tasks if you already have a Utility Database
		GO
			
			IF DB_ID ('DBA_Tasks') IS NULL

				BEGIN

					Print 'Database DBA_Tasks has been created'

					CREATE DATABASE	[DBA_Tasks];

				END

		GO
```

The first section is going to check to make sure that the DBA_Tasks database exists, if it doesn't it will create it, of course, if your utility database is called something else you will want to go ahead and change that section.

```
		USE [DBA_Tasks]

			IF OBJECT_ID ('dbo.p_ExcludedJobCheck') IS NOT NULL 

				BEGIN

					Print 'Procedure p_ExcludedJobCheck already exits, we will drop it and add the version from this script'

					DROP PROCEDURE [dbo].[p_ExcludedJobCheck]

				END
```

At this point it will also check to see if you already have the stored procedure p_ExcludedJobs in the DBA_Tasks database, if it does exist already it will be dropped and re-created.

```
		IF OBJECT_ID ('dbo.ExcludedJobs') IS NULL

			BEGIN

				PRINT 'Required table Excluded Jobs dosen''t exist, we will create that now'

				CREATE TABLE [dbo].[ExcludedJobs]
				(
					[ID] [int] IDENTITY(1,1) NOT NULL PRIMARY KEY,
					[Job_name] [nvarchar](128) NOT NULL,
					[Active] [bit] NULL
				)

			END

		GO
```

We also need to make sure that the required table exists, as mentioned in the Setup section, [dbo].[ExcludedJobs] is used to store the name of the jobs we would like the stored procedure to manage for us, if this table is missing it won't have any jobs to manage.

```
		CREATE PROCEDURE [dbo].[p_ExcludedJobCheck]

		AS

		BEGIN

			SET NOCOUNT ON;

			DECLARE
				@Availability_Role nvarchar(20)
				,@Job_Name NVARCHAR(128)
				,@SQLEnabled NVARCHAR(250)
				,@SQLDisabled NVARCHAR(250)
				,@Counter INT
				,@MaxID int
```

These are the variables we are going to use in the Stored Procedure, nothing too fancy going on here.

```
		CREATE TABLE #Excluded_Jobs
		(ID INT IDENTITY(1, 1) NOT NULL,
		Job_Name NVARCHAR(128) NOT NULL,
		Active BIT
		);
```

#Excluded_Jobs is a SQL Temp Table, these are per session tables that are deleted when the session ends or when they are dropped as part of the executing batch, this table is going to be used to store all of the jobs we have in the Excluded Jobs table that we created earlier so that we can loop through them.

```
		INSERT INTO #Excluded_Jobs (Job_Name)
		SELECT 
			Job_Name 
		FROM 
			[DBA_Tasks].[dbo].[ExcludedJobs] 
		WHERE 
			Active = 1
```

We now need to insert all of the jobs we are managing into the temp table, you will notice we are only getting jobs where active = 1, if for some reason someone has decided that a job shouldn't be disabled/enabled automatically and set active = 0 we don't want to include that in this check.

```
		SET @Counter = 1;
		SET @MaxID =
		(
		SELECT MAX(ID)
		FROM #Excluded_Jobs
		);
```

Now we have two variables

@MaxID &#8211; We will set this to the MAX value of the ID column from #Excluded_Jobs essentially this will give us the number of passes we need to do before ending the script, for example, if #Excluded_Jobs has 4 rows @MaxID would be set to 4

@Counter &#8211; Initially we are setting this to 1 it will then increment until it reaches the value of @MaxID then the stored procedure will end, this allows us to loop through all of the jobs we are managing and disable/enable them based on the availability group member status.

```
		SET @Availability_Role = 
			(
			SELECT 
				ars.role_desc
			FROM 
				sys.dm_hadr_availability_replica_states AS ars INNER JOIN
				sys.availability_groups AS ag ON ars.group_id = ag.group_id
			WHERE 
				ag.name = 'InstanceName' 
				AND ars.is_local = 1
			)
```

Availability_Role  &#8211; We need to find out what the member status is of the node where the agent job is running, you will need to change the name **ag.name = &#8216;InstanceName'** to the name of your availability group else NULL will be returned and nothing will happen when the job runs.

```
		WHILE @Counter &lt;= @MaxID
```

This is where the loop begins, we are checking to make sure that @Counter is less than or equal to @MaxId

```
		BEGIN
			SET @Job_Name = (
				SELECT 
					Job_Name 
				FROM 
					#Excluded_Jobs 
				WHERE 
					ID = @Counter
				)

			IF @Availability_Role = 'PRIMARY'

				BEGIN

					SET @SQLEnabled = 'EXEC msdb..sp_update_job @job_name =' + '''' + @Job_Name + '''' +  ', @enabled = 1'				

					EXEC sp_executesql @SQLEnabled

				END


			ELSE

				BEGIN

					SET @SQLDisabled = 'EXEC msdb..sp_update_job @job_name =' + '''' + @Job_Name + '''' + ', @enabled = 0'				

					EXEC sp_executesql @SQLDisabled

				END

				SET @Counter = @Counter + 1

		END
```

The next section is where the jobs are either enabled or disabled depending on **@Availability_Role** at the very top we are going to get the @Job_Name we do this by selecting Job_Name from #Excluded_Jobs where the ID of the row equals the ID of @Counter.

Once we have this we check to see what **@Availability_Role** is set to, if it is Primary we are going to Enable the Job, if **@Availability_Role** is not set to Primary we are going to disable the job.

```
		SET @Counter = @Counter + 1
```

Once the Enable or Disable of the job is done we will increment the @Counter so that the loop can continue or end, depending on if the WHILE is matched.

```
		DROP TABLE  #Excluded_Jobs
```

If the @Counter is greater than @MaxID we will end and drop the table #Excluded_Jobs, that will then end the procedure.

### Fitting It All Together

Now that you have all the required elements to make sure that this process is going to work we need to fit it all together.

Your going to need to create a SQL Server Agent Job that will call the stored procedure we created above, when creating the Agent Job I usually set it to run once every minute so that if the Availability Group fails over the jobs will be disabled or enabled before too much goes wrong.

Let's show you how to do that then yeah?

![](/img/Availability_Groups_And_Server_Agents_00.jpg)

First, On the object explorer in SQL Management Studio expand the SQL Server Agent by clicking the +

![](/img/Availability_Groups_And_Server_Agents_01.jpg)

Next, you will see a bunch of folders appear, one of which is called Jobs, right-click that

![](/img/Availability_Groups_And_Server_Agents_02.jpg)

A context menu will appear, from this menu select New Job.

![](/img/Availability_Groups_And_Server_Agents_03.jpg)

This will launch a new window where we can setup our job, enter a name in the name field, I have gone ahead and called mine Availability Member Check, you can call it whatever you want, I have also changed the Owner to sa (I have renamed my sa account)

![](/img/Availability_Groups_And_Server_Agents_04.jpg)

Next, from the new job window, select Steps from the Select a page pane on the right-hand side of the window. Once you have clicked Steps from the bottom select New.

![](/img/Availability_Groups_And_Server_Agents_06.jpg)

1. Enter a name for the step, I call called this Run Stored Procedure
2. Choose the Database in which the stored procedure lives in, DBA_Tasks should exist if you have followed this post all the way, if not select your utility database.
3. Enter the following T-SQL

```
	EXEC dbo.ExecludedJobCheck
```

4. Once you have completed that, Click OK

![](/img/Availability_Groups_And_Server_Agents_07.jpg)

From the side panel, of the New Job window, select Schedules, then click New

![](/img/Availability_Groups_And_Server_Agents_08.jpg)

1. Enter a name for the step, I have called mine Every Minute

2. From the Frequency Section, change the Occurs drop down to Daily

3. From the Daily Frequency section, select the Occur Every radio button and change the drop down to minutes then enter 1 into the box provided.

4. Once you are happy Click Ok

![](/img/Availability_Groups_And_Server_Agents_09.jpg)

Finally, select the Notifications page from the side bar on the New Job window.

1. Check the email box

2. Choose the notification group who you would like to Notify in the event that this job fails, I have selected The DBA Team.

Once you are happy, Click OK the new job will now be created.
