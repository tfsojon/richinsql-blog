---
title: Local SQL Server Test Lab In Hyper-V
date: 2022-02-28T09:00:00.000+01:00
image: "/images/post/apple-health-powerBi-dashboard.jpg"
author: Rich
layout: post
draft: true
categories:
- sqlserver
tags:
- SQLServer
- T-SQL
- Advanced
---

# Contents 

- [Contents](#contents)
    - [Introduction](#introduction)
    - [My Workstation Specifications](#my-workstation-specifications)
    - [Create Virtual Machine In Hyper-V](#create-virtual-machine-in-hyper-v)
    - [Install Windows Server](#install-windows-server)
    - [Configuring Windows](#configuring-windows)
    - [Installing Domain Controller Role](#installing-domain-controller-role)
    - [Creating a user account in active directory](#creating-a-user-account-in-active-directory)
    - [Joining server to the domain](#joining-server-to-the-domain)
    - [Install Microsoft SQL Server](#install-microsoft-sql-server)
    - [Installing Failover Feature](#installing-failover-feature)
    - [Creating a failover cluster](#creating-a-failover-cluster)
    - [Configuring quorum and file share whitness](#configuring-quorum-and-file-share-whitness)
    - [Configuring SQL Server Service For Availabiltiy Group](#configuring-sql-server-service-for-availabiltiy-group)
    - [Creating SQL Server Availability Group](#creating-sql-server-availability-group)

### Introduction

In this post we are going to take a look at creating a Test Lab on our local machine, if like me you don't have a home lab where you can install lots of different servers creating a home lab on your workstation could be the next best thing. 

### My Workstation Specifications

Here are the specifications of my workstation which I will be using to Install my lab onto. 

- Ryzen 5 6400 
- 16GB Ram 
- 1TB Western Digital Hard Drive
- Windows 11 Enterprise 

The full specifications can be found [here](/gear)

###  Create Virtual Machine In Hyper-V

First we need to create a new virtual machine in hyper-v. 

1. Load the Hyper-V application 

![](/img/setup-hyperv-1.png)

1. From the Actions menu, select New then choose Virtual Machine

![](/img/setup-hyperv-2.png)

1. Enter the name of the Virtual machine, I started with the Domain Controller which I called DC01

![](/img/setup-hyperv-3.png)

4. Specifiy the generation you wish to use, I selected Generation 2 

![](/img/setup-hyperv-4.png)

5. For the memory, I gave each machine 2GB of RAM. 

![](/img/setup-hyperv-5.png)

6. Connect the Hyper-V Default Switch to the virtual machine

![](/img/setup-hyperv-6.png)

7. Leave the virtual hard disk options at default 

![](/img/setup-hyperv-7.png)

8. From the installlation options, select "Install an operating system from a bootable image file" and specify where the ISO for Windows Server is saved. 

![](/img/setup-hyperv-8.png)

![](/img/setup-hyperv-9.png)

9.  Once you are happy click Finish and Hyper-V will create the virtual machine

![](/img/setup-hyperv-10.png)

You will need to repeat these steps twice more so that in the end you have the following 

| Server Role | Server Name | IP Address |
|-------------|-------------|------------|
| Domain Controller | DC01 | 10.10.10.1 |
| SQL Server | SQL01 | 10.10.10.100 |
| SQL Server | SQL02 | 10.10.10.101 |


###  Install Windows Server

![](/img/install-win-serv1.png)
![](/img/install-win-serv2.png)
![](/img/install-win-serv3.png)
![](/img/install-win-serv4.png)
![](/img/install-win-serv5.png)
![](/img/install-win-serv6.png)
![](/img/install-win-serv7.png)
![](/img/install-win-serv8.png)
![](/img/install-win-serv9.png)
![](/img/install-win-serv10.png)
![](/img/install-win-serv11.png)
![](/img/install-win-serv12.png)
![](/img/install-win-serv13.png)
![](/img/install-win-serv14.png)

###  Configuring Windows

![](/img/config-windows-1.png)
![](/img/config-windows-2.png)
![](/img/config-windows-3.png)
![](/img/config-windows-4.png)
![](/img/config-windows-5.png)
![](/img/config-windows-6.png)
![](/img/config-windows-7.png)
![](/img/config-windows-8.png)
![](/img/config-windows-9.png)
![](/img/config-windows-10.png)
![](/img/config-windows-11.png)
![](/img/config-windows-12.png)
![](/img/config-windows-13.png)
![](/img/config-windows-14.png)
![](/img/config-windows-15.png)

###  Installing Domain Controller Role

![](/img/installing-dc-role-1.png)
![](/img/installing-dc-role-2.png)
![](/img/installing-dc-role-3.png)
![](/img/installing-dc-role-4.png)
![](/img/installing-dc-role-5.png)
![](/img/installing-dc-role-6.png)
![](/img/installing-dc-role-7.png)
![](/img/installing-dc-role-8.png)
![](/img/installing-dc-role-9.png)
![](/img/installing-dc-role-10.png)
![](/img/installing-dc-role-11.png)
![](/img/installing-dc-role-12.png)
![](/img/installing-dc-role-13.png)
![](/img/installing-dc-role-14.png)
![](/img/installing-dc-role-15.png)
![](/img/installing-dc-role-16.png)
![](/img/installing-dc-role-17.png)
![](/img/installing-dc-role-18.png)
![](/img/installing-dc-role-19.png)
![](/img/installing-dc-role-20.png)

###  Creating a user account in active directory

![](/img/creating-ad-user-1.png)
![](/img/creating-ad-user-2.png)
![](/img/creating-ad-user-3.png)
![](/img/creating-ad-user-4.png)
![](/img/creating-ad-user-5.png)
![](/img/creating-ad-user-6.png)
![](/img/creating-ad-user-7.png)
![](/img/creating-ad-user-8.png)
![](/img/creating-ad-user-9.png)
![](/img/creating-ad-user-10.png)

###  Joining server to the domain

![](/img/join-server-domain-1.png)
![](/img/join-server-domain-2.png)
![](/img/join-server-domain-3.png)
![](/img/join-server-domain-4.png)
![](/img/join-server-domain-5.png)
![](/img/join-server-domain-6.png)
![](/img/join-server-domain-7.png)
![](/img/join-server-domain-8.png)
![](/img/join-server-domain-9.png)

###  Install Microsoft SQL Server

###  Installing Failover Feature

![](/img/failover-feature-1.png)
![](/img/failover-feature-2.png)
![](/img/failover-feature-3.png)
![](/img/failover-feature-4.png)
![](/img/failover-feature-5.png)
![](/img/failover-feature-6.png)
![](/img/failover-feature-7.png)
![](/img/failover-feature-8.png)

###  Creating a failover cluster

![](/img/failover-cluster-1.png)
![](/img/failover-cluster-2.png)
![](/img/failover-cluster-3.png)
![](/img/failover-cluster-4.png)
![](/img/failover-cluster-5.png)
![](/img/failover-cluster-6.png)
![](/img/failover-cluster-7.png)
![](/img/failover-cluster-8.png)
![](/img/failover-cluster-9.png)
![](/img/failover-cluster-10.png)
![](/img/failover-cluster-11.png)
![](/img/failover-cluster-12.png)
![](/img/failover-cluster-13.png)
![](/img/failover-cluster-14.png)
![](/img/failover-cluster-15.png)
![](/img/failover-cluster-16.png)
![](/img/failover-cluster-17.png)

###  Configuring quorum and file share whitness

![](/img/quorum-setup-1.png)
![](/img/quorum-setup-2.png)
![](/img/quorum-setup-3.png)
![](/img/quorum-setup-4.png)
![](/img/quorum-setup-5.png)
![](/img/quorum-setup-6.png)
![](/img/quorum-setup-7.png)
![](/img/quorum-setup-8.png)
![](/img/quorum-setup-9.png)
![](/img/quorum-setup-10.png)
![](/img/quorum-setup-11.png)
![](/img/quorum-setup-12.png)
![](/img/quorum-setup-13.png)
![](/img/quorum-setup-14.png)

###  Configuring SQL Server Service For Availabiltiy Group

![](/img/sql-service-config-ag-1.png)
![](/img/sql-service-config-ag-2.png)
![](/img/sql-service-config-ag-3.png)
![](/img/sql-service-config-ag-4.png)
![](/img/sql-service-config-ag-5.png)
![](/img/sql-service-config-ag-6.png)
![](/img/sql-service-config-ag-7.png)
![](/img/sql-service-config-ag-8.png)

###  Creating SQL Server Availability Group

![](/img/sql-service-config-ag-1.png)
![](/img/sql-service-config-ag-2.png)
![](/img/sql-service-config-ag-3.png)
![](/img/sql-service-config-ag-4.png)
![](/img/sql-service-config-ag-5.png)
![](/img/sql-service-config-ag-6.png)
![](/img/sql-service-config-ag-7.png)
![](/img/sql-service-config-ag-8.png)
![](/img/sql-service-config-ag-9.png)
![](/img/sql-service-config-ag-10.png)
![](/img/sql-service-config-ag-11.png)
![](/img/sql-service-config-ag-12.png)
![](/img/sql-service-config-ag-13.png)