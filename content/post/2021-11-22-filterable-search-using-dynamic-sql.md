---
title: Creating a filterable search using dynamic sql
date: 2021-11-22T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/filterable-search-using-dynamic-sql"
categories:
- sqlserver
tags:
- SQL
- T-SQL
---

There was a situation recently where the developers had built an application in C# but wanted some help to implement a search using SQL server and a stored procedure which would return different results depening on the results passed in. In this post we are going to look at how I went about that.

<!--more-->

**Before we start, word of warning, Dynamic SQL is useful in some cases but it is always worth being mindful that it is open to SQL injection attacks if incorrectly implemented.**

This is what the completed stored procedure looks like, but that isn't helpful to demonstrate how it works, so I am going to break it down. 

```
CREATE PROCEDURE dbo.p_StudentSearch

@year INT,
@houseID INT,
@classID INT,
@studentID INT

AS

BEGIN

	SET NOCOUNT ON;

	DECLARE 
		@sqlQuery NVARCHAR(MAX),
		@yearFilter NVARCHAR(MAX),
		@houseFilter NVARCHAR(MAX),
		@classFilter NVARCHAR(MAX),
		@studentFilter NVARCHAR(MAX)

	SET @yearFilter = ' AND c.[Year] = ' + CONVERT(varchar,@year)
	SET @houseFilter = ' AND s.[HouseID] = ' + CONVERT(varchar,@houseID)
	SET @classFilter = ' AND c.ID = ' + CONVERT(varchar,@classID)
	SET @studentFilter = ' AND s.[ID] = ' + CONVERT(varchar,@studentID)

	SET @sqlQuery = 'SELECT 

		s.[ID] as [StudentID],
		c.ID as [ClassID],
		s.[HouseID],
		s.[Name],
		h.HouseName,
		c.ClassName,
		c.[Year]

	FROM
		[dbo].[Students] s

	LEFT OUTER JOIN [dbo].[StudentClasses] sc
		ON s.ID = sc.StudentId

	INNER JOIN [dbo].[Classes] c
		ON sc.ClassID = c.ID

	INNER JOIN [dbo].[Houses] h 
		ON s.HouseID = h.ID

	WHERE 1=1'

	IF @year != 0

	SET @sqlQuery = @sqlQuery + @yearFilter

	IF @houseID != 0

	SET @sqlQuery = @sqlQuery + @houseFilter

	IF @classID != 0

	SET @sqlQuery = @sqlQuery + @classFilter

	IF @studentID != 0

	SET @sqlQuery = @sqlQuery + @studentFilter

	BEGIN
		BEGIN TRY

		    EXEC (@sqlQuery)

		END TRY

		BEGIN CATCH
			Print ERROR_MESSAGE()
		END CATCH
	END
END
```
At the top we have our variables that are passed in from the application tier, these are in our case Integar values 

```
CREATE PROCEDURE dbo.p_StudentSearch

@year INT,
@houseID INT,
@classID INT,
@studentID INT
```

What we are doing next is declaring the variables that are going to be used to hold the conditions that we will add to our where clause, these are NVARCHAR(MAX) so we can ensure we don't run into any truncation. 

@sqlQuery will hold the completed query which we will programatically build. As the values are Integars we do need to convert them to varchar to get them to concatenate. 

```
DECLARE 
    @sqlQuery NVARCHAR(MAX),
    @yearFilter NVARCHAR(MAX),
    @houseFilter NVARCHAR(MAX),
    @classFilter NVARCHAR(MAX),
    @studentFilter NVARCHAR(MAX)

	SET @yearFilter = ' AND c.[Year] = ' + CONVERT(varchar,@year)
	SET @houseFilter = ' AND s.[HouseID] = ' + CONVERT(varchar,@houseID)
	SET @classFilter = ' AND c.ID = ' + CONVERT(varchar,@classID)
	SET @studentFilter = ' AND s.[ID] = ' + CONVERT(varchar,@studentID)
```

Next up is the query, I wrote this outside of the dynamic SQL first to make sure that it works as I intended and returned ever row, once I was happy I wrapped it in single quotes and SET it into the @sqlQuery variable that we declared earlier. 

```
SET @sqlQuery = 'SELECT 

    s.[ID] as [StudentID],
    c.ID as [ClassID],
    s.[HouseID],
    s.[Name],
    h.HouseName,
    c.ClassName,
    c.[Year]

FROM
    [dbo].[Students] s

LEFT OUTER JOIN [dbo].[StudentClasses] sc
    ON s.ID = sc.StudentId

INNER JOIN [dbo].[Classes] c
    ON sc.ClassID = c.ID

INNER JOIN [dbo].[Houses] h 
    ON s.HouseID = h.ID
```

At the end of the query you will see 1=1 as shown below, this offers no functional value to the query other than allowing us to simply add on every filter condition after it without having to worry if the conditions need to have AND keywords at the start. 

```
WHERE 1=1'
```
Next what I am doing is making sure that none of the incoming values are 0, this would mean that no option has been selected in my application tier, you could change this to whatever you are using to highlight no option selected. 

If these values are not set to 0 what I want to do is set the filter for that variable, which we set at the top of the procedure to the end of the @sqlQuery. 

What this will do is if any of these values are set to 0 meaning not selected the condition will not be added to the where clause. 

```
IF @year != 0

	SET @sqlQuery = @sqlQuery + @yearFilter

	IF @houseID != 0

	SET @sqlQuery = @sqlQuery + @houseFilter

	IF @classID != 0

	SET @sqlQuery = @sqlQuery + @classFilter

	IF @studentID != 0

	SET @sqlQuery = @sqlQuery + @studentFilter
```
Finally, now that we have the query built and ready to go, we are going to run it, I have wrapped it inside a TRY so that if it errors I can pass that error back to the application tier but in this example the error will simply be printed. 

```
BEGIN
    BEGIN TRY

        EXEC (@sqlQuery)

    END TRY

    BEGIN CATCH
        Print ERROR_MESSAGE()
    END CATCH
END
```
Now that everything is in place, lets run the code and see what happens, in this example I am just getting all rows where the class is year 1

```
EXEC dbo.p_StudentSearch @year = 1, @houseID = 0, @classID = 0, @studentID = 0
```
![](/img/filterable-search-1.png)


Now where the class is a year 1 class and the houseID is also 1

```
EXEC dbo.p_StudentSearch @year = 1, @houseID = 1, @classID = 0, @studentID = 0
```
![](/img/filterable-search-2.png)

Finally, where the year is 1, houseID is 1 and the Student is Harry Potter who has StudentID of 1

```
EXEC dbo.p_StudentSearch @year = 1, @houseID = 1, @classID = 0, @studentID = 1
```
![](/img/filterable-search-3.png)

The filter works as expected, different results are returned depending on the values which we are passing in, this is what the query looks like when the stored procedure is called, classID isn't included as it was set to 0 when it was called. 

```
SELECT 

		s.[ID] as [StudentID],
		c.ID as [ClassID],
		s.[HouseID],
		s.[Name],
		h.HouseName,
		c.ClassName,
		c.[Year]

	FROM
		[dbo].[Students] s

	LEFT OUTER JOIN [dbo].[StudentClasses] sc
		ON s.ID = sc.StudentId

	INNER JOIN [dbo].[Classes] c
		ON sc.ClassID = c.ID

	INNER JOIN [dbo].[Houses] h 
		ON s.HouseID = h.ID

	WHERE 1=1 AND c.[Year] = 1 AND s.[HouseID] = 1 AND s.[ID] = 1
```