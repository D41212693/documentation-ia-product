---
title: "Performance investigation in oracle"
description: "How-To help the support team to investigate performance issues when using an Oracle database"
---

# Performance investigation in oracle

In order to efficiently investigate performance issues, some information on Identity Analytics's infrastructure and configuration is required, such as :  

- The list of parameters
- The sizing of the database (Oracle SGA and tablespaces)
- The index fragmentation
- And so on

This article includes a SQL script, and it's associated documentation. The results should be sent to Radiant Logic support service to aide in the resolution of performance issues.

## Prerequisites

The scripts have been tested on the Oracle database versions 10,11g or 12c.  
An instance of SQLPlus must be used to perform the data extraction.
An Oracle system account to retrieve all information as all parameters of database or sizing of this one.

## Procedure

The attached file include all the necessary scripts provided by Identity Analytics :

- [DBCheck_Oracle_V5.zip](./assets/DBcheck_Oracle_v5.zip)

In order to run the provided SQL scripts it is necessary to :

- Copy and decompress the “DBCheck_Vxx.zip” file into a folder while ensuring that it can be accessed by the oracle account. (On UNIX, `chmod –R 777 <folder>`)
- execute the script `db_check.sql` in a SQLPlus session and using the oracle system account, .

> If your Oracle Schema is not "IGRC", please modify the `env.sql` file to specify the `USERNAME` parameter before to execute the db\_check script file

The following commands allow you to run the script and output the results to the `/tmp/results_db_check.txt` file:  

```sh
Sqlplus system/<password>@<TNS Connect String>  
@db_check.sql /tmp/results_db_check.txt ;  
Exit;
```

- Send the resulting output file to Radiant Logic's support team.
