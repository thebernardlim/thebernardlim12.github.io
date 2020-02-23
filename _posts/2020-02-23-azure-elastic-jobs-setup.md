---
layout: post
title:  "Azure Elastic Jobs - Setup Guide"
author: Bernard Lim
date:   2020-02-23 00:00:00 +0000
category: azure
summary: Guide on setting up Azure Elastic Jobs
thumbnail: elasticjobagents.png
---

# Azure Elastic Job Setup Guide

## Create Jobs Database

1. Login to Azure Portal. <br/>
![sql db](/assets/img/posts/2020-02-23-azure-elastic-jobs-setup/create-jobs-db-1.png)

2. Type ‘SQL Databases’ on Search box located at the top of the page. Click on ‘SQL databases’ option.
![add sql db](/assets/img/posts/2020-02-23-azure-elastic-jobs-setup/create-jobs-db-2.png)
 
3. Click on ‘Add’ within the ‘SQL databases’ page.
 
4. In the ‘Create SQL Database’ page, fill in the following fields accordingly: <br/>

    **Subscription** – Subscription where Jobs Database will be billed against <br/>
    **Resource group** – Resource group to logically group Jobs Database <br/>
    **Database name** – Name of Jobs Database <br/>
    **Server** – Choose SQL Server to host Jobs Database

5. Click on ‘Configure database’ for ‘Compute + storage’ field
6. In the ‘Configure’ page, choose a tier which will be either ‘Standard’ or beyond. Elastic Jobs Databases will require a **tier of S0 and above**
<img src="/assets/img/posts/2020-02-23-azure-elastic-jobs-setup/create-jobs-db-3.png" width="650px" />
 
7. Fill up remaining fields as per required, and proceed to click ‘Review + Create’ to create Jobs Database.

## Enable access to Jobs Database from client machine

1. Ensure that the Job SQL Server firewall allows access to your client, by clicking ‘Set server firewall’ and subsequently adding your Client IP.
<img src="/assets/img/posts/2020-02-23-azure-elastic-jobs-setup/enable-access-1.png" width="750px" />

## Create Elastic Job Agent

1. Type  ‘Elastic Job Agents’ on Search box located at the top of the page. Click on ‘Elastic Job Agents’ option.
<img src="/assets/img/posts/2020-02-23-azure-elastic-jobs-setup/create-elastic-job-agent-1.png" width="650px" />

2. Click on ‘Add’. <br/>
![elasticjob2](/assets/img/posts/2020-02-23-azure-elastic-jobs-setup/create-elastic-job-agent-2.png)

3. On the ‘Elastic Job agent’ page, <br/>
a. Fill up the following fields:  <br/>
   **Name** – Name of Job Agent <br/> 
   **Subscription** – Subscription where Elastic Job costs will be billed against <br/>
b. Click on ‘Preview terms’ and check the checkbox to accept the terms. <br/>
c. Click on ‘Job database’, and select the server and Jobs Database which was created earlier.
![elasticjob3](/assets/img/posts/2020-02-23-azure-elastic-jobs-setup/create-elastic-job-agent-3.png)

4. Click on ‘Create’ to create Job Agent
5. To verify that the Elastic Job Agent has successfully connected to the Jobs Database, access the Jobs Database through SQL Server Management Studio. A number of tables & stored procedures should have been created to indicate successful connection.
![elasticjob4](/assets/img/posts/2020-02-23-azure-elastic-jobs-setup/create-elastic-job-agent-4.PNG)

## Create Elastic Job

### Job Database Server Configurations
1. Connect to the Job Database Server through SSMS.
2. Create Database Scoped Credential.

```sql
--Connect to the job database specified when creating the job agent

-- Create a db master key if one does not already exist, using your own password.  
CREATE MASTER KEY ENCRYPTION BY PASSWORD='<EnterStrongPasswordHere>';  
  
-- Create a database scoped credential – Identity used to execute job against target database 
CREATE DATABASE SCOPED CREDENTIAL myjobcred WITH IDENTITY = 'jobcred',
    SECRET = '<EnterStrongPasswordHere>'; 
GO

-- Create a database scoped credential – Identity to connect to master database
CREATE DATABASE SCOPED CREDENTIAL mymastercred WITH IDENTITY = 'mastercred',
    SECRET = '<EnterStrongPasswordHere>'; 
GO
```

3. Create a Target Group

```sql
-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group '<Target Group Name>'

-- Add a server target member
EXEC jobs.sp_add_target_group_member
'<Target Group Name>',
@target_type = 'SqlServer',
@refresh_credential_name='mymastercred', --credential required to refresh the databases in server
@server_name='<Target Database Server Name>'

```
### Target Database Server Configurations

1.	Connect to the *Target Database Server* through SSMS
2.	Create Login & User objects in master database in target database server 

```sql
-- Create 'mastercred' Login & User in target server master db
CREATE LOGIN mastercred WITH PASSWORD = 'Password1'
CREATE USER mastercred FROM LOGIN mastercred

-- Create 'jobcred' Login in target server master db
CREATE LOGIN jobcred WITH PASSWORD = 'Password1'

```
3. Create user object to run queries and grant required permissions in target database
```sql
-- Create 'jobcred' User in target server target db
CREATE USER jobcred FROM LOGIN jobcred

-- Grant required permissions to user
GRANT ALTER ON SCHEMA::dbo TO jobcred
GRANT SELECT,UPDATE,INSERT,DELETE,EXEC TO jobcred

```

### Job Database – Add Job

1. Add job (and specify frequency)
```sql
-- Add job
EXEC jobs.sp_add_job 
	@job_name='BatchUpdateJob',
	@description='Job to run to update x tables',
	@schedule_interval_type='Hours',
	@schedule_interval_count=1

```
2. Add job steps for job

```sql
-- Add job step for create table
EXEC jobs.sp_add_jobstep @job_name='Step1',
@command=N'EXEC [dbo].[teststoredproc] ',
@credential_name='myjobcred',
@target_group_name='Group1'
```

## Official References

https://docs.microsoft.com/bs-cyrl-ba/azure//sql-database/sql-database-job-automation-overview#elastic-database-jobs-preview
https://docs.microsoft.com/en-us/azure/sql-database/elastic-jobs-tsql
