---
title: "Clean temporary IAP metadata"
description: "Clean temporary IAP metadata"
---

Versions prior to IAP 1.2 included the use of temporary metadata data. These technical metadata included certain computations that were later aggregated into a final version. The final version being queried in the webportal pages.  

An enhancement in the metadata engine now allows the user to flag these temporary metadata so that they are automatically deleted once the execution plan is finished in order to minimize the size of the metadata tables and save storage space, improving performances.
In order to optimize storage and thus free up the disk space occupied by the pre-existing IAP temporary metadata an SQL script to delete these unused temporary metadata in the database history.

> [!warning] This operation must be executed only:
>
> - If migrating a database where the project included is a version of IAP prior to 1.2
> - After the technical migration to Curie R2 (Schema version 37).

The included sub-pages will provide the user with the procedure to execute in order to clean temporary metadata

# Prerequisites

> [!warning] Require the version 37 of the database schema.

Before attempting the operation on the database it is recommended to:  

- Turn off **all** services that connect to the database. This includes:
  - Closing all opened Studios
  - Shutdown the webportal
  - If relevant, deactivate all scheduled tasks (batch commands, and database maintenance operations)
- Backup up the current database, including both the ledger and the activiti (workflow) schema
- The restore procedure of the database has been tested and validated

# Scripts

The clean temporary metadata scripts are provided for all supported databases:  

- SQL server
- PostgreSQL
- Oracle

## Download the Script

Please download the correct script from the following list according to your requirements:  

|                               Script                                |
| :-----------------------------------------------------------------: |
|     [oracle](./sqlscript/orcl_clean_IAP_temporary_metadata.sql)     |
|    [postgresql](./sqlscript/pg_clean_IAP_temporary_metadata.sql)    |
| [sqlserver](./sqlscript/sqlserver_clean_IAP_temporary_metadata.sql) |

# Warning

Depending on the database engine (SQLServer, Oracle, PostgreSQL) and the size of the database this operation, which executes many delete requests, can be time consuming and require a large amount of free disk space.

Internal tests have shown the following results on metadata tables containing about 10 000 000 records

- **SQLServer**
  - Duration : 1h16 minute
  - Space reserved for transaction logs from 9 GB to 73 GB.
- **PostgreSQL**
  - Duration: 3 minutes
  - Disk space recovered after a full vacuum (from 23 GB to 12,5 GB)
- **Oracle**
  - Duration: 22 minutes
  - UNDO tablespace : 15 GB  

On PostgreSQL a vacuum full operation is necessary at the end of the migration to retrieve the storage.  

A shrink on datafiles can be done for the others engine.  
