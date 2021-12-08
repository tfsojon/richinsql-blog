---
title: "Formula One Database"
date: 2021-08-27T16:39:56+01:00
draft: false
---

{{< rawhtml >}}
<center><iframe src="https://ghbtns.com/github-btn.html?user=richinsql&repo=FormulaOneDatabase&type=star&count=true&size=large" frameborder="0" scrolling="0" width="170" height="30" title="GitHub"></iframe>

<br />

<a style="text-align: center;" href="https://github.com/RichInSQL/FormulaOneDatabase" target="_blank">View On Github</a>

<bt />
</center>
{{< /rawhtml >}}

Do you have an interest in Formula One? Do you need a database for T-SQL demonstrations or testing of your T-SQL code that is open source and not the adventure works database? I might just have the answer, I have put together using freely available data a database for just that.

The un-official Formula One Database.

### What Is Included 

* Drivers 
  - Nationality
  - Races Entered
  - Races Started
  - Pole Positions
  - Race Wins
  - Podiums
  - Fastest Laps
  - Total Points 
* Driver Seasons
  - DriverID **FK**
  - SeasonRefID **FK**
* Driver Championships
  - DriverID **FK**
  - SeasonID **FK**
* Driver Standings
  - DriverID **FK**
  - SeasonRefID **FK**
  - Points
* Constructors
  - Country Licensed
  - Races Entered
  - Races Started
  - Drivers
  - Total Entries
  - Wins
  - Points
  - Poles
  - Fastest Laps
  - Podiums
  - Constructor Championships Won
  - Driver Championships Won
* Constructor Seasons
  - ConstructorID **FK**
  - SeasonRefID **FK**
* Constructor Nationality
  - ConstructorID **FK**
  - CountryID **FK**
* Constructor Standings 
  - ConstructorID **FK**
  - SeasonRefID **FK**
  - Points
* Circuits
   - Circuit
   - Grand Prix
   - CircuitID
   - TypeRefID **FK**
   - DirectionRefID **FK**
   - LastLengthUsed
   - GrandPrixHeld
* Circuits Location
  - CircuitID
  - CountryID **FK**
  - Longitude
  - Latitude
* Circuit Images
  - CircuitID **FK**
  - ImageURL   
* Circuit Seasons
  - CircuitID **FK**
  - SeasonRefID
* Circuit Type
  - CircuitID **FK**
  - TypeRefID **FK**    
* Circuit Directions
  - CircuitID **FK**
  - DirectionRefID **FK**
* Seasons
  **Reference table for seasons used in parent tables**
  - SeasonID
  - SeasonRefID **FK**
* Country
  **Reference table for countries used in parent tables**
  - CountryID
  - CountryRefID **FK**

### Found An Issue?

If you have found an issue with the data, something isn't quite right or I have made a mistake during the import, feel free to [open an Issue](https://github.com/RichInSQL/FormulaOneDatabase/issues) on the [GitHub repo](https://github.com/RichInSQL/FormulaOneDatabase). 

If you would like to fix the issue yourself, I would also encourage you to create a pull request, the repo is public and a pull request would really help take this project forward. 

### Pull Requests 

If you would like to make a suggested change to the database, be that the schema or the data itself you can submit a pull request which I can then review and merge into the Main branch once tested. 

### This Sounds Cool, Where Can I Get It?

If you would like to create the database on your own instance of SQL Server you can download the installtion script from the [GitHub repo](https://github.com/RichInSQL/FormulaOneDatabase) and run it on your SQL Server Instance. 


