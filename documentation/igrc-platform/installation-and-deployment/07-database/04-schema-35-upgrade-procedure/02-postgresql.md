---
title: "Schema 35 upgrade procedure"
description: "Schema 35 upgrade procedure"
---

# PostgreSQL Schema 35 upgrade

## Pre-requisits

Before atempting any upgrade of the database please go through all steps detailed in the page below.

[**Release notes**](./index)

## Warning

This upgrade modifies the type of all recorduids and related foreign keys in the database from `INT` to `BIGINT`.
This operation implies in PostgreSQL the use of the command `Alter Table` to update the column. In PostgreSQL the `UPDATE` operation corresponds to an `INSERT` followed by a `DELETE` which requires a significant amount of free storage space on the database server.

As guide the minimal required free storage space on the database server corresponds to 2 times the size of the largest table in the database.  

For example: if the largest table size is 50 Gb it is recommended to have 100 Gb of free storage space on the database server.

In addition this operation is resource and time consuming.
Internal tests have shown that migrating a 170 Gb database took up to 5 hours on a 4vCPU and 16 Gb VM.

## Procedure

### Determining script

To determine the script to execute in order to update the database schema

```sql
SELECT cvalue AS "Current Schema version",
  CASE 
    WHEN cvalue = '35' THEN 'Schema version up to date' 
    WHEN cvalue = '32' THEN 'postgesql_BrailleR1_to_Curie.sql'
    WHEN cvalue = '31' THEN 'postgesql_AderR1_to_Curie.sql'
    WHEN cvalue = '30' THEN 'postgesql_2017R3_to_Curie.sql'
    WHEN cvalue = '29' THEN 'postgesql_2017R2_to_Curie.sql'
    ELSE 'Schema version not recognised. Please contact the support service' 
    END AS "Script to execute"
FROM     
    tproperties
WHERE
    cpropertiesuid='VERSION'
```

If required you can force the execution of the scripts within a given schema by adding the following

```sql
SET search_path = '<schema>' ;
```

### Download the Script

Depending on the version of the script displayed after the previous SQL request please download the correct script from the following list:  

|                                        Script                                         |
| :-----------------------------------------------------------------------------------: |
|   [postgresql\_2017R2\_to\_Curie.sql](./sqlscripts/postgresql_2017R2_to_Curie.sql)    |
|   [postgresql\_2017R3\_to\_Curie.sql](./sqlscripts/postgresql_2017R3_to_Curie.sql)    |
|   [postgresql\_AderR1\_to\_Curie.sql](./sqlscripts/postgresql_AderR1_to_Curie.sql)    |
| [postgresql\_BrailleR1\_to_Curie.sql](./sqlscripts/postgresql_BrailleR1_to_Curie.sql) |

### Execute the script

> [!warning] Update and uncomment the following script block, lines 32 to 35, before running the script:  

```sql
SET search_path = '<schema>';                        --  !REPLACE schema_name
\set SchemaVariable '''<schema>'''                   --  !REPLACE SchemaVariable
```

For PostgreSQL the script must be executed via command line. Use the following command:

```sh
psql -U user -d database -a -f "<ABSOLUTE PATH TO FILE>/postgresql_XXXXX_to_Curie.sql" -W > /tmp/output.txt 2>&1
```

Where:

- `-U` user
- `-d` databasename
- `-p` portnumber
- `-a` echo all
- `-f` script file to execute
- `-W` force prompt password

See below for more information on the usage of `psql`:  
https://www.postgresql.org/docs/13/app-psql.html  

### Execute VACUUM FULL operation

VACUUM FULL releases the wasted space back to OS.  

Running VACUUM FULL can lead to an unexpected runtime and during this runtime the tables will be on exclusive lock the whole time.  

```psql
vacuum (full, analyse, verbose);  
```

## After CURIE R1 upgrade

### Checking

To ensure the CURIE R1 upgrade script has been executed successfully, the below SQL request can be launched on the database.  

- Check the global number of indexes (replace **\<schemaName>** by the name of the Ledger schema)

```sql
SELECT
    count(indexname)
FROM
    pg_indexes
WHERE
    schemaname = '<schemaName>'
```

This request should return **1196** if the CURIE R1 upgrade successfully worked.

- Check the number of indexes per table (replace **\<schemaName>** by the name of the Ledger schema)

```sql
SELECT
    count(indexname) nb_index,tablename as name
FROM
    pg_indexes
WHERE
    schemaname = '<schemaName>'
GROUP BY name
ORDER BY name ASC;
```

In CURIE R1, and without project indexes, this request should return [this result](./assets/bw_postgresql_CURIE_R1_indexes_number_per_table_aggregation.csv) if the CURIE R1 upgrade successfully worked.
