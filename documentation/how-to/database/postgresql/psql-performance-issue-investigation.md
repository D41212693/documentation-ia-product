---
title: "Performance investigation in postgresql"
description: "How-To help the support team to investigate performance issues when using an PostgreSQL database"
parent: "PostgreSQL how-to's"
grand_parent: "Database how-to's"
toc: true
---

# Performance investigation in postgresql  

In order to efficiently investigate performance issues, some information on Identity Analytics's infrastructure and configuration is required, such as :  

- The list of parameters
- The sizing of the database
- The sizing of objects (Table, Index)
- And so on

This article includes a SQL script, as  well as it's the associated documentation. The results should be sent to Radiant Logic support service to aide in the resolution of performance issues.  

## Prerequisites  

The scripts have been tested on the PostgreSQL database versions 9.6.  
An instance of PSQL must be used to perform the data extraction.  
An Postgres system account to retrieve all information as all parameters of database or sizing of this one.

## Procedure

In order to run the provided SQL scripts it is necessary to :  

- Copy the `DBCheck_PostgreSQL_Vxx.sql` file into a folder while ensuring that it can be accessed by the PostgreSQL account. (On Linux, `chmod –R 777 <folder>`)
- Execute the script `DBCheck_PostgreSQL_Vxx.sql` within a PSQL session and using the `postgres` system account.

> If your database schema is not "igrc", please modify the script file accordingly. Replace "igrc" by the name of your schema.

The following commands allow you to run the script and output the results to the `/tmp/results_db_check.txt` file:  

```bash
psql -h hostname -p 5432 -d database_name -U postgres -f <absolute path to file>/DBcheck_postgreSQL_Vxx.sql > /tmp/results_db_check.txt
```

Please send the resulting output file to Radiant Logic's support team.

## Long queries

In addition it can be interesting to list the long queries being executed in the database. To do so please execute the following sql request:  

```sql
-- List long queries on PostgreSQL
SELECT
  pid,
  pg_stat_activity.query_start,
  now() - pg_stat_activity.query_start AS duration,
  query,
  state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes' and state = 'active';
```

## Downloads

All scripts are available for download at the following links:

- [DBCheck_PostgreSQL_V10.sql](/assets/DBCheck_PostgreSQL_v10.sql)  
- [DBCheck_PostgreSQL_V9.sql](/assets/DBCheck_PostgreSQL_v9.sql)  
- [DBCheck_PostgreSQL_V8.sql](/assets/DBCheck_PostgreSQL_v8.sql)

