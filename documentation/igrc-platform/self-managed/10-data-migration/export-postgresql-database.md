---
title: "Export a PostgreSQL database"
description: "Export the Identity Ledger schema of an on-premises PostgreSQL database for import into Self-Managed."
---

# Export a PostgreSQL database

When migrating a PostgreSQL database, no data conversion is required: export the schema holding the Identity Ledger tables with `pg_dump`. See the [official documentation](https://www.postgresql.org/docs/current/app-pgdump.html) for more information.

> [!note] The output format must be the custom format (`--format=c`). Name the file with the `.sql` extension expected by the import scripts, even though it is not a plain SQL file.

Example:

```bash
pg_dump --file "<output_file>.sql" --format=c \
  --host "<source_database_host>" --port <source_database_port> \
  --schema "<source_database_schema>" -d "<source_database_name>" \
  --username "<user>"
```

Where:

- `output_file`: full path of the output file.
- `source_database_host`, `source_database_port`: server of the on-premises database.
- `source_database_schema`: schema holding the Identity Ledger tables. It is also the value of `dataRecovery.from.schema` at import time.
- `source_database_name`: name of the on-premises database.
- `user`: account with read rights on all the objects of the schema.

Use a `pg_dump` version at least as recent as the source server, and check that the source server version is not newer than the target database (see [Import database](./import-database.md)). Then put the file in a folder of an `export.tgz` archive as described on the import page.
