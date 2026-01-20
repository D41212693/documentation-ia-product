---
title: "SQL Server configuration recommendations"
description: "SQL Server configuration recommendations"
---

# SQL Server configuration recommendations

The recommendations provided in the following document were established by performing tests in a integration environment using a volume of data representative of a production environment. These are the result of several data loading phases including cycles of testing, and tuning.

The overall data loading phase, as well the end user usage in the web portal (restitution phase) has been found to be optimal when respecting the following recommendations or best practices.

However, it will be up to the SQL Server DBA to adapt parameters according to the hardware and software environment on which the SQL Server engine is running as well as the physical location of data files, transaction logs. It is also recommended to adapt the configuration to production constraints.

## Recommendations

### Software environment

The following database repository and Operating systems are supported by Identity Analytics.

Only 64 bits versions are supported, Web and Express Editions are not supported.

The certified environment matrix is available on the Identity Analytics documentation site [here](../../igrc-platform/installation-and-deployment/04-brainwave-grc-certified-environments)

### Hardware requirements

A minimum of 8Gb of RAM is required, however it is recommended to allocate between 16Gb and 24Gb depending on the number of imported concepts (identities, accounts, permissions,...).

The quantity of loaded data that have an impact on required storage space of the database. The usage in the portal, i.e. the number of simultaneously connected users will impact the require simultaneous connections to the database.

- 8Go RAM minimum are required, 16Go to 32 Gb
- 4 CPU allocated (or 4 vCPU on an virtual environment as Hyper-V or VMware solutions)

### SQL Server prerequisites

#### Disk space requirements

The target volume depends on the number of objects collected (identities, accounts, groups, permissions, applications,...), the frequency of loading these data, as well as the retention period of the different Time Slots.

Radiant Logic recommends a volume of **10 Gb per Time Slot** (excluding the sizing of TEMPDB and Transaction logs files).

Assuming a retention period of 12 months and a data load every two weeks, then the required storage space would amount to 240 Gb at the end of a year.

Please note that there is relatively little variation in size between Time Slots, considering that functional scope and data is relatively constant during.

However, be aware that the final storage used for a timeslot is not representative of the maximum storage space used during the data loading phase. As a rule of thumb, if the end storage of a timeslot is 10 Gb then the maximum storage space reached during the data loading phase is then approximately 15 Gb.

#### SQL Server configuration

It is recommended to use a dedicated SQL Server engine for the Identity Analytics solution. During the data loading phase the database is greatly used and as such could impact the performance of the other hosted applications.  

The following configuration is recommended:

**Maximum Server Memory**

> [!warning] **Do not** set this value to the Total Physical Memory of your system/environment. Reserve 4 Gb to the OS itself.

It is recommended to configure between 8 Gb to 24 Gb depending on the available configuration.

All other parameters system parameters can remain as the default values. Some of them are set at installation according to the configuration of the current platform.

**Maximum Degree of Parallelism (MAXDOP)**

To ensure stable CPU utilization during data loading and analytics processing, it is recommended to set the Maximum Degree of Parallelism (MAXDOP) to 4.
You may need to adjust this value based on your system configuration and performance requirements.

#### The TEMPDB database

The **TEMPDB** database is a system database is used as a global resource that is available to all users connected to the instance of SQL Server or connected to SQL Database.

This database is used to hold:

- **Temporary user objects** (as global or local temporary tables and indexes)
- **Internal objects** that are created by the database engine. (Work tables, intermediate sort results for operations such as creating or rebuilding indexes, or Group by, order by, or union queries.
- **Version stores**, which are a collection of data pages that hold the data rows that are required to support the features that use row versioning.

Operations within **TEMPDB** are minimally logged so that transaction
can be rolled back is necessary.

> The **TEMPDB** is re-created every time SQL Server is started. The system always starts with a clean copy of the database.

The following good practices to apply to the TEMPDB should be kept in mind:

- Pre-allocate space for all TEMPDB files by setting the file size to a value large enough to accommodate the typical workload in the environment. Preallocation prevents TEMPDB from expanding too frequently, which affects performance.
- Data files should be of equal size within each [filegroup](https://docs.microsoft.com/en-us/sql/relational-databases/databases/database-files-and-filegroups?view=sql-server-ver15#filegroups), as SQL Server uses a proportional-fill algorithm that favors allocations in files with more free space. Dividing TEMPDB into multiple data files of equal size provides a high degree of parallel efficiency in operations that use TEMPDB.
- **Set the file growth** increment to a reasonable size to avoid the TEMPDB database files from growing by too small a value. If the file growth is too small, compared to the amount of data that is being written to TEMPDB, then TEMPDB may have to constantly expand affecting performance.
- Put the TEMPDB database on a fast I/O subsystem and on disks that differ from those that are used by user databases.

In practice, multiply the highest used space of TEMPDB PRIMARY filegroup (all files included) by 5, then divide that number by the total number of cores on the server to obtain the size of each file.

To get the highest value of TEMPDB, you need to monitor the system during at least 2 or 3 execution plan (Identity Analytics) because this value can change.

For example, if the highest value of used TEMPDB space in a day is 2Gb on a 4 core server then you need about 10Gb of total space . If you have 4 cores then create 4 files of 2.5Gb each and 1 file of 10 Mb.

The 4 files should be configured to NOAUTOGROWTH and the last file to
AUTOGROWTH. This serves as a safety net in case of the first 4 are filled.

A good start value of the size fo the TEMPDB is 8 Gb.  

Considering the system uses a 8 CPUs, then the TEMPDB should include 8 files of 1 Gb with no growth option configured and an additional 10Mb file configured in "AUTOGROWTH" with an auto-extend of 1024 Mb.

#### TRANSACTION LOGS

Every SQL Server database has a transaction log that records all transactions as well as the database modifications made by each transaction.

The transaction log is a critical component of the database. If there is a system failure, you will need that log to bring your database back to a healthy state.

The management of transaction logs depends on the recovery model defined at the database level. See section  [Recovery model](#recovery-model) for more information.

Having multiple log files in a database does not enhance performance in any way. The transaction log files do not use the [proportional fill](https://docs.microsoft.com/en-us/sql/relational-databases/pages-and-extents-architecture-guide?view=sql-server-ver15#ProportionalFill) feature as do the data files in a same filegroup.

Log files can be set to shrink automatically. However this is **not recommended**. Furthermore the `auto_shrink` database property is set to `FALSE` by default.

When setting **auto-growth** for data and log files using the FILEGROWTH option, it might be preferred to set it in **size** instead of **percentage**. This allows better control on the growth ratio, as percentage is an ever-growing amount. It is important that the file growth increment on a log file be sufficiently large to avoid frequent expansion.

A mechanism exists that allows the reuse of space within log files: Transaction log truncation after a checkpoint. However, several factors can delay log truncation, and a such monitoring log size evolution during the execution plan is important.

There is no optimal value for the initial size and auto-growth values of the transaction log file that will fit all situations. However, setting the initial size of the SQL Server Transaction Log file to **20-30%** of the database data file size and the auto-growth to a sufficiently large amount, above 1024MB, can be considered as a good starting point for the normal workload.

Another option is to preallocate the required space. Creating two transaction log files. A main, fixed size,  file and a failsafe file configured to allow "Auto-growth".

A good **start** value to recommended size for transaction log is: **25Gb**

The file can be created using  the following SQL script:

```sql  
LOG ON
( NAME = N'igrc_01_log', FILENAME = 'D:\LOG\igrc-01-log.ldf', SIZE = 12GB, FILEGROWTH = 0 ), -- NO AUTOGROWTH
( NAME = N'igrc_02_log', FILENAME = 'D:\LOG\igrc-02-log.ldf', SIZE = 12GB, FILEGROWTH = 1024MB, MAXSIZE = 30GB )
```

### iGRC Database configuration recommendations

#### Sizing datafiles

The configuration of the ledger database should follow the same best practices as those defined for the TEMPDB in previous section [Disk space requirements](#disk-space-requirements):

- Preallocate the required space for all datafiles by setting the file size to a value large enough to accommodate the typical workload in the environment. Preallocation prevents expanding too frequently, which affects performance.

- Data files should be of equal size within each [filegroup](https://docs.microsoft.com/en-us/sql/relational-databases/databases/database-files-and-filegroups?view=sql-server-ver15#filegroups), as SQL Server uses a proportional-fill algorithm that favors allocation to files with more free space. Dividing into multiple data files of equal size provides a high degree of parallel efficiency in operations that use datafiles.

- **Set the file growth** increment to a reasonable size to avoid the datafiles from growing repeatedly by too small a value.

As an example, On a Windows 10 computer using a Core i7, to create ad an estimated target database of 80 Gb you can use the following SQL script:

```sql  
USE master;
GO

-- Create the new database
CREATE DATABASE igrc
CONTAINMENT = NONE
ON PRIMARY
( NAME = N'igrc_01_dat', FILENAME = 'D:\DATA\igrc-01-dat.mdf',
SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_02_dat', FILENAME = 'D:\DATA\igrc-02-dat.mdf',
SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_03_dat', FILENAME = 'D:\DATA\igrc-03-dat.mdf',
SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_04_dat', FILENAME = 'D:\DATA\igrc-04-dat.mdf',
SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_05_dat', FILENAME = 'D:\DATA\igrc-05-dat.mdf',
SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_06_dat', FILENAME = 'D:\DATA\igrc-06-dat.mdf',
SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_07_dat', FILENAME = 'D:\DATA\igrc-07-dat.mdf',
SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_08_dat', FILENAME = 'D:\DATA\igrc-08-dat.mdf',
SIZE = 10GB, FILEGROWTH = 1GB )

LOG ON
( NAME = N'igrc_01_log', FILENAME = 'D:\LOG\igrc-01-log.ldf',
SIZE = 12GB, FILEGROWTH = 0), -- NO AUTOGROWTH
( NAME = N'igrc_02_log', FILENAME = 'D:\LOG\igrc-02-log.ldf',
SIZE = 12GB, FILEGROWTH = 1024MB, MAXSIZE = 30GB)

GO
```

#### Recovery Model

Radiant Logic recommends using the **Simple Recovery** model but perform a regular **FULL** backup of the database after each data load.

Nevertheless, following this recommendation is at the discretion of the SQL Server's DBA and as determined by the defined recovery policies.

To set the recovery model on the iGRC database, execute the following statement (FULL is the default value):

```sql  
-- Change the database recovery model
ALTER DATABASE <database name> SET RECOVERY SIMPLE ;
GO
```

#### Isolation level

Identity Analytics automatically sets the option `READ_COMMITTED_SNAPSHOT` of the `READ_COMMITTED` isolation level to "ON" during the initialization of the iGRC database.

When `READ_COMMITTED_SNAPSHOT` is set to ON (the default on SQL Azure Database), the Database Engine uses row versioning to present each statement with a transactionally consistent snapshot of the data as it existed at the start of the statement. Locks are not used to protect the data from updates by other transactions.

```sql  
ALTER DATABASE <database name> SET READ_COMMITED_SNAPSHOT ON WIH ROLLBACK IMMEDIATE;

GO
```

#### Login, Schema, Users and Permissions

When objects are created in the database without specifying which filegroup they belong to, they are assigned to the default filegroup. At any time, exactly one filegroup is designated as the default filegroup. The files in the default filegroup must be large enough to hold any new objects not allocated to other filegroups.

```sql  
IF NOT EXISTS (SELECT name FROM sys.filegroups WHERE is_default=1 AND name = N'PRIMARY') ALTER DATABASE <database name> MODIFY FILEGROUP [PRIMARY] DEFAULT

GO
```

The "**Login**" is the server level entity which is only used for
authentication.

```sql  
USE [master]

CREATE LOGIN [<Login name>] WITH PASSWORD=N'<Password>', DEFAULT_DATABASE=[<Ledger database name>]

GO
```

The "**Schema**" can be imagined as a container that holds objects in the SQL Database. The schema is owned by a user or a database wide role.

**User and permissions**: The user is used to assign database side permissions. The user is mapped to login. Permissions can be granted or revoked at the database level.

The required minimal permissions for the ledger database and workflow
(activiti) databases can be set as following:

For the ledger database user:  

```sql  
USE igrc; -- Use ledger database
GO

-- IGRC
-- Create the Schema
CREATE SCHEMA igrc AUTHORIZATION dbo;
GO

-- Create Users and schema
CREATE USER igrc FOR LOGIN igrc WITH DEFAULT_SCHEMA = igrc ;
GO

GRANT CREATE TABLE TO igrc ;
GRANT CREATE VIEW TO igrc ;
GRANT ALTER TO igrc ;
GRANT ALTER ON SCHEMA :: igrc TO igrc ;
GRANT INSERT ON SCHEMA :: igrc TO igrc ;
GRANT SELECT ON SCHEMA :: igrc TO igrc ;
GRANT DELETE ON SCHEMA :: igrc TO igrc ;
GRANT REFERENCES ON SCHEMA :: igrc TO igrc ;
GRANT UPDATE ON SCHEMA :: igrc TO igrc ;
GO
```

For the workflow (Activiti) database user:  

```sql  
-- ACTIVITI
-- Create the Schema
CREATE SCHEMA activiti AUTHORIZATION dbo;
GO

-- Create Users and schema
CREATE USER activiti FOR LOGIN activiti WITH DEFAULT_SCHEMA = activiti ;
GO

GRANT CREATE TABLE TO activiti ;
GRANT ALTER ON SCHEMA :: activiti TO activiti ;
GRANT REFERENCES ON SCHEMA :: activiti TO activiti ;
GRANT ALTER TO activiti ;
GRANT SELECT ON SCHEMA :: activiti to activiti ;
GRANT UPDATE ON SCHEMA :: activiti to activiti ;
GRANT INSERT ON SCHEMA :: activiti to activiti ;
GRANT DELETE ON SCHEMA :: activiti to activiti ;
GO
```

### Backup and indexes/statistics maintenance plan

Radiant Logic recommends a regular FULL backup of the database after each loading. And for this reason, the recovery model can be set to **Simple**.

In addition, an index maintenance plan (drop and recreate indexes or a
simple rebuild) must be scheduled regularly to maintain overall
performance and avoid the indexes fragmentation.

If selected in the execution plan tab of the technical configuration then the statistics and the indexes are updated during the Identity Analytics execution plan.

### Recovery model

The **Recovery Model** is one of the mechanisms which controls and manages the growth of the transaction log file. The Recovery Model controls how transactions are logged, whether there is automatic log truncation, whether the transaction log requires and/or allows backing up the transaction log, and what kind of restore operations are available.

SQL Server has a property called **Recovery Model**, which can have either **Simple**, **Bulk-logged** and **Full** values based on your different needs for performance, storage space, and protection against data loss.

The **Simple** recovery model is the simplest of all. It maintains only a minimum amount of information in the SQL Server transaction log file. SQL Server, on its own, truncates the transaction log files (excluding logs from any open transactions) and removes the information related to transactions which have reached transaction checkpoints (when the data has been written to the data file) so that the space can be reused, leaving no transaction log entries for disaster recovery purposes.

With the **Simple** recovery model, the data is recoverable only to the most recent full database or differential backups. No transaction log backups are supported. Under the **Simple** recovery model, transaction log truncation happens after a checkpoint or as soon as you change the recovery model of your database to Simple recovery model.

## Appendix

### General information

This appendix provides the machine configurations used in our validation environment. This is provided only for information purposes.

In addition a sample script file is provided containing all the recommendations previously defined in this document.

### Configuration validated

#### Principal physical server

**PowerEdge R430 for Intel v4 CPUs**

- Processor : Intel(R) Xeon(R) CPU E5-2660 v4 @ 2.00GHz
- Disks:
  - 6 TB 7.2K RPM SATA 6Gbps (2 hard drives)
  - 300GB 15K RPM SAS 12 Gbps (1 hard drive)
- PERC H330 RAID Controller
- Memory: 128 Gb
- Ethernet: 1Gb

#### Hyper-V configuration

A virtual machine hosting SQL Server engine and Database and a separate virtual machine hosting Identity Analytics solution.

**VM configuration - Database**

- Windows 10 (64bits)
- SQL Server 2017
- 4 vCPU
- Memory: 12Gb

Data and log files are distributed on different disk volumes.

**VM configuration - Identity Analytics application**

- Windows Server 2012 R2
- 4 vCPUS
- Mamory: 4Gb to 12Gb - Dynamic allocation
- iGRC batch (JVM) : 8Gb

### Script to create iGRC database and schema

**Context:**

- Platform: Windows 10 (64bits) / Core i7 / 16 Gb RAM
- database name: igrc
- Login & user for the ledger: igrc
- Ledger schema: ledger
- Login & user for the workflow: activiti
- Workflow schema: activiti
- Estimated database size: 80 Gb

```sql  
USE master;
GO

-- Create the new database
CREATE DATABASE igrc
CONTAINMENT = NONE
ON PRIMARY
( NAME = N'igrc_01_dat', FILENAME = 'D:\DATA\igrc-01-dat.mdf', SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_02_dat', FILENAME = 'D:\DATA\igrc-02-dat.mdf', SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_03_dat', FILENAME = 'D:\DATA\igrc-03-dat.mdf', SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_04_dat', FILENAME = 'D:\DATA\igrc-04-dat.mdf', SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_05_dat', FILENAME = 'D:\DATA\igrc-05-dat.mdf', SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_06_dat', FILENAME = 'D:\DATA\igrc-06-dat.mdf', SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_07_dat', FILENAME = 'D:\DATA\igrc-07-dat.mdf', SIZE = 10GB, FILEGROWTH = 1GB ),
( NAME = N'igrc_08_dat', FILENAME = 'D:\DATA\igrc-08-dat.mdf', SIZE = 10GB, FILEGROWTH = 1GB )
LOG ON
( NAME = N'igrc_01_log', FILENAME = 'D:\LOG\igrc-01-log.ldf', SIZE = 12GB, FILEGROWTH = 0), -- NO AUTOGROWTH
( NAME = N'igrc_02_log', FILENAME = 'D:\LOG\igrc-02-log.ldf', SIZE = 12GB, FILEGROWTH = 1024MB, MAXSIZE = 30GB)
GO

-- Set Default filegroup 
IF NOT EXISTS (SELECT name FROM sys.filegroups WHERE is_default=1 AND name = N'PRIMARY') ALTER DATABASE igrc MODIFY FILEGROUP [PRIMARY] DEFAULT
GO  

-- Change the database recovery model 
ALTER DATABASE igrc SET RECOVERY SIMPLE ;
GO

-- Set option for isolation level 
ALTER DATABASE igrc SET READ_COMMITED_SNAPSHOT ON WIH ROLLBACK IMMEDIATE;
GO

-- Create ledger Login
CREATE LOGIN igrc WITH PASSWORD=N'<Passwordtochange>', DEFAULT_DATABASE=[igrc]

-- Optional : Create activiti Login
CREATE LOGIN activiti WITH PASSWORD=N'<Passwordtochange>', DEFAULT_DATABASE=[igrc]

-- Create schema for Ledger 
USE igrc;   -- Use ledger database
GO
-- LEDGER IGRC
-- Create the Schema
CREATE SCHEMA igrc AUTHORIZATION dbo;
GO
-- Create Users and schema
CREATE USER igrc FOR LOGIN igrc WITH DEFAULT_SCHEMA = igrc ;
GO
GRANT CREATE TABLE TO igrc ;
GRANT CREATE VIEW TO igrc ;
GRANT ALTER TO igrc ;
GRANT ALTER ON SCHEMA :: igrc TO igrc ;
GRANT INSERT ON SCHEMA :: igrc TO igrc ;
GRANT SELECT ON SCHEMA :: igrc TO igrc ;
GRANT DELETE ON SCHEMA :: igrc TO igrc ;
GRANT REFERENCES ON SCHEMA :: igrc TO igrc ;
GRANT UPDATE ON SCHEMA :: igrc TO igrc ;
GO
 
-- OPTIONAL :  ACTIVITI
-- Create the Schema
CREATE SCHEMA activiti AUTHORIZATION dbo;
GO
-- Create Users and schema
CREATE USER activiti FOR LOGIN activiti WITH DEFAULT_SCHEMA = activiti ;
GO
GRANT CREATE TABLE TO activiti ;
GRANT ALTER ON SCHEMA :: activiti TO activiti ;
GRANT REFERENCES ON SCHEMA :: activiti TO activiti ;
GRANT ALTER TO activiti ;
GRANT SELECT ON SCHEMA :: activiti to activiti ;
GRANT UPDATE ON SCHEMA :: activiti to activiti ;
GRANT INSERT ON SCHEMA :: activiti to activiti ;
GRANT DELETE ON SCHEMA :: activiti to activiti ;
GO 
```
