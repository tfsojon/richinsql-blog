---
title: Installing Visual Studio SQL Intergration Services
date: 2021-08-09T09:00:00+01:00
author: Rich
layout: post
permalink: /installing-visual-studio-sql-server-intergration-services
categories:
  - sqlserver
tags:
  - SSIS
  - SQL
  - VisualStudio
---

In this post, I am going to show you how to install SQL Server Intergration services for Visual Studio 2019, SSDT or SQL Server Data Tools no longer exists for Visual Studio 2019 so installation is a little bit different for newer versions of Visual Studio. 

<!--more-->

First we are going to need to make sure we have Data Storage & Processing installed, Open up Visual Studio and select Create New Blank Project, from the bottom of the template window select Install more tools and features as shown below; 

![](/img/vs-more-tools.png)

One that Window has opened, you will need to scroll down until you come to Other Toolsets, from this category select Data Storage and Processing.

![](/img/vs-data-storage-processing.png)

Tick the box and then select Modify from the lower right corner of the window. **You will need to make sure Visual Studio is fully closed**. 

One the installation has completed, you will need to load up Visual Studio, from the top menu select **Extensions**.

![](/img/vs-extensions1.png)

Then select **Manage Extensions**

![](/img/vs-extensions2.png)

A new window will open within this window is a search box to the upper right corner, enter **SQL Server Intergration Services Projects** and press enter, the SQL Server Intergration Services Projects extension should appear, click **Download**.

![](/img/vs-extensions3.png)

Your web browser of choice will then open and a download will begin, save this in a location of your choosing, I selected my downloads folder.

![](/img/vs-extensions4.png)

Once the download has finished, you going to need to locate the location which you selected to save the extension, once you have it double click the exe which in my case was **Microsoft.DataTools.IntegrationServices**

![](/img/vs-extensions5.png)

Next you will see a series of wizard windows, stepping you through the installation process.

![](/img/vs-extension-ssis-install-1.png)

![](/img/vs-extension-ssis-install-2.png)

![](/img/vs-extension-ssis-install-3.png)

![](/img/vs-extension-ssis-install-4.png)

![](/img/vs-extension-ssis-install-5.png)

Once the installation has completed, you can go ahead and load up Visual Studio, again from the window that appears, select Create new blank project. Clear anything that might be in the search box and enter **intergration** this should then bring up the option of **Intergration Services Project** which is what we want, click that and select Next. 

![](/img/vs-extension-ssis-new-proj.png)

You should now be able to start creating your SQL Server SSIS package from within Visual Studio 2019.

![](/img/vs-extension-ssis-new-proj-2.png)