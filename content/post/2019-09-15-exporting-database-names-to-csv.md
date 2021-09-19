---
title: Exporting Database Names To CSV
date: 2019-09-15T11:30:45+01:00
author: Rich
layout: post
permalink: /exporting-database-names-to-csv
categories:
  - powershell
tags:
  - PowerShell
  - Server
  - SQL

---

I needed a way of getting a complete list of databases over multiple instances in SQL Server into something that I was able to manipulate, of course I could load up SSMS and run a query against the MSDB database to return me the name of every database on that instance, but that is slow, I have to connect to each instance I want to query an individually run the code;

<pre> 
	<code class="sql">
		SELECT name FROM sys.databases
	</code>
</pre>

But that takes too long, especially when you have lots of instances to cycle through. Instead, I decided to use PowerShell and the DbaTools module. With one line of code I was able to get the names of every database out to a CSV file which I could then manipulate. 

<pre> 
	<code class="powershell">
		$instance = 'localhost','localhost\SQL2016'
		$instance | Get-DbaDatabase -ExcludeSystem | Sort-Object | Select-Object InstanceName,Name | Export-Csv -Path C:\Temp\database.csv 
	</code>
</pre>

But what does all of this mean?

<pre> 
	<code class="powershell">
		$instance = 'localhost','localhost\SQL2016'
	</code>
</pre>

So here, I am specifying which instances I would like to query with the command

<pre> 
	<code class="powershell">		
		$instance | Get-DbaDatabase -ExcludeSystem
	</code>
</pre>

I am then passing that into the Get-DbaDatabase command and telling it with a switch that I want to exclude any system databases, we will assume that every instance has the standard system databases and I don't need to know about them for every instance I query.

<pre> 
	<code class="powershell">
		| Sort-Object |
	</code>
</pre>

The next part is telling PowerShell that I would like the results ordered, this seems to order the results by database name A-Z.

<pre> 
	<code class="powershell">
		Select-Object InstanceName,Name
	</code>
</pre>

Now we need to tell PowerShell to only return the information we are interested in, which in my case is the InstanceName & Name which is the database name as it appears in SSMS

<pre> 
	<code class="powershell">
		Export-Csv -Path C:\Temp\database.csv 
	</code>
</pre>

Finally, I would like the results to be exported to a CSV file and stored on my local C:\ 

Pretty neat, this all took 5.1s to run which is far less time that it would have taken me to load up each instance, run the T-SQL query and export the results to CSV.




