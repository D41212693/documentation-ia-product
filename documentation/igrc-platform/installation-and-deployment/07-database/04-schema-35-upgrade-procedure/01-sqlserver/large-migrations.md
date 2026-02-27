---
title: "Large SQL server migrations"
description: "Large SQL server migrations"
---

# Large SQL server migration

To help with the migration of large SQL server database the provided scripts were split into 4 different steps to allow the user to re-execute the required scripts if an error occurs.  

> [!warning] It is recommended to backup the database before performing the upgrade. This procedure does NOT replace an database backup and is provided as an aid in the migration process.  

> [!warning] The following scripts are delivered ONLY for the migration of Braille R1 to Curie.

## Download  

To determine the version of the scripts to download, mono or multilingual, please use the following command:

```sql
IF ( SELECT top 1 cpropertiesuid FROM tproperties WHERE cpropertiesuid = 'SQLSERVER_MULTILINGUAL') is not null
BEGIN
  PRINT 'Download scripts for "Multilingual - mssql"'
END
ELSE
BEGIN
  PRINT 'Download scripts for "Monolingual - mssql noml"'
END
```

The scripts to download are available for download only for Braille and SQL server in the corresponding tables below:

|       Multilingual - mssql       |
| :------------------------------: |
| [STEP1.sql](STEP1.sql) |
| [STEP2.sql](STEP2.sql) |
| [STEP4.sql](STEP4.sql) |

|          Monolingual - mssql noml           |
| :-----------------------------------------: |
| [STEP1\_noml.sql](STEP1_noml.sql) |
| [STEP2\_noml.sql](STEP2_noml.sql) |
| [STEP4\_noml.sql](STEP4_noml.sql) |

## Upgrade procedure

> [!warning] As a reminder it is recommended to backup the database before performing the upgrade. This procedure does NOT replace an database backup and is provided as an aid in the migration process.  

### Upgrade method

#### Powershell

If the upgrade is done using powershell execute the following command replacing the `<FullPathToUpgradeScript>` by the full path of the sql script you wish to execute.  

```powershell
$key=Read-Host -AsSecureString
Invoke-Sqlcmd -InputFile "<FullPathToUpgradeScript>" -ServerInstance "<ServerInstance>" -Database "<Database>" -Username "<USER>" -Password $key -verbose 
```

#### SQL server management studio  

When executing the desired opened script in SQL server management studio please ensure that the correct database and login are used.  

This is done by uncommenting the following block in the script:

```sql
USE "<database>"
EXECUTE AS LOGIN='<USER>'
```

Where `<DATABASE>` is the database and `<USER>` is the user with which to execute the request.  

### Step 1

Execute the script `STEP1_XXXX_noml_BrailleR1_to_schema_33.sql` to upgrade the schema version from version 32 to version 33

### Step 2

Execute the script `STEP2_XXXX_PRINT_int_to_bigint_migration.sql` to print to the output all the commands.  
It is recommended to output the results to a file `.sql` file that can be used as an import.

> If executing the command in powershell remember to output the verbose stream only to the file using `4>&1 > Step3_queries.sql` at the end of the command to output the correct stream.  
>
> ```powershell
> $key=Read-Host -AsSecureString
> Invoke-Sqlcmd -InputFile "<FullPathToUpgradeScript>" -ServerInstance "<ServerInstance>" -Database "<Database>" -Username "<USER>" -Password $key -Verbose 4>&1 > Step3_queries.sql
> ```

> If executing the script in SSMS you'll have to clean the output to remove the lines of the format displayed below:  
>
> ```sql
> (XXX rows affected)
> Completion time: 2023-07-13T18:11:06.5681281+02:00
> ```

### Step 3

Execute the script resulting from [Step 2](#step-2), in the case above `Step3_queries.sql`.  
The queries have been configured such that if an error occurs during this step, after the Analyses of the root cause of the issue, it is only necessary to re-execute the same script until it finishes correctly.  

### Step 4  

Execute the final script `STEP4_XXXXX_schema_33_to_schema_35.sql` to finalize the migration to teh version 35 of the schema.  

### Validation  

To ensure the CURIE R1 upgrade script has been executed successfully, two below SQL requests can be launched on the database.  

> Both requests must be executed with **sysadmin** rights on the Identity Analytics Ledger database.

- Check if all needed columns has been moved from **INT** to **BIGINT** type

```sql
SELECT (T.name) AS Table_Name, schema_name(t.schema_id), C.name AS Column_Name, c.is_nullable
FROM   sys.objects AS T 
  JOIN sys.columns AS C ON T.object_id = C.object_id
  JOIN sys.types AS P ON C.system_type_id = P.system_type_id
WHERE
  T.type_desc = 'USER_TABLE'
  AND P.name = 'int'
  AND (C.name = 'crecorduid' 
    OR C.name like '%fk' 
    OR C.name in ('cforeignrecorduid','cobjectrecorduid','cobjectuid','coptionuid','crequestid','cworkrecuid')
  );
```

This request should return **0** result if the CURIE R1 upgrade successfully worked.

- Check the global number of indexes

```sql
SELECT 
     count(ind.name) nb_index
FROM 
     sys.indexes ind 
INNER JOIN 
     sys.tables t ON ind.object_id = t.object_id
```

This request should return **1192** if the CURIE R1 upgrade successfully worked.

- Check the number of indexes per table

```sql
SELECT 
     count(ind.name) nb_index,t.name
FROM 
     sys.indexes ind 
INNER JOIN 
     sys.tables t ON ind.object_id = t.object_id
GROUP BY 
     t.name
ORDER BY t.name ASC;
```

In CURIE R1, and without project indexes, this request should return [this result](.assets/bw_sqlserver_CURIE_R1_indexes_number_per_table_aggregation.csv) if the CURIE R1 upgrade successfully worked.  
