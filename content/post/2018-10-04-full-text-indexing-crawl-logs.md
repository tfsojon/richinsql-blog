---
title: Full Text Indexing Crawl Logs
date: 2018-10-04T19:26:27+01:00
author: Rich
layout: post
permalink: /full-text-indexing-crawl-logs
image: /wp-content/uploads/2018/10/Full-Text-Indexing-Crawl-Logs-1200x280.png
categories:
  - sqlserver
tags:
  - fulltext
  - index
  - SQL
  - sqlserver
---

Recently I ran into an issue I had never experienced before, a third party vendor was using Full-Text Indexing on one of the databases in their application, as part of their nightly routine they would do an incremental rebuild of the full-text index and then once a week a full rebuild of that index would take place, this wasn&#8217;t causing any SQL related problems but I found one day that the C:\ on that instance had become rather full and had triggered the disk alarm on our monitoring application.

Upon investigation, I found that there was a <span style="text-decoration: underline;"><strong>17GB</strong></span> crawl log on disk! Due to the instance not being restarted in some time the index rebuild had been appending the same log over and over.

I needed to find a way to shrink that log or get SQL to make a new one each time the rebuild job runs, therein lies the problem, for some reason Microsoft has little to no documentation on the crawl logs themselves so I set about trying to find articles written by other people who may have been in a similar situation previously, unfortunately, I found very very few, so I started testing&#8230;

### Anatomy Of The Crawl Log

When an error occurs during a crawl, the Full-Text crawl logging facility creates and _maintains\*\* \*\*\_a crawl log, for me this also included informational events.

Each crawl log corresponds to a specific Catalog in a specific database.

The crawl log file uses the following naming schema when creating the crawl log files.

SQLFT<DatabaseID><FullTextCatalogID>.LOG[<n>]

**<DatabaseID>** &#8211; The ID of the database, this is a 5 digit number with leading zeros, you can find the dbid in msdb.sys.databases

**<FullTextCatalogID>** &#8211; This is the Id of the catalog, this is also a 5 digit number with leading zeros.

**<n>** &#8211; In an incremental integer value that is appended to the crawl log in the event that more than one log exists for the same database & catalog.

So for example

_**SQLFT0001000001.LOG.4**_ is the crawl log for the database with  ID 10 and the catalog id 1, the 4 at the end of the file indicates that more than one log file exists for this catalog.

### Find The Catalog

So, first thing is first, let&#8217;s find the catalog that we want to rebuild the crawl log for.

We know which database and catalog is causing the problem because the file on disk has got out of hand and using the information provided in the anatomy section of this post shows us which catalog we need to refresh the log for.

Now we need to use this query to get the name of the catalog in the affected database.

**_ID & Names are unique to the database context, so make sure your in the correct database._**

```
    SELECT * FROM sys.fulltext_catalogs
```

Once you have run this, make a note of the catalog name, we will need that in a moment.

### Refresh The Catalog

Now that we know the name of the catalog in question we can use the below stored procedure to refresh the crawl log for that catalog.

This uses an undocumented Microsoft shipped stored procedure that will refresh the crawl log. To my knowledge, there is no other way at this point to refresh the log, truncate it or automate it&#8217;s recycling other than to use this stored procedure.

```
    EXEC sp_fulltext_recycle_crawl_log @ftcat = 'CatalogName'
```

Once executed, a new log file will be created in your log directory. After a period of time, mostly defined by you the old log files can be removed from disk.

### That&#8217;s all folks

The crawl log now gets recycled once a day before the index rebuild takes place, we worked alongside the third-party vendor to implement sp_fulltext_recycle_crawl_log into their existing SQL Agent Job, this has made the crawl log much more manageable.

### Some Notable Resources

Here are some of the resources I used to understand these logs and come up with a workable solution.

- <https://docs.microsoft.com/en-us/sql/relational-databases/search/populate-full-text-indexes?view=sql-server-2017>
- <https://thesqldude.com/tag/crawl-log/>
- <https://www.sqlrx.com/march-2016-tip-of-the-month-full-text-indexing-clean-up-those-log-files/>
