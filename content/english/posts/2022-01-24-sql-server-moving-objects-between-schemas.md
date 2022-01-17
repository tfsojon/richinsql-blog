---
title: SQL Server Moving Objects Between Schemas
date: 2022-01-24T09:00:00.000+01:00
image: "/images/post/moving-object-schema.jpg"
author: Rich
layout: post
categories:
- sqlserver
tags:
- SQL
- T-SQL
- Beginner
---

Something I always end up forgetting how to do is moving objects between schemas within a database, so I thought I would put together a small post to show how to do it. 

Lets assume you have an object on the dbo schema called Attendances and you want to move that table to a new schema called Students. 

![](/img/move-object-schema-1.png)

To do that you would use the following T-SQL

```
ALTER SCHEMA Students 
TRANSFER [dbo].[Attendances]
```

As you can see below, the object has now been moved to the new schema

![](/img/move-object-schema-2.png)

That is all there is to it. 

