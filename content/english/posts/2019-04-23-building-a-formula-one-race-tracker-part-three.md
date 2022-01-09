---
title: Building a Formula One Race Tracker – Part Three
date: 2019-04-23T16:00:04+01:00
author: Rich
layout: post
permalink: /building-a-formula-one-race-tracker-part-three
categories:
  - sqlserver
  - csharp
tags:
  - application
  - SQL
  - sqlserver
  - t-sql
---
**Contents**

  1. <a href="/posts/2019-04-09-building-a-formula-one-race-tracker-part-one" target="_blank" rel="noopener noreferrer">Part One – Project Overview</a>
  2. <a href="/posts/2019-04-16-building-a-formula-one-race-tracker-part-two" target="_blank" rel="noopener noreferrer">Part Two – Database Schema</a>
  3. <a href="/posts/2019-04-23-building-a-formula-one-race-tracker-part-three" target="_blank" rel="noopener noreferrer">Part Three – Stored Procedures</a>

Stored Procedures are going to power the flow of data in and out of the Formula One Race Tracker, I will make use of a stored procedure for sending my data into the application and I am also going to make use of a stored procedure for getting the data back out again.

### The Insert Procedure

In [Part Two](/posts/2019-04-16-building-a-formula-one-race-tracker-part-two) I showed how I built the database schema for the race tracker, I now know which tables relate to which and what columns I have available, now I need to produce a stored procedure that will allow the web application to insert data into the database.

```
        -- =============================================
        -- Author:      Bonza Owl
        -- Create date: 5.4.18
        -- Description: Inserts race data into database
        -- =============================================
        CREATE PROCEDURE [dbo].[Insert_Race_Data]

        @Race_Date DATETIME,
        @Driver_ID INT,
        @Circuit_ID INT,    
        @Final_Position TINYINT,
        @Points INT,
        @Race_Type TINYINT,
        @State INT = 0 OUTPUT

        AS
        BEGIN
            SET NOCOUNT ON;
            SET XACT_ABORT,
                QUOTED_IDENTIFIER,
                ANSI_NULLS,
                ANSI_PADDING,
                ANSI_WARNINGS,
                ARITHABORT,
                CONCAT_NULL_YIELDS_NULL ON;
            SET NUMERIC_ROUNDABORT OFF;

            DECLARE @localTran bit
            IF @@TRANCOUNT = 0
            BEGIN
                SET @localTran = 1
                BEGIN TRANSACTION LocalTran 
            END

            BEGIN TRY       		

                INSERT INTO Race 
                (
                Race_Date,
                Driver_ID,		
                Circuit_ID,    
                Final_Position,
                Race_Type,
            Points
                )
                VALUES
                (
                    @Race_Date,
                    @Driver_ID,			
                    @Circuit_ID,
                    @Final_Position,
                    @Race_Type,
                @Points
                );

                COMMIT TRANSACTION LocalTran;

                SET @State = 1;

            END TRY
            BEGIN CATCH        
        
                IF @localTran = 1 AND XACT_STATE() &lt;&gt; 0
                    ROLLBACK TRAN LocalTran
        
            END CATCH

        END
  ```

To explain how the above stored procedure works I will break it down.

```
CREATE PROCEDURE [dbo].[Insert_Race_Data]

@Race_Date DATETIME,
@Driver_ID INT,
@Circuit_ID INT,    
@Final_Position TINYINT,
@Points INT,
@Race_Type TINYINT,
@State INT = 0 OUTPUT

AS
```

At the top, there are 7 parameters above the **AS,** these are parameters that the stored procedure will expect to be presented when it is called if they are not all presented it will throw an error

  1. Race_Date &#8211; The date of the race in which results are being recorded.
  2. Driver_ID &#8211; The ID of the driver whose results are being recorded.
  3. Circuit_ID &#8211; The ID of the circuit of which results are being recorded.
  4. Final_Position &#8211; The finishing position of the driver whose results are being recorded.
  5. Points &#8211; The total points collected for the position in which the driver finished
  6. Race_Type &#8211; The ID of the Race Type
  7. State &#8211; If the insert was successful this will be 1 if it wasn't it will be 0, State is passed back to the application to let the user know if the procedure completed successfully.

```
        DECLARE @localTran bit
        IF @@TRANCOUNT = 0
        BEGIN
            SET @localTran = 1
            BEGIN TRANSACTION LocalTran 
        END
  ```

In the above code block, I have declared a variable called @localtran as a BIT datatype, I have then asked SQL Server inside an IF statement that if @localtran = 0 please set @localtran to 1 and Begin a new transaction called LocalTran, if however the stored procedure is called and @localtran is not 0 the stored procedure will fail as a transaction is already open.

```
        BEGIN TRY       		

                INSERT INTO Race 
                (
                Race_Date,
                Driver_ID,		
                Circuit_ID,    
                Final_Position,
                Race_Type,
            Points
                )
                VALUES
                (
                    @Race_Date,
                    @Driver_ID,			
                    @Circuit_ID,
                    @Final_Position,
                    @Race_Type,
                @Points
                );

                COMMIT TRANSACTION LocalTran;
            SET @State = 1;
  ```

The above code block is going to first tell SQL Server that I am now going to try and insert the data which was passed in using the variables at the top of the procedure into the Race table, if successful the transaction will be committed and @State will be set to 1, if however, it was not successful, the TRY will fall through to the CATCH and @State will be left as 0

```
        END TRY
            BEGIN CATCH
        
                IF @localTran = 1 AND XACT_STATE() &lt;&gt; 0
                    ROLLBACK TRAN LocalTran
        
            END CATCH
  ```

The above code block is where we tell SQL Server what we want to do in the event of an error being thrown inside the TRY.

I have specified another IF and said to SQL Server &#8211; If @localtran is 1 and XACT_STATE is not 0 please roll back the transaction that I just tried as something went wrong and I don't want that data in the database.

The catch is then ended and the stored procedure will pass @State back to the application, set as 0 which will show the user that something went wrong.

### Getting Data Out

As I showed in <a href="https://www.codenameowl.com/building-a-formula-one-race-tracker-part-one" target="_blank" rel="noopener noreferrer">Part One</a> the web application will have a number of drop-down boxes to allow for easy selection of drivers & circuits, as I have a relational database I need to ensure that the correct ID for the driver & circuit are passed to the database when I am recording results, to do this I am going to pre-populate the dropdown boxes with the information I require, to do this I need a couple of stored procedures which will return the data for me.

Each of the stored procedures for returning data will be built up using the following framework.

```
        - =============================================
        -- Author:      Bonza Owl
        -- Create date: 5.4.18
        -- Description: Returns all race types
        -- =============================================

        CREATE PROCEDURE [dbo].[Get_Race_Types]

        AS
        BEGIN
            SET NOCOUNT ON;
            SET XACT_ABORT,
                QUOTED_IDENTIFIER,
                ANSI_NULLS,
                ANSI_PADDING,
                ANSI_WARNINGS,
                ARITHABORT,
                CONCAT_NULL_YIELDS_NULL ON;
            SET NUMERIC_ROUNDABORT OFF;	    
            
            CODE HERE

        END
  ```

At the top there is a comments block, this is where I usually like to put my name, the date the procedure was first introduced and then a short description about its use, it is useful if I return to this project in 12 months to have an understanding of what each procedure does.

Next is the create procedure line, this is where I am going to tell SQL Server what I would like to call this procedure and the schema that I would like it to belong to.

Now the main part of the procedure, the procedure always stars with a BEGIN after that I need to tell SQL Server what settings need to be enabled during the execution of the procedure;

  * <a href="https://docs.microsoft.com/en-us/sql/t-sql/statements/set-nocount-transact-sql" target="_blank" rel="noopener noreferrer">NOCOUNT</a> &#8211; When this is set to on SQL Server will not return the number of affected rows.
  * <a href="https://docs.microsoft.com/en-us/sql/t-sql/statements/set-xact-abort-transact-sql" target="_blank" rel="noopener noreferrer">XACT_ABORT</a> &#8211; When set to on this will inform SQL Server that the current transactional process should be rolled back in the event of a run time error.
  * <a href="https://docs.microsoft.com/en-us/sql/t-sql/statements/set-ansi-nulls-transact-sql" target="_blank" rel="noopener noreferrer">ANSI_NULLS</a> &#8211; Specifies ISO compliant behavior of the Equals (=) and Not Equal To (<>) comparison operators when they are used with null values in SQL Server 2017
  * [ANSI_PADDING](https://docs.microsoft.com/en-us/sql/t-sql/statements/set-ansi-padding-transact-sql) &#8211; Controls the way the column stores values shorter than the defined size of the column, and the way the column stores values that have trailing blanks in **char**, **varchar**, **binary**, and **varbinary** data
  * [ANSI_WARNINGS](https://docs.microsoft.com/en-us/sql/t-sql/statements/set-ansi-warnings-transact-sql) &#8211; When set to ON, if null values appear in aggregate functions, such as SUM, AVG, MAX, MIN, STDEV, STDEVP, VAR, VARP, or COUNT, a warning message is generated. When set to OFF, no warning is issued.
  * <a href="https://docs.microsoft.com/en-us/sql/t-sql/statements/set-arithabort-transact-sql" target="_blank" rel="noopener noreferrer">ARITHABORT</a> &#8211; Ends a query when an overflow or divide-by-zero error occurs during query execution.
  * [CONCAT\_NULL\_YIELDS_NULL](https://docs.microsoft.com/en-us/sql/t-sql/statements/set-concat-null-yields-null-transact-sql) &#8211; Controls whether concatenation results are treated as null or empty string values
  * <a href="https://docs.microsoft.com/en-us/sql/t-sql/statements/set-numeric-roundabort-transact-sql" target="_blank" rel="noopener noreferrer">NUMERIC_ROUNDABORT</a> &#8211; Specifies the level of error reporting generated when rounding in an expression causes a loss of precision. This must be OFF when you're creating or changing indexes on computed columns or indexed views.

The procedure is then finalized with an END which matches the BEGIN from the top of the procedure, which tells SQL Server that this transaction is now over.

#### Get Race Types

Get race types is the stored procedure that returns all of the race types to the drop down box on the web application.

This is just a simple select with returning all values in the Race_Types table.

```
        - =============================================
        -- Author:      Bonza Owl
        -- Create date: 5.4.18
        -- Description: Returns all race types
        -- =============================================

        CREATE PROCEDURE [dbo].[Get_Race_Types]

        AS
        BEGIN
            SET NOCOUNT ON;
            SET XACT_ABORT,
                QUOTED_IDENTIFIER,
                ANSI_NULLS,
                ANSI_PADDING,
                ANSI_WARNINGS,
                ARITHABORT,
                CONCAT_NULL_YIELDS_NULL ON;
            SET NUMERIC_ROUNDABORT OFF;	    
            
            SELECT 
                R.Race_Type_ID,
                R.Race_Type
            FROM
            Race_Types R

        END
  ```

![](/img/Race_Types_Table_Output.png)

#### Get Circuits

Exactly the same as Race\_Types, Get\_Circuits will just return a list of current circuits which is used to populate the drop-down list on the web application.

In the select, I have returned both the name of the Circuit and the Circuit ID as the Circuit ID is what I need to send back to the database on submission of the web form, but we will come to that in more detail in a later section.

```
        -- =============================================
        -- Author:      Bonza Owl
        -- Create date: 5.4.18
        -- Description: Returns all circuits
        -- =============================================

        CREATE PROCEDURE [dbo].[Get_Circuits]

        AS
        BEGIN
            SET NOCOUNT ON;
            SET XACT_ABORT,
                QUOTED_IDENTIFIER,
                ANSI_NULLS,
                ANSI_PADDING,
                ANSI_WARNINGS,
                ARITHABORT,
                CONCAT_NULL_YIELDS_NULL ON;
            SET NUMERIC_ROUNDABORT OFF;	 
            
            SELECT 

                C.Circuit_ID,
                C.Grands_Prix_Name as Circuit_Name
                
            FROM
                Circuit C

            ORDER BY 
                Circuit_Name
        END
  ```

![](/img/Circuit_Table_Output.png)

#### Get Drivers

Get_Drivers is slightly different, it only returns drivers that have not retired because I don't want to record times against drivers who are no longer racing, it would be pointless. The driver name is concatenated together and returned as DriverName so the user of the web application knows who it is they are selecting when completing the form.

```
        -- =============================================
        -- Author:      Bonza Owl
        -- Create date: 5.4.18
        -- Description: Returns all drivers
        -- =============================================

        CREATE PROCEDURE [dbo].[Get_Drivers]

        AS
        BEGIN
            SET NOCOUNT ON;
            SET XACT_ABORT,
                QUOTED_IDENTIFIER,
                ANSI_NULLS,
                ANSI_PADDING,
                ANSI_WARNINGS,
                ARITHABORT,
                CONCAT_NULL_YIELDS_NULL ON;
            SET NUMERIC_ROUNDABORT OFF;	 
            
            SELECT 
                D.Driver_ID,
                D.Forename + ' ' + D.Surname as DriverName
            FROM
                Drivers D
            WHERE 
                D.Retired = 0
            ORDER BY
                DriverName
        END
  ```

![](/img/Driver_Table_Output.png)

#### Get Results

Get_Results will power the Grid View that will show us the current runnings of the race season.

```
        -- =============================================
        -- Author:      Bonza Owl
        -- Create date: 5.4.18
        -- Description: Returns all results
        -- =============================================

        CREATE PROCEDURE [dbo].[Get_Results]

        AS
        BEGIN
            SET NOCOUNT ON;
            SET XACT_ABORT,
                QUOTED_IDENTIFIER,
                ANSI_NULLS,
                ANSI_PADDING,
                ANSI_WARNINGS,
                ARITHABORT,
                CONCAT_NULL_YIELDS_NULL ON;
            SET NUMERIC_ROUNDABORT OFF;	 

            SELECT DISTINCT
                D.Forename + ' ' + D.Surname as DriverName,
                T.Team_Name,
                SUM(R.Points) as Points

            FROM Race R

            INNER JOIN Driver_Team DT ON 
                DT.Driver_ID = R.Driver_ID
                AND End_Date IS NULL --We only want current drivers for this team

            INNER JOIN Teams T ON 
                T.Team_ID = DT.Team_ID 
                AND T.Active = 1 --We only want active teams

            INNER JOIN Drivers D ON 
                D.Driver_ID = DT.Driver_ID
                AND D.Retired = 0 --We don't want drivers that have retired

            WHERE 
                YEAR(Race_Date) = YEAR(GETDATE()) --We just want this season

            GROUP BY 
                D.Forename,
                D.Surname,
                T.Team_Name
        END
  ```

It is slightly larger than the other get procedures so I will break it down.

```
        SELECT DISTINCT
            D.Forename + ' ' + D.Surname as DriverName,
            T.Team_Name,
            SUM(R.Points) as Points
  ```

The select statement requests the following data

  * Forename & Surname from the Drivers table and joins them together, to give the drivers name as one field, this is required for displaying in the web application.
  * Team Name from the team's table
  * Points from the race table, this produces a total sum of all values however, I will need to specify some additional parameters in the were clause and group by for this to work as I want.

```
        INNER JOIN Driver_Team DT ON 
            DT.Driver_ID = R.Driver_ID
            AND DT.End_Date IS NULL --We only want current drivers for this team
  ```

To ensure that I get the data returned correctly, I need to join the driver\_team table, to do this I am joining on the driverID in both the driver\_team table and Race table additionally I have specified that I only want results returned where End\_date on the Driver\_Team table is NULL so the driver is currently driving for the team returned.

```
        INNER JOIN Teams T ON 
            T.Team_ID = DT.Team_ID 
            AND T.Active = 1 --We only want active teams
  ```

Now that the Driver\_Team table is joined to the race table the team can be obtained and returned, this is done by joining the TeamID to the TeamID in the Driver\_Team table, however, I have specified that I only want active teams.

```
        INNER JOIN Drivers D ON 
            D.Driver_ID = DT.Driver_ID
            AND D.Retired = 0 --We don't want drivers that have retired
  ```

Finally, the driver's table is joined to the driver_team table to get the drivers name and current state, I have specified in the join that I only want drivers, where retired is 0, this will ensure any previous drivers from previous seasons that have retired are not returned.

```
        WHERE YEAR(Race_Date) = YEAR(GETDATE()) --We just want this season

        GROUP BY 
            D.Forename,
            D.Surname,
            T.Team_Name
  ```

In the where clause, I have specified that I only want this year's data, this is done by using the YEAR function on the DateTime column Race_Date and matching it to the Year of the GETDATE() function which will return the current date in DateTime format.

Finally, the data is then grouped, first by driver forename, then driver surname and finally Team_Name to give me the total points for the current season per driver.

This should return something like this;

![](/img/Race_Results_Output.png)