---
title: Monitoring SQL Server With Perfmon
date: 2022-02-28T09:00:00.000+01:00
image: "/images/post/apple-health-powerBi-dashboard.jpg"
author: Rich
layout: post
categories:
- sqlserver
draft: true
tags:
- SQLServer
- T-SQL
- Advanced
---


### Which Counters To Use

Perfmon Object	Perfmon Counter
Memory	Available MBytes
Processor	% Processor Time
SQLServer:Access Methods	Forwarded Records/sec
SQLServer:Access Methods	Full scans/sec
SQLServer:Access Methods	Page Splits / Sec
SQLServer:Buffer Manager	Buffer Cache hit ratio
SQLServer:Buffer Manager	Checkpoint Pages / Sec
SQLServer:Buffer Manager	Page life expectancy
SQLServer:General Statistics	User Connections
SQLServer:Locks	Average Wait Time (ms)
SQLServer:Locks	Lock Waits / Sec
SQLServer:Memory Manager	Memory Grants Pending
SQLServer:Memory Manager	Target Server Memory (KB)
SQLServer:Memory Manager	Total Server Memory (KB)
SQLServer:SQL Statistics	Batch Requests/Sec
SQLServer:SQL Statistics	SQL Compilations/Sec
SQLServer:SQL Statistics	SQL Re-Compilations/Sec

