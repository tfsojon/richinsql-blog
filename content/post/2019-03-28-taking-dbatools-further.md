---
title: Taking dbatools further
date: 2019-03-28T16:00:09+01:00
author: Rich
layout: post
permalink: /taking-dbatools-further
categories:
  - powershell
tags:
  - dba
  - dbatools
  - powershell
  - SQL
  - sqlserver
---
Last week I wrote about how I was [getting started with dbatools](https://www.codenameowl.com/getting-started-with-dba-tools) a lot has happened since then, I mean I have done some more reading and tried a few new things and I wanted to expand on some of the really cool things this module can do.

### Tables here, tables there

Some times for whatever reason I need to be able to script a table out, this involves loading up SSMS, connecting to the instance, getting to the database, right clicking&#8230;you get the idea it is a long drawn out task, but what if I could do that with just one line of code? Well with dbatools I can do just that, let&#8217;s have a look.

<pre> 
	<code class="ps">
		Get-Dbadbtable -sqlinstance localhost -database StackOverflow2010 -table dbo.users | export-dbascript -path C:\temp\tables\so-users.sql -append
	</code>
</pre>

![](/img/table-script-out.png)

So the script has executed, but what exactly has it done? If I navigate to C:\temp\tables and open up so-users.sql I will find the following;

<pre>  
	<code class="sql">
		/*
			Created by DESKTOP\BonzaOwl using dbatools Export-DbaScript for objects on DESKTOP at 03/27/2019 12:39:11
			See https://dbatools.io/Export-DbaScript for more information
		*/
		SET ANSI_NULLS ON
		SET QUOTED_IDENTIFIER ONk
		CREATE TABLE [dbo].[Users](
			[Id] [int] IDENTITY(1,1) NOT NULL,
			[AboutMe] [nvarchar](max) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
			[Age] [int] NULL,
			[CreationDate] [datetime] NOT NULL,
			[DisplayName] [nvarchar](40) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
			[DownVotes] [int] NOT NULL,
			[EmailHash] [nvarchar](40) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
			[LastAccessDate] [datetime] NOT NULL,
			[Location] [nvarchar](100) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
			[Reputation] [int] NOT NULL,
			[UpVotes] [int] NOT NULL,
			[Views] [int] NOT NULL,
			[WebsiteUrl] [nvarchar](200) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
			[AccountId] [int] NULL
		) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
	</code>
</pre>

Now I can take this and run it on another server.

I can also use the following, which will specify specific options for the export including adding the database context into the exported script.

<pre>  
	<code class="ps">
		PS C:\WINDOWS\system32&gt; $options = new-dbascriptingoption
		PS C:\WINDOWS\system32&gt; $options.scriptschema = $true
		PS C:\WINDOWS\system32&gt; $options.Includeheaders = $false
		PS C:\WINDOWS\system32&gt; $options.IncludeDatabaseContext = $true
		PS C:\WINDOWS\system32&gt; Get-Dbadbtable -sqlinstance localhost -database StackOverflow2010 -table dbo.users | export-dbascript -path C:\temp\tables\so-users.sql -ScriptingOptions $options
	</code>
</pre>

### Table Data

In some situations, I may need to get all of the reference data from one table and load it into another, however just like scripting out the table I need to load up SSMS, connect to the instance, find the database it is quite an involved task. Let&#8217;s see how we can do this with dbatools.

<pre>  
	<code class="ps">
		Get-DbadbTable -SqlInstance localhost -Database StackOverFlow2010 -Table dbo.users | Export-DBADBTableData -Path C:\temp\tables\so-users-data.sql -Append
	</code>
</pre>

![](/img/copy-table-data.png)

The script finished, and in C:\temp\tables there is a file called so-users-data.sql just like what I asked for, I can now take this script to another server run it and import all of the data into another table with the same schema._ _

I could even go one further and just copy the data directly from PowerShell, at the moment my destination table is empty, it resides on the same server as the source table inside the same database.

![](/img/copy-table-data-2.png)

If I execute the following line of code

<pre>   	
	<code class="ps">
		Get-dbadbtable -sqlinstance localhost -Database StackOverflow2010 -table dbo.users | copy-dbadbtabledata -DestinationTable dbo.users2
	</code>
</pre>

dbatools is going to go and get all of that data, and copy it into the new table, just like I asked

![](/img/copy-table-data-3.png)

But is the data actually there?

![](/img/copy-table-data-4.png)

Of course, it is!

### Copy Copy Copy

So I just got my new SQL Server from the infrastructure team but I want to move some jobs and alerts maybe even some database mail configuration from another server in the estate to this one. Of course, I could script out each object individually or, yep you guessed it, I could use dbatools to copy them from the existing instance to the new one.

Let&#8217;s see how I can do this with dbatools for various objects within the instance.

#### Agent Operator

Agent Operators are useful for notifications when jobs fail or when an alert is triggered, but sometimes I need to make sure the same operator exists on every server. With dbatools this is really easy.

<pre>   	
	<code class="ps">
		Copy-DbaAgentOperator -source localhost -destination localhost\sql2016 -Operator DBATeam
	</code>
</pre>

If I want to copy the operator to more than one server all I need to do is comma separate the servers passed to the -destination flag.

![](/img/copy-operator.png)

If the operator already exists on the destination but I would like to replace it with the one from source all I need to do is pass the -force flag which will drop the operator from the destination before copying it, which is demonstrated below.

![](/img/copy-operator-drop.png)

#### Agent Jobs

I have a situation quite often where people outside of the team I work will load up a SQL Agent Job onto one of the Availability Group (AG) members and fail to add it to one of the secondary members when the AG fails over the job doesn&#8217;t run as it doesn&#8217;t exist.

To fix this, I have to script the job out, take the T-SQL that the scripting function in SSMS creates and run it onto the server where the job is missing, it all takes a little bit of time, however with dbatools I can simplify this and do it with just one line

![](/img/copy-agent-job-ssms.png)

I want to copy the dbatools test job from **localhost\sql2016** to **localhost\sql2014**

<pre>   	
	<code class="ps">
		Copy-DBAAgentJob -source localhost\sql2016 -destination localhost\sql2014 -Job "dbatools test job"
	</code>
</pre>

![](/img/copy-agent-job.png)

The job has been successfully copied from **localhost\sql2016** to **localhost\sql2014**

![](/img/copy-agent-job-ssms-2.png)

Now then, let&#8217;s say that your job has a notification setup, when it fails I want an operator to be notified, what if that operator doesn&#8217;t exist on the destination server? Well, let&#8217;s find out.

![](/img/copy-agent-job-2.png)

As you can see dbatools will not copy the job as it has a dependency, pretty neat huh.

#### Database Mail

In a lot of cases, the database mail profiles and accounts that are in use across the estate I manage are the same, I need the same database profiles & accounts on every server I manage but setting them up takes time. We can simplify this with dbatools by copying all of the profiles & accounts from an existing server to a server with no database mail.

In this configuration I have purposely configured one of our SQL Server instances to have no database mail configuration, as shown below

![](/img/dba-mail-ssms.png)

I want to copy all of the database mail configurations from an existing server in the estate to this instance. Word of warning though, by default, copy-dbadbmail will copy all database mail accounts and profiles.

<pre>   	
	<code class="ps">
		Copy-DbaDbMail -source localhost\sql2016 -destination localhost\sql2014<
	</code>
</pre>

![](/img/dba-mail-ssms-2.png)

Now if I go back to SSMS I will see that all the database mail configuration has been copied

![](/img/dba-mail-ssms-3.png)

#### Logins

Sometimes I need to copy all or some of the logins from one server to another, you can do that with dbatools

<pre>   	
	<code class="ps">
		Export-dbalogin -sqlinstance localhost -path c:\temp\cred.sql
	</code>
</pre>

![](/img/copy-logins.png)

If I hope into C:\temp\cred.sql I will see that the SID is even copied which is great for Availability Group situations where I need the same SQL Account with the same SID spread over multiple availability group members.

However, let&#8217;s say I want to export just one login from the server this can be done with the following line of code

<pre>   	
	<code class="ps">
		Export-dbalogin -sqlinstance localhost -login login1 -path c:\temp\cred.sql
	</code>
</pre>

![](/img/copy-logins-single.png)

What about multiple accounts? They can be passed to -login with a comma, separating each login as shown below.

<pre>   	
	<code class="ps">
		Export-dbalogin -sqlinstance localhost -login login1, login2, login3 -path c:\temp\cred.sql
	</code>
</pre>

It is also worth noting, if you don&#8217;t want your logins to be overwritten each time you execute this command, specify the -append flag and dbatools will pop the logins onto the end of the existing file, but if the login already exists in the file dbatools will not tell you about it, it will just keep adding.

#### Databases

This one is really cool! So I have a database on **servera** (localhost\sql2014) that I need to copy to **serverb** (localhost\sql2016) maybe for some testing or some other scenario.

As you can see bwlow the database exists on **servera** but not on **serverb**

![](/img/copy-database-ssms.png)

To copy if we need to execute one line of code

<pre>   	
	<code class="ps">
		Copy-dbadatabase -source localhost\sql2014 -destination localhost\2016 -database MigrationTest -BackupRestore -SharedPath C:\temp\migration
	</code>
</pre>

![](/img/copy-database-ps-1.png)

What has happened behind the scenes here is that dbatools has taken a copy only backup of the **MigrationTest** database and popped it into C:\temp\migration it has then taken that backup and restored it to the specified destination which in this case is serverb, I am able to see that it was successful by looking directly inside SSMS

![](/img/copy-database-ssms-2.png)

Once dbatools was happy that the restore had completed successfully, the backup that it initially took is removed unless you explicitly tell it not to delete it by passing the -NoBackupCleanUp flag.

_In a production environment the -SharedPath will need to be a valid UNC path accessible by both instances._

_**Note:**_ You can only copy a database from a SQL Server instance which has a version less than or equal to the version of the SQL Server you are restoring to. You can&#8217;t copy a database from a SQL Server with a version greater than the version of the SQL Server you are restoring to.

![](/img/copy-database-no-sorry.png)

### The Wrap Up

I really think that dbatools is going to find a home in my day to day toolkit, once I learn a little bit more about how various part of it work and how it can solve some of the common problems I am faced with on a daily basis I will be sure to get it onto my work laptop.