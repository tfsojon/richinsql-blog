---
title: "Availability Agent"
date: 2021-01-23T00:00:00Z
draft: true
description: "Availability agent will check which node in an availability group is master and disable jobs on secondary nodes."

weight: 1
categories: ["sqlserver"]

thumbnail: "images/scripts/availability-agent.png"
tools_documentation: "https://youtu.be/yRzeTQdLaIA"

tools_info:
- title: "Types :"
  content: "T-SQL"
#- title: "Colors :"
#  content: "Blue, Purple, White, Orange"

tools_images:
- image: "images/scripts/availability-agent.png"
#- image: "images/tools/figma.jpg"
---

Availabiltiy Agent is a T-SQL script that is designed to be created as a stored procedure and referenced inside a SQL Agent Job that runs on a regular frequency to disable agent jobs on the secondary node that would otherwise cause a job failure.

### Downloading

The script can be downloaded from the projects GitHub

### Installation 

These steps should be first carried out on the primary node in your availabiltiy group.

Run the install-soloution.sql script from the zip file, this will create the DBA_Tasks database if it doesn't already exist, if you already have a DBA database, update the script accordinly. 


