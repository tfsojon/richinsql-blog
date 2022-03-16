---
title: "SQL Server Execution Plans"
date: 2021-12-27T05:00:00Z
draft: false
description: "A Microsoft SQL Server database of Formula One information for training and development."

weight: 2
categories: ["SQL Server"]

thumbnail: "images/tools/f1-db.jpg"
book_link: "https://www.amazon.co.uk/Server-Execution-Plans-Grant-Fritchey/dp/190643493X/ref=sxts_rp_s1_0?cv_ct_cx=SQL+Server+Execution+Plans&dchild=1&keywords=SQL+Server+Execution+Plans&pd_rd_i=190643493X&pd_rd_r=ee4ddf7c-1b8f-49c6-a371-a70dd39a393b&pd_rd_w=SVVxy&pd_rd_wg=0DvTL&pf_rd_p=ceacd189-2bf5-431e-8894-7c8195b61116&pf_rd_r=XBR47BZKD4EK5AEX041T&psc=1&qid=1633858145&sr=1-1-eecbb009-a700-4b7c-89a4-776abc2e4acc"

book_info:
- title: "Author :"
  content: "Grant Fritchey"  
- title: "ISBN :"
  content: "190643493X"
- title: "Release Date :"
  content: "12 Oct. 2012"
---

Every day, out in the various online forums devoted to SQL Server, and on Twitter, the same types of questions come up repeatedly: Why is this query running slowly? Why is SQL Server ignoring my index? Why does this query run quickly sometimes and slowly at others? My response is the same in each case: have you looked at the execution plan? An execution plan describes what's going on behind the scenes when SQL Server executes a query. It shows how the query optimizer joined the data from the various tables defined in the query, which indexes it used, if any, how it performed any aggregations or sorting, and much more. It also estimates the cost of all of these operations, in terms of the relative load placed on the system. Every Database Administrator, developer, report writer, and anyone else who writes T-SQL to access SQL Server data, must understand how to read and interpret execution plans. My book leads you right from the basics of capturing plans, through how to interrupt them in their various forms, graphical or XML, and then how to use the information you find there to diagnose the most common causes of poor query performance, and so optimize your SQL queries, and improve your indexing strategy.