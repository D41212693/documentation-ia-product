# Export an Oracle database

## Goal

Migrate a Oracle database to PostgreSQL using ora2pg, an open source tool (see [licensing](#licenses)). The tool can export most Oracle objects into SQL code compatible for ingestion in PostgreSQL.

## Test Environnement

Oracle and PostgreSQL databases on the same host (Windows 11 (64bits)), 32 GB RAM and Core i7

- Target database : PostgreSQL 17.2
- Oracle 19c Standard Edition
- Descartes R5 SP4

## Prerequisites

> Please note that the ledger database schema to migrate is Descartes R5 (schema version 40).  
> It is **mandatory** to migrate Identity Analytics to the latest version of the product, Descartes R5, before executing the migration procedure.  

The following are required to execute the procedure:

- The latest version of ora2pg 24.3 see [here](https://github.com/darold/ora2pg/releases/tag/v24.3)
- Oracle Instant Client or Full Oracle installation
- The Contents of `support_tools\MigrateDBtoPG` folder (On bitbucket - Project `Support_tools`)
- An installation of Perl version 5.32 or above including for ORACLE the module `DBD::Oracle`

You'll also need:

- an ORCL account with the permission to SELECT from the tables you want to migrate.
- The connection parameters to the source database: IP address, port, username & password, database name
- An empty PostgreSQL database corresponding to the target database. The target user and schema must be created before migration.

> **Warning:** Only ledger schema of Identity Analytics will be migrated. This procedure must be launched only if there are no reviews in progress.  
> The Activiti schema used by the workflow engine will **not** be migrated.

## Licenses

See: https://ora2pg.darold.net/license.html

## Installation and configuration

### Oracle client

See Oracle documentation for this installation.

### Perl Installation

Install Perl on the host targeted to run ora2pg. See [here](https://www.perl.org/) for more information

Perls `DBD::Oracle` module is required to allow connection to the source database.

To install the module:

- Set the variable ORACLE_HOME accordingly. When using powershell for example:

```sh
$env:ORACLE_HOME="C:\tools\ORCL\db_home"
```

- Install the module using CPAN:

```sh
perl -MCPAN -e 'install DBD::Oracle'
```

To check the installation of the module please perform the following command:

```sh
perl -e 'use DBI; use DBD::Oracle; DBI->installed_versions();'
```

The output should display the version of `DBD::Oracle` as shown in the example below.

```plaintext
PS C:\Users> perl -e 'use DBI; use DBD::Oracle; DBI->installed_versions();'
  Perl            : 5.040000    (MSWin32-x64-multi-thread)
  OS              : MSWin32     (10.0.22621.3880)
  DBI             : 1.643
  DBD::Sponge     : 12.010003
  DBD::SQLite     : 1.74
  DBD::Proxy      : install_driver(Proxy) failed: Can't locate RPC/PlClient.pm in @INC (you may need to install the RPC::PlClient module)
  DBD::Pg         : 3.8.0
  DBD::Oracle     : 1.90
  DBD::ODBC       : 1.61
  DBD::Mem        : 0.001
  DBD::Gofer      : 0.015327
  DBD::File       : 0.44
  DBD::ExampleP   : 12.014311
  DBD::DBM        : 0.08
  DBD::CSV        : 0.60
  DBD::ADO        : 2.99
```

### Ora2pg installation

Download the archive containing the latest version of ora2pg from [here](https://github.com/darold/ora2pg/releases/tag/v24.3).

Before installation set the requires environment variables for your `ORACLE_HOME`, the Oracle client software location and `LD_LIBRARY_PATH` the lib folder in the Oracle client software location. For example:

```powershell
$env:ORACLE_HOME="C:\tools\ORCL\db_home"
$env:LD_LIBRARY_PATH="C:\tools\ORCL\db_home\lib"
```

To install ora2pg:

- Unzip the archive in a dedicated folder, e.g. `C:\tools\ora2pg\ora2pg-24.3`
- In a PowerShell terminal navigate to the above folder and execute:

```powershell
perl .\Makefile.PL
```

- Then execute:

```powershell
gmake 
gmake install
```

To test the installation of Ora2pg run the following command:

```powershell
ora2pg --version
```

> When using Perl version 5.40 some warnings are displayed. To fix it, open files mentionned in warning and replace `use File::Spec` with `use File::Spec::Functions`. Warning : By default, the files are in Read Only mode.

Once installed it is recommended to create a dedicated workspace for the migration. Execute the following command, adapted to your context, to do so:

```powershell
ora2pg --init_project orcl --project_base c:\tools\ora2pg
```

This will generate a workspace with all tools necessary for the migration, including a generic `ora2pg.conf` configuration file in the `/config` subdirectory that should be updated for the procedure.

### Ora2pg configuration

Update the ora2pg configuration file `ora2pg.conf` created in the `/config` subfolder of your workspace. The following parameters, in code blocs, must be updated:

```sh
ORACLE_DSN  dbi:Oracle:host=localhost;sid=orcl;port=1521
ORACLE_USER SCHEMA40
ORACLE_PWD  xxxx
```

In the above example:

- `ORACLE_DSN` corresponds to the Oracle connection string, with:
  - `host=localhost` Hostname of the Oracle database source server
  - `sid=SCHEMA40` Source database SID
  - `port=1521` Source database port
- `ORACLE_USER` is the ORCL user for authentication
- `ORACLE_PWD` is the ORACLE password for authentication

```sh
SCHEMA IGRC
```

Where `ÌGRC` is the schema containing the ledger tables to migration.  
If this is unknown, you can execute the command `ora2pg -t SHOW_SCHEMA -c .\config\ora2pg.conf` to list all existing schema in the source database


```sh
OUTPUT output.sql
OUTPUT_DIR  C:\temp\ora2pg\migration\data
```

Where OUTPUT is the name of the file to execute the data migration and OUTPUT_DIR is the folder in which all migration scripts will be exported. This folder must exist. And correspond to the subfolder created previously in executing the `ora2pg --init_project`.

```sh
DEBUG 1
```

It is recommended to activate debug mode to have all error messages if any.

```sh
TRANSACTION  readonly
```

To force all the execution of all transactions to readonly.

```sh
FILE_PER_TABLE 1
```

To allow the generation of an output file for each table.

```sh
DISABLE_SEQUENCE  1
TRUNCATE_TABLE    1
```

Where `DISABLE_SEQUENCE` Disables the reinitialization of sequences after import. Will be done at the end of the procedure, and `TRUNCATE_TABLE` adds truncate table if exists orders before import

```sh
DATA_LIMIT 5000
```

Where `DATA_LIMIT` determines the number of tuples exported at once in bulk. If you set a high value be sure to have enough memory if you have million of rows.

```sh
# SELECT_TOP 1000
```

Comment out the line `SELECT_TOP` to allow export of All data. This parameter can be used to test out the procedure on a subset of data.

To tests ora2pg's operation you can execute the following commands, where `.\config\ora2pg.conf` is the full path to the configuration file to use.

- list all schemas

```sh
ora2pg -t SHOW_SCHEMA -c .\config\ora2pg.conf
```

- To list all tables and the number of records per table

```sh
ora2pg -t SHOW_TABLE -c .\config\ora2pg.conf
```

- To show the encoding used by the target database:

```sh
ora2pg -t SHOW_ENCODING -c .\config\ora2pg.conf
```

## Data Migration procedure

### Data Export

To export all data contained in the source database tables, execute the following command:

```sh
ora2pg -t COPY -c .\config\ora2pg.conf
```

Where -c Path to the configuration file to use.

This generates 305 files. An `output.sql` which will be used to import all data and 304 files that corresponds to the data of the 304 tables included in the schema of Identity Analytics. The advantage of having data multiple files available is that they can be reloaded manually several times in the event of a problem, until the right fix is found.

> [!warning] It is necessary to create an archive (`tar` `gzip` format) of all exported data files at root, no sub directory containing the files. This archive will be used when importing the database see [here](./import-database)
