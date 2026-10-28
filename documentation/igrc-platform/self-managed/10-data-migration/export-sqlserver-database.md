# Export a SQL server database

> ora2pg was initially created to migrate an Oracle database to PostgreSQL before being extended to allow migrations of other DBMS flavors (e.g. MSSQL, MySQL ...). As a result, the variables used are labeled ORACLE even if this is not the target database to migrate.  

## Goal

Migrate a SQLServer database to PostgreSQL using ora2pg, an open source tool (see [licensing](#licenses)). The tool can export most MSSQL objects into SQL code compatible for ingestion in PostgreSQL.

## Test Environnement

SQLServer and PostgreSQL databases on the same host (Windows 11 (64bits)), 32 GB RAM and Core i7

- Target database: PostgreSQL 17.2
- SQLServer Standard Edition 2019 and 2022
- Descartes R5 SP4

## Prerequisites

> Please note that the ledger database schema to migrate is Descartes R5 (schema version 40).  
> It is **mandatory** to migrate Identity Analytics to the latest version of the product, Descartes R5, before executing the migration procedure.  

The following are required to execute the procedure:

- The latest version of ora2pg 24.3 see [here](https://github.com/darold/ora2pg/releases/tag/v24.3)
- A Microsoft SQLServer Standard or Enterprise Edition 2019 or above
- The Contents of `support_tools\MigrateDBtoPG` folder (On bitbucket - Project `Support_tools)
- An installation of Perl version 5.32 or above including for SQLServer the driver `DBD::ODBC`
- ODBC drivers for database connection: Microsoft ODBC Driver for SQLServer

You'll also need:

- An Microsoft SQLserver account with the permission to SELECT from the tables you want to migrate.
- The connection parameters to the source database: IP address, port, username & password, database name
- An empty PostgreSQL database corresponding to the target database. The target user and schema must be created before migration.

> **Warning:** Only ledger schema of Identity Analytics will be migrated. This procedure must be launched only if there are no reviews in progress.  
> The Activiti schema used by the workflow engine will **not** be migrated.

## Licenses

See: https://ora2pg.darold.net/license.html

## Installation and configuration

### Perl Installation

Install perl on the host targeted to run ora2pg. See [here](https://www.perl.org/) for more information

Perls `DBD::ODBC` module is required to allow connection to the source database. The module is available for download [here](https://metacpan.org/dist/DBD-ODBC). This is sometimes already installed with Perl.

To check the installation of the module please perform the following command:

```sh
perl -e 'use DBI; use DBD::ODBC; DBI->installed_versions();'
```

The output should display the version of `DBD::ODBC` as shown in the example below.

```plaintext
PS C:\Users> perl -e 'use DBI; use DBD::ODBC; DBI->installed_versions();'
  Perl            : 5.040000    (MSWin32-x64-multi-thread)
  OS              : MSWin32     (10.0.22621.3880)
  DBI             : 1.643
  DBD::Sponge     : 12.010003
  DBD::SQLite     : 1.74
  DBD::Proxy      : install_driver(Proxy) failed: Can't locate RPC/PlClient.pm in @INC (you may need to install the RPC::PlClient module)
  DBD::Pg         : 3.8.0
  DBD::ODBC       : 1.61
  DBD::Mem        : 0.001
  DBD::Gofer      : 0.015327
  DBD::File       : 0.44
  DBD::ExampleP   : 12.014311
  DBD::DBM        : 0.08
  DBD::CSV        : 0.60
  DBD::ADO        : 2.99
```

If not installed the following command can be used:

```sh
perl -MCPAN -e 'install DBD::ODBC'
```

### ODBC installation and configuration

Install and configure Microsoft ODBC Driver 18 for SQLServer. See [here](https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16) for more information.

Once installed run the tool ODBC data sources to configure a new connection.

- Click "Add" to add new connection
- In the list select "ODBC driver 18 for SQL Server"
- Provide a name, _e.g._ `MSSQLODBC` in this example, a description as well as the hostname of the server running the Microsoft SQL server target database. The name will used in the DSN connection string and click next
- Select the option to connect using SQL Server authentication using login and password entered by the user and provide the desired credentials. Click next
- Is is recommended to change the default database to the target database before Clicking next
- Change the Connection encryption to optional and click finish

### Ora2pg installation

Download the archive containing the latest version of ora2pg from [here](https://ora2pg.darold.net/).

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
ora2pg --init_project mssql --project_base c:\tools\ora2pg
```

This will generate a workspace with all tools necessary for the migration, including a generic `ora2pg.conf` configuration file in the `/config` subdirectory that should be updated for the procedure.

### Ora2pg configuration

Update the ora2pg configuration file `ora2pg.conf` created in the `/congfig` subfolder of your workspace. The following parameters, in code blocs, must be updated:

```sh
ORACLE_DSN  dbi:ODBC:dsn=MSSQLODBC;server=localhost;database=migration;TrustServerCertificate=yes
ORACLE_USER igrc
ORACLE_PWD  xxxx
```

In the above example:

- `ORACLE_DSN` corresponds to the OBDC connection string, with:
  - `dsn=MSSQLODBC` the name of the configured ODBC datasource above
  - `server=localhost` the hostname of the server hosting the SQL Server database
  - `database=migration` the name of the target database to migrate
- `ORACLE_USER` is the MSSQL user for authentication
- `ORACLE_PWD` is the MSSQL password for authentication

```sh
SCHEMA ledger
```

Where ledger is the schema containing the ledger tables to migration.  
If this is unknown, you can execute the command `ora2pg -t SHOW_SCHEMA -c .\config\ora2pg.conf` to list all existing schema in the source database

```sh
OUTPUT output.sql
OUTPUT_DIR  C:\temp\ora2pg\migration\data
```

Where OUTPUT is the name of the file to generate to execute the data migration and OUTPUT_DIR is the folder in which all migration scripts will be exported.

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

To tests ora2pg's operation you can execute the following commands, where `C:\tools\ora2pg\mssql\config\ora2pg.conf` is the full path to the configuration file to use.

- list all schemas

```sh
ora2pg -M -t SHOW_SCHEMA -c C:\tools\ora2pg\mssql\config\ora2pg.conf
```

- To list all tables and the number of records per table

```sh
ora2pg -M -t SHOW_TABLE -c C:\tools\ora2pg\mssql\config\ora2pg.conf
```

- To show the encoding used by the target database:

```sh
ora2pg -M -t SHOW_ENCODING -c C:\tools\ora2pg\mssql\config\ora2pg.conf
```

## Data Migration procedure

### Data Export

To export all data contained in the source database tables, execute the following command:

```sh
ora2pg -M -t COPY -c C:\tools\ora2pg\mssql\config\ora2pg.conf
```

Where -c Path to the configuration file to use.

This generates 305 files. An output.sql which will be used to import all data and 304 files that corresponds to the data of the 304 tables included in the schema of Identity Analytics. The advantage of having data multiple files available is that they can be reloaded manually several times in the event of a problem, until the right fix is found.

> [!warning] It is necessary to create an archive (`tar` `gzip` format) of all exported data files at root, no sub directory containing the files. This archive will be used when importing the database see [here](./import-database)
