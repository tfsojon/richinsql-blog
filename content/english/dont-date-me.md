---
title: "Dont Date Me"
date: 2021-10-10T09:54:08+01:00
draft: false
---

## A collection of dates

Remembering the syntax for every date senario is hard, sometimes I forget and end up googling to find the answers, I have put this page together as a reference which might not only help me but someone else too. 

### Today

```
SELECT GETDATE() 
```

### Tomorrow

```
SELECT DATEADD(dd,1,GETDATE())
```

### Yesterday

```
SELECT DATEADD(dd,-1,GETDATE())
```

### Seven Days Prior To Today

```
SELECT DATEADD(dd,-7,GETDATE())
```

### Seven Days After Today

```
SELECT DATEADD(dd,7,GETDATE())
```

### Start Of This Week

```
SELECT DATEADD(dd,-(DATEPART(dw, GETDATE())-1), GETDATE())
```

### End Of This Week

```
SELECT DATEADD(dd,-(DATEPART(dw, GETDATE())-1), GETDATE())
```

### Start Of Last Week

```
SELECT DATEADD(day,-7,DATEADD(wk,DATEDIFF(wk,6,GETDATE()),6))
```

<!-- ### End Of Last Week -->

<!-- ### Start Of Next Week -->

<!-- ### End Of Next Week -->

 ### Start Of Current Month

```
SELECT DATEADD(month,DATEDIFF(month,0,GETDATE()),0)
```

For SQL Server 2012 and above you could use, rememebr, [this only returns a DATE](/posts/2021-09-06-be-careful-when-using-eomonth/) 

```
SELECT DATEADD(d,1,EOMONTH(GETDATE(),-1))
```

### End Of Current Month

For SQL Server 2012 and above you could use, rememebr, [this only returns a DATE](/posts/2021-09-06-be-careful-when-using-eomonth/) 

```
SELECT EOMONTH(GETDATE())
```

### Next Month

```
SELECT DATEADD(mm,1,GETDATE())
```

<!-- ### Start Of Next Month -->

### End Of Next Month

***SQL Server 2012+***

```
SELECT EOMONTH(DATEADD(mm,1,GETDATE()))
```
<!-- ### Last Month

### Start Of Last Month -->

### End Of Last Month

***SQL Server 2012+***

```
SELECT EOMONTH(DATEADD(mm,-1,GETDATE()))
```
