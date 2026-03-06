---
title: "Performance investigation in SQL server"
description: "How-To help the support team to investigate performance issues when using Microsoft SQL Server"
---

# Performance investigation in SQL server

In order to efficiently investigate performance issues, some information on Identity Analytics's infrastructure and configuration is required (dbcheck), such as :  

- The list of parameters
- The sizing of the database
- The index fragmentation
- The list of views
- And so on  

This article includes a SQL script, and it's associated documentation. The results should be sent to RadiantOne's support service to aide in the resolution of performance issues.  

## Prerequisites  

The following scripts have been tested using SQL Server 2008 to 2016.  
You should have an instance of Microsoft SQL Management Studio installed to execute the scripts.  

In order to be able to retrieve all the necessary information on the sizing and configuration of the database it is necessary to use an SQL Server administration account.  
To retrieve the information related to the ledger schema you can use the iGRC user account.

## Procedure  

### Architecture  

Performance issues are highly dependent on the architecture use on the platform presenting the issues. In order to help the support team to diagnose the situation it is necessary to fill out the excel file `BrainwavePerformanceAnalysisTemplate.xlsx` attached to this page and send it to the support team.  
The more information provided and the easier the diagnostic will be.

### SQL Database

This page includes attached, two scripts, that are provided by Identity Analytics :

- `CheckDB_SQLServer_Vxx.sql` - Provides All information on the database : parameters, sizing, statistics, indexes fragmentation and iGRC schema
- `SQLserver_iGRC_Schema.sql` - Provides only information on the iGRC schema as number of records per tables, per Time Slots and so on  

> To execute the first script an **administration** account **is required** (the system tables are queried).  
> The second script can be executed simply by igrc account only

Open either script in SQL Management Studio and before running it. It is necessary to indicate the name of the database to be queried on the following lines:  

| Line Number | Command                                |
| :---------- | :------------------------------------- |
| 12          | `use <database_name>;`                 |
| 18          | `set @dbname = '<database_name>';`     |
| 19          | `set @dbschema = '<database_schema>';` |

Save the results to an output file and send all to the Radiant Logic support team.

## Downloads  

All files are available for download in the following links:

- [BrainwavePerformanceAnalysisTemplate.xlsx](https://download.brainwavegrc.com/index.php/s/YMJY53CFXTqgiF7)
- [CheckDB_SQLServer_V15.sql](./assets/CheckDB_SQLServer_v15.sql)
- [CheckDB_SQLServer_V16.sql](./assets/CheckDB_SQLServer_v16.sql)
- [SQLServer_iGRC_Schema.sql](https://download.brainwavegrc.com/index.php/s/kY4CfAQsR9gS3QA)
