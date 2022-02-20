---
title: Build A SQL Server Availability Group Lab In Hyper-V
date: 2022-02-21T09:00:00.000+01:00
image: "/images/post/availability-group-lab.jpg"
author: Rich
layout: post
draft: false
categories:
- sqlserver
tags:
- SQLServer
- Testing
- Server
- Advanced
featured: true
---

### Introduction

In this post we are going to look at creating a Test Lab on our local machine, if like me you don't have a home lab where you can install lots of different servers, creating a home lab on your workstation could be the next best thing. 

This post covers all the steps except installing SQL Server itself. 

# Contents 

- [Contents](#contents)
    - [My Workstation Specifications](#my-workstation-specifications)
    - [Create Virtual Machine In Hyper-V](#create-virtual-machine-in-hyper-v)
    - [Install Windows Server](#install-windows-server)
    - [Configuring Windows](#configuring-windows)
      - [Rename the server](#rename-the-server)
      - [Change network settings](#change-network-settings)
        - [SQL Server 1](#sql-server-1)
        - [SQL Server 2](#sql-server-2)
      - [Change firewall settings](#change-firewall-settings)
    - [Installing Domain Controller Role](#installing-domain-controller-role)
    - [Creating a user account in active directory](#creating-a-user-account-in-active-directory)
    - [Joining server to the domain](#joining-server-to-the-domain)
    - [Install Microsoft SQL Server](#install-microsoft-sql-server)
    - [Installing Failover Feature](#installing-failover-feature)
    - [Creating a failover cluster](#creating-a-failover-cluster)
    - [Configuring quorum and file share witness](#configuring-quorum-and-file-share-witness)
      - [Create the fileshares](#create-the-fileshares)
      - [Configure the quorum](#configure-the-quorum)
    - [Configuring SQL Server Service For Availabiltiy Group](#configuring-sql-server-service-for-availabiltiy-group)
    - [Creating SQL Server Availability Group](#creating-sql-server-availability-group)

### My Workstation Specifications

Here are the specifications of my workstation which I will be using to Install my lab onto. 

- Ryzen 5 6400 
- 16GB Ram 
- 1TB Western Digital Hard Drive
- Windows 11 Enterprise 

The full specifications can be found [here](/gear)

###  Create Virtual Machine In Hyper-V

First, we need to create a new virtual machine in Hyper-V. You can use any Hypervisor of your choosing, if you decide to not use Hyper-V skip this step but make a note of the settings we are using. 

1. Load the **Hyper-V** application 

![](/img/setup-hyperv-1.png)

1. From the Actions menu, select **New** then choose **Virtual Machine**

![](/img/setup-hyperv-2.png)

1. Enter the name of the Virtual machine, I started with the Domain Controller which I called DC01

![](/img/setup-hyperv-3.png)

4. Specify the generation you wish to use, I selected **Generation 2** because it supports UEFI and newer virtualization features. 

Click **Next**

![](/img/setup-hyperv-4.png)

5. For the memory, I gave each machine **2GB** of RAM and left the Dynamic Memory allocation option checked.

Click **Next**

![](/img/setup-hyperv-5.png)

6. Connect the Hyper-V **Default Switch** to the virtual machine

Click **Next**

![](/img/setup-hyperv-6.png)

7. Leave the virtual hard disk options at default 

Click **Next**

![](/img/setup-hyperv-7.png)

8. From the installation options, select "Install an operating system from a bootable image file" and specify where the ISO for Windows Server is saved. 

Click **Next**

![](/img/setup-hyperv-8.png)

![](/img/setup-hyperv-9.png)

9.  Once you are happy click **Finish** and Hyper-V will create the virtual machine

![](/img/setup-hyperv-10.png)

You will need to repeat these steps twice more so that in the end you have the following 

| Server Role | Server Name | IP Address |
|-------------|-------------|------------|
| Domain Controller | DC01 | 10.10.10.1 |
| SQL Server | SQL01 | 10.10.10.100 |
| SQL Server | SQL02 | 10.10.10.101 |

###  Install Windows Server

Windows Server is the base operating system that we are going to use for all three of the servers required for this lab, I am using Windows Server 2019 Standard Edition Eval.

1. Select your language, Click **Next**

![](/img/install-win-serv1.png)

2. Click the **install now** button

![](/img/install-win-serv2.png)

3. Select which version you want, go with the *Desktop Experience* if you want the UI.

![](/img/install-win-serv3.png)

4. Click **Custom: Install Windows Only**

![](/img/install-win-serv4.png)

5. Agree to the license, Click **Next**

![](/img/install-win-serv5.png)

6. Click the **new** button from the window that appears 

![](/img/install-win-serv6.png)

7. We are going to use all the disk, so leave the size as it is and click **Apply**.

![](/img/install-win-serv7.png)

8. You will get an information window informing you that that installer will create more partition for system files, you can click **ok** to this. 

![](/img/install-win-serv8.png)

9. The location where you want the operating system to install will automatically be selected, click **Next**.

![](/img/install-win-serv9.png)

10. Windows will now begin installing 

![](/img/install-win-serv10.png)

11. Once it is complete the virtual machine will restart, if you don't want to wait, Click **Restart Now**

![](/img/install-win-serv11.png)

12. You will then be asked to provide a password for the local admin account, you will need this in future steps so record it somewhere. Click **Finish**

![](/img/install-win-serv12.png)

13. Windows will then finish things up. 

![](/img/install-win-serv13.png)

14. You should now be in a position to login and Windows successfully be installed.

![](/img/install-win-serv14.png)

###  Configuring Windows

Now that Windows is installed we can begin to configure it. 

It is worth noting that this is a test lab, with no access to the internet, some of the practices here are not to be used in a production environment but are acceptable for our lab. 

With that out of the way, lets begin. 

#### Rename the server

1. Open up file explorer. you can use the **Windows & E** key combo to get to this. 

![](/img/config-windows-1.png)

2. From here right click on **This PC** and select **properties**.

![](/img/config-windows-2.png)

3. From the window that appears, click **Change Settings** under the sub-heading **Computer name, domain and workgroup settings**

![](/img/config-windows-3.png)

4. In the **system properties** window click the **Change** button

![](/img/config-windows-4.png)

5. Enter a name for the server, DC01 for the Domain Controller, SQL01 for the first SQL Server and SQL02 for the second SQL Server. But you can choose whatever you want as long as you know what they all mean. 

Click **OK**.

![](/img/config-windows-5.png)

6. You will get a warning that the computer needs to be restarted, Click **OK**.  

![](/img/config-windows-6.png)

7. When prompted click the **Restart Now** button

![](/img/config-windows-7.png)

That concludes the configuration of the Server, repeat this for each of the servers you build. 

If you are using Hyper-V it is also a good idea to checkpoint your servers after each step in case something goes wrong. 

#### Change network settings

1. Open the **settings** application from the **start menu** and go to **Network & Internet**, Select **Ethernet** from the left menu then select **Change Adapter Options**.

![](/img/config-windows-8.png)

2. Right click on the **adapter** and select **properties**. 

![](/img/config-windows-9.png)

3. Select the **IPv4 Internet Protocol** and click **Properties**

![](/img/config-windows-10.png)

4. Enter the IP address information as the screenshot shows for the Domain Controller. Configuration for the other two servers is below. 

##### SQL Server 1

- **IP Address:** 10.10.10.100
- **Gateway:** 10.10.10.1
- **Preferred DNS:** 10.10.10.1

##### SQL Server 2

- **IP Address:** 10.10.10.101
- **Gateway:** 10.10.10.1
- **Preferred DNS:** 10.10.10.1

![](/img/config-windows-11.png)

#### Change firewall settings

1. From the **Server Manager**, click **Tools** and select **Windows Defender with Advanced Security**

![](/img/config-windows-12.png)

2. Click **Windows Defender Firewall Properties**

![](/img/config-windows-13.png)

3. In each of the tabs set the Firewall state to **Off** and Click **Apply**

![](/img/config-windows-14.png)

4. The firewall should now be disabled for all network types

**REMINDER:** Don't do this in production

###  Installing Domain Controller Role

The first thing we need to do is install the domain controller role on DC01, this is so that we can join the other servers to it and create a small, networked domain inside the lab.

This doesn't need to be done on the SQL Servers.

1. From the **Server Manager** click **Add roles and features**

![](/img/installing-dc-role-1.png)

2. Click **next** on the Before you begin screen

![](/img/installing-dc-role-2.png)

3. From the installation type screen, select **Role-based or feature based instalation**

![](/img/installing-dc-role-3.png)

4. Click **Select a server from the server pool** then select your server from the list 

![](/img/installing-dc-role-4.png)

5. Check **Active Directory Domain Services**

![](/img/installing-dc-role-5.png)

6. You will be asked to add additional features, these are required so click the **Add Features** button.

Click **Next**

![](/img/installing-dc-role-6.png)

7. On the Select Features page, leave the options selected and click **Next**. 

![](/img/installing-dc-role-7.png)

8. Click **Next** on the **Active Directory Domain Services** page.

![](/img/installing-dc-role-8.png)

9. Click **install** on the confirmation screen

![](/img/installing-dc-role-9.png)

10. Installation should now begin

![](/img/installing-dc-role-10.png)

11. Once the installation has completed, click **Promote this server to a domain controller**

![](/img/installing-dc-role-11.png)

12. Select **Add a new forest** and enter a name for your new forest, I went with **richinsql.local**

Once you are happy, click **next**.

![](/img/installing-dc-role-13.png)

1.  Enter a **DRSM password**, make a note of this somewhere 

![](/img/installing-dc-role-14.png)

14. Leave these settings as default and click **next**. 

![](/img/installing-dc-role-15.png)

15. Leave these settings as default and click **next**. 

![](/img/installing-dc-role-16.png)

1.  Leave these settings as default and click **next**.

![](/img/installing-dc-role-17.png)

17. Leave these settings as default and click **next**. 

![](/img/installing-dc-role-18.png)

18. Leave these settings as default and click **Install**. 

![](/img/installing-dc-role-19.png)

19. The domain services role will now be installed, once this is complete, you will need to reboot the server. 

![](/img/installing-dc-role-20.png)

###  Creating a user account in active directory

Now that the domain service role is installed on the domain controller, we need to add a user account with the Domain Admin group applied.

1. From the **Server Manager** on DC01 click the **Tools** button, then select **Active Directory Users & Computers** from the list.

![](/img/creating-ad-user-1.png)

2. Expand the forest you created in the previous step, then right click on **Users** and select **New** then **User**. 

![](/img/creating-ad-user-2.png)

3. Enter some details for your user in the box that appears. 
   
Click **Next**

![](/img/creating-ad-user-3.png)

4. Enter a password for your user, uncheck the **User must change password at next login** box and check the **password never expires** box. 

Click **Next**

![](/img/creating-ad-user-4.png)

5. Click **Finish**

![](/img/creating-ad-user-5.png)

6. Click the **users** folder from the tree view, find your user in the list. 

![](/img/creating-ad-user-6.png)

7. Right click on your account and select **Properties**

![](/img/creating-ad-user-7.png)

8. Click the **Member Of** tab and press the **Add** button

![](/img/creating-ad-user-8.png)

9. Type **Domain Admins** into the box and click **Check Names** 

Click **Ok**.

![](/img/creating-ad-user-9.png)

10. Click **Ok**

![](/img/creating-ad-user-10.png)

Your domain account is now setup, you will be able to use this account to login to the domain from each of the servers in the lab. 

###  Joining server to the domain

With our user account now successfully created, we need to get the other two servers joined to the domain. If you haven't yet created the two SQL Servers, go and do that now and configure them as per the [Configuring Windows](#configuring-windows) section and come back. 

1. Open up **File Explorer**, you can do this by pressing **Windows & E** on your keyboard

![](/img/join-server-domain-1.png)

2. Right click on **This PC** and select **Properties**

![](/img/join-server-domain-2.png)

3. From the **System** window, click **Change Settings**

![](/img/join-server-domain-3.png)

4. From the **System Properties** screen, click the **Change** Button

![](/img/join-server-domain-4.png)

5. From the **Computer Name/Domain Changes** screen change the radio button from **Workgroup** to **Domain** and enter the name of the domain you created when you created the forest earlier. 

![](/img/join-server-domain-5.png)

6. You should be asked to enter the username and password of someone who has permission to join the domain, that will be the user account you created in the previous step. Enter those details and click **Ok**. 

![](/img/join-server-domain-6.png)

7. You should now be presented with a box welcoming you to the domain. Click **Ok**. 

![](/img/join-server-domain-7.png)

8. Click **Close**

![](/img/join-server-domain-8.png)

9. Click **Ok**

![](/img/join-server-domain-9.png)

10. Click **Restart**

![](/img/join-server-domain-10.png)

When the server reboots, you will need to log back in with the domain user account. 

###  Install Microsoft SQL Server

Installing SQL Server needs to be completed on both of the SQL Servers you created, I am not going to walk through how to Install SQL Server, but you need to do that here, I used SQL Server 2017 Developer Edition.

###  Installing Failover Feature

This needs to be completed on both SQL Servers that we are going to add to the Failover Cluster. 

1. From the **server manager**, Click **Add Roles and Features**

![](/img/failover-feature-1.png)

2. Click **Role-Based or feature based installation**

![](/img/failover-feature-2.png)

3. Click S**elect A Server from the server pool**

![](/img/failover-feature-3.png)

4. Check **Failover Clustering**

![](/img/failover-feature-4.png)

5. Add the additional features that you are prompted to add. 

![](/img/failover-feature-5.png)

6. Click **Install**

![](/img/failover-feature-6.png)

7. Installation of the role should now begin

![](/img/failover-feature-7.png)

8. Once complete, click **Close** and reboot the server.

![](/img/failover-feature-8.png)

###  Creating a failover cluster

This only needs to be completed on one of the SQL Servers but can't be completed until the previous step is completed on both servers. 

1. From the **Server Manager**, Click **Tools** then Select **Failover Cluster Manager**

![](/img/failover-cluster-1.png)

2. Click **Create Cluster** from the Action menu

![](/img/failover-cluster-2.png)

3. Click **Next**

![](/img/failover-cluster-3.png)

4. Click **browse**

![](/img/failover-cluster-4.png)

5. Enter the name of the two SQL Servers separated with a semicolon and click **Check Names**, then click **Next**.

![](/img/failover-cluster-5.png)

6. Leave as default and Click **Next**

![](/img/failover-cluster-6.png)

7. Leave as default and Click **Next**

![](/img/failover-cluster-7.png)

8. Leave as default and Click **Next**

![](/img/failover-cluster-8.png)

9. Leave as default and Click **Next**

![](/img/failover-cluster-9.png)

10. Leave as default and Click **Next**

![](/img/failover-cluster-10.png)

11. Validation tests will now be carried out on the configuration

![](/img/failover-cluster-11.png)

12. Click **Finish**

![](/img/failover-cluster-12.png)

13. Enter a name for your cluster in the **Cluster Name** box, you can use something like **cluster1** 

Click in the **Address** box and enter an IP Address for the Cluster

Click **Next**

![](/img/failover-cluster-13.png)

14. Leave as default and Click **Next**

![](/img/failover-cluster-14.png)

15. The Cluster will now be created

![](/img/failover-cluster-15.png)

16. Once the Cluster has been completed, Click **Finish**

![](/img/failover-cluster-16.png)

17. Back in the **Cluster Manager**, Click **Nodes** from the cluster tree on the left, you should now see both nodes you added appear as up.

![](/img/failover-cluster-17.png)

###  Configuring quorum and file share witness

We now need to Configuring quorum and file share witness, this involves creating two directories somewhere in the lab, I decided to put them on the Domain controller, once that is done, we then need to configure the failover cluster quorum settings so it knows about the witness. 

#### Create the fileshares

Back on the domain controller we need to create two file shares. 

- Witness - A witness log file is maintained by the cluster, it contains information about the health of the cluster, witness and quorum voting. Each node has a cluster service running that periodically checks in with the other nodes and witnesses to validate the health of the cluster. If a node cannot communicate with other nodes, it will then verify connectivity with the witness, access the witness log and decide if based on the vote if it should take control. 
- SQLBackups - this will be used by the availability group to back up the databases in the AG when joining them, this is like a scratch space, it isn't used for anything else once the database is added to the AG. 

1. Open up file explorer and create the above two directories somewhere on the local filesystem.

![](/img/quorum-setup-1.png)

2. Right click on the folder created and select **Properties** 

![](/img/quorum-setup-2.png)

3. Click the **Sharing** tab

![](/img/quorum-setup-3.png)

4. Tick the Share this folder checkbox, then click **Permissions**

![](/img/quorum-setup-4.png)

5. Add **Everyone** with full permission.

**DONT** do this in a production environment, you should limit the access to those who need it, but for our lab, this will be fine. 

Click Ok.

![](/img/quorum-setup-5.png)

#### Configure the quorum

Jump back over to one of the SQL Servers and load up the Failover Cluster Manager.

1. From the actions menu on the right, click the **More Actions** button then click **Configure Cluster Quorum Settings**.

![](/img/quorum-setup-6.png)

2. Click **Next**

![](/img/quorum-setup-7.png)

3. Change the option to **Select the quorum witness**

Click **Next**

![](/img/quorum-setup-8.png)

4. Change the option to **Configure a file share witness** 

Click **Next**

![](/img/quorum-setup-9.png)

5. Click **Browse**

![](/img/quorum-setup-10.png)

6. Enter the name of your domain controller, click the **Witness** folder and Click **Ok**.

![](/img/quorum-setup-11.png)

7. Click **Next**

![](/img/quorum-setup-12.png)

8. Click **Finish**

![](/img/quorum-setup-13.png)

7. Scroll down to **Cluster Core Resources**, you should now see the file share witness showing as an active resource.

![](/img/quorum-setup-14.png)

###  Configuring SQL Server Service For Availabiltiy Group

With SQL Server installed and your failover cluster created, we need to make some changed to the SQL Service so that it can be used as an availability group. 

1. From the start menu, find SQL Server [verion] Configuration Manager 

![](/img/sql-service-config-ag-1.png)

2. Click the **SQL Server Services** option from the left menu, then select **SQL Server** and right click it, selecting **properties** from the menu that appears. 

![](/img/sql-service-config-ag-2.png)

3. Change the account to the domain account you created earlier - *I had to do this as I was running into issues when creating the availability group*. Click **Apply**.

![](/img/sql-service-config-ag-3.png)

4. You will get a warning that doing this will cause the Service to restart, which in production would cause users to lose connection to the database.

![](/img/sql-service-config-ag-4.png)

5. Next click the **AlwaysOn High Availabiltiy** tab and check the **Enable AlwaysOn Availability Groups**.

Click **Apply** then **Ok**. 

![](/img/sql-service-config-ag-5.png)

6. Expand the SQL Server Network Configuration, select **Protocols** for MSSQLSERVER and enable **TCP/IP** if it isn't already enabled. 

![](/img/sql-service-config-ag-6.png)

![](/img/sql-service-config-ag-7.png)

7. Go back to the SQL Server Services and right click **SQL Server** and select **Restart** 

![](/img/sql-service-config-ag-8.png)

You will need to carry out these steps on **BOTH** of the SQL Servers.

###  Creating SQL Server Availability Group

1. From SQL Management Studio, connect to SQL1, Expand the tree, right click on Always On Availability Group and Click New Availability Group Wizard.

![](/img/sql-ag-1.png)

2. Click **Next** on the Introduction window

![](/img/sql-ag-2.png)

3. Enter an availability group name, this can be anything you want, I went with AG1, Click **Next**.

![](/img/sql-ag-3.png)

4. Select the databases you would like to put into the Availability Group, you must select at least one and it must; 

- Be in Full Recovery Mode
- Have a recent FULL backup 

![](/img/sql-ag-4.png)

5. Click the **Add Replica** Button 

![](/img/sql-ag-5.png)

6. Add SQL2 into the Replicas

![](/img/sql-ag-6.png)

7. Click the listener tab, select **Create an availability group listener now** and enter AG1 into the DNS name, the port can be 1433 

This is going to let us connect to our availability group by simply supplying AG1 in the username field in SQL Management Studio and return the primary instance. 

Click **Next**.

![](/img/sql-ag-7.png)

8. From the synchronization preferences, select Full database & log backup, enter the path to the SQL Backups share you created earlier. 

Click **Next**.

![](/img/sql-ag-8.png)

9. A number of checks will be carried out, if everything went well you should get a list of green check marks. 

![](/img/sql-ag-9.png)

10. Click **Finish**

![](/img/sql-ag-10.png)

11. The wizard will now attempt to create the availability group and add your database(s), if this succeeds you will get more green check marks. 

![](/img/sql-ag-11.png)

12. In SQL Management Studio connect to a new SQL Instance and enter the name you entered into the listener step. 

![](/img/sql-ag-12.png)

13. You should now be connected to your availability group with both nodes showing as up and the database(s) synchronised. 

![](/img/sql-ag-13.png)

This concludes the configuration of the lab, you should now have a functional SQL Server Availability group lab running inside Hyper-V.