---
title: Configuring Database Mail
date: 2021-12-06T09:00:00.000+01:00
author: Rich
layout: post
permalink: "/configuring-databasemail"
categories:
- sqlserver
tags:
- SQL
- T-SQL
draft:
    true
---

In this post we are going to look at how to configure Database Mail in SQL Server. Having Database mail configured on your SQL Server instance has many benefits including sending alerts when SQL Server encounters a problem to sending automated reports. 

### Prerequisites

* If you have Two Factor Authentication enabled on your Google Account you will need an application password 

Open up SQL Server Management Studio, Expand the **Management** Folder and right click Database Mail

![](/img/database-mail-0.png)

![](/img/database-mail-1.png)

![](/img/database-mail-2.png)

![](/img/database-mail-3.png)

![](/img/database-mail-4.png)

![](/img/database-mail-5.png)

![](/img/database-mail-6.png)

![](/img/database-mail-7.png)

![](/img/database-mail-8.png)

![](/img/database-mail-9.png)

![](/img/database-mail-10.png)

![](/img/database-mail-11.png)

![](/img/database-mail-12.png)

![](/img/database-mail-13.png)

### Outgoing Mail Server (SMTP)

* E-mail Address - Your gmail email address
* Server Name - As shown in the screenshot
* Port number - As shown in the screenshot
* The server requires a secure connection (SSL) - Make sure this is ticked.

### SMTP Authentication

* Select 'Basic authentication'
* User Name - Enter your gmail username
* Password - Enter password of your Gmail Account.
* Confirm Password - Enter password of your Gmail Account, if you have 2FA enabled you will need an application password.

