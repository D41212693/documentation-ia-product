# Export a PostgreSQL database

When migrating a Postgres database no data conversion is required. The procedure to export the data is then a database dump using the `pg_dump` utility. See the [official documentation](https://www.postgresql.org/docs/current/app-pgdump.html) for more information.

> The output format required is custom `--format=c`

An example of the `pg_dump` command:

```sql
pg_dump --file "<output_file>" --format=c --host "<target_database_host>" --port <target_database_port> --schema "<target_database_schema>" -d "<target_database_name>" --username "<user>"
```

Where:

- `output_file`: is the full path to the output sql file. The file extension must be `.sql`
- `target_database_host`: is the hostname of the server running the target database
- `target_database_port`: is the target database port number
- `target_database_schema`: is the Schema of the target database to export. This is the schema containing the ledger tables
- `target_database_name`: is the name of the target database
- `user`: is the user to perform the dump. This user must have read rights on all elements.
