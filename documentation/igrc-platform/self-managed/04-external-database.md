---
title: "External database"
description: "Run Self-Managed Identity Analytics on a PostgreSQL server outside the Kubernetes cluster."
---

# External database

Both charts can use a PostgreSQL server you operate instead of the built-in CloudNativePG database: set `database.type: external` in the Shared Services values (Keycloak database) and in the Identity Analytics values (application databases). In this mode, the CloudNativePG operator is not needed, and backups, high availability and tuning of the database are your responsibility.

> [!note] This page completes [Installation](./03-installation.md): skip step 2 (CloudNativePG) and add the `database` and `keycloak.service` blocks below to your values files.

## Requirements

- PostgreSQL 17 or later (validated with 17; the built-in database of the same chart versions runs PostgreSQL 18.3), reachable from the cluster nodes on its port (5432 by default). Do not use `localhost`: give an address or DNS name the pods can resolve.
- An administration account used by the initialisation jobs at installation and at each upgrade: a superuser, or an account prepared as described in [Administration account](#administration-account). `CREATEDB` and `CREATEROLE` alone are not enough.
- TLS: if the server requires it, the CA certificate of the server, stored in a secret (see [Encrypted connections](#encrypted-connections)).
- The `pg_stat_statements` extension is recommended for performance analysis. It is not created by the charts: ask your DBA to enable it (`shared_preload_libraries`) and to run `CREATE EXTENSION pg_stat_statements;` in `analytics_db`.

## Administration account

The initialisation jobs connect with the administration account to create the databases and the roles, then they:

- create schemas owned by the roles they created, and give them the databases (`CREATE SCHEMA ... AUTHORIZATION`, `ALTER DATABASE ... OWNER TO`). Since PostgreSQL 16, this needs the rights of these roles, which an account with `CREATEROLE` does not get by default;
- refresh the collation version of the databases, including `template1` and `postgres` (`ALTER DATABASE ... REFRESH COLLATION VERSION`), which only their owner or a superuser can do.

With an account that only has `CREATEDB` and `CREATEROLE`, the jobs fail with `must be able to SET ROLE "<role>"` (`init-external-keycloak-db`, `external-db-init`) and `must be owner of database template1` (`analytics-update-db`).

If you do not use a superuser, have your DBA run, as a superuser, before the installation:

```sql
CREATE ROLE <DB_ADMIN_USER> LOGIN CREATEDB CREATEROLE PASSWORD '<DB_ADMIN_PASSWORD>';
-- Give the account the rights of the roles it creates
ALTER ROLE <DB_ADMIN_USER> SET createrole_self_grant = 'set, inherit';
-- Let the account refresh the collation version of these databases
ALTER DATABASE template1 OWNER TO <DB_ADMIN_USER>;
ALTER DATABASE postgres OWNER TO <DB_ADMIN_USER>;
```

`createrole_self_grant` applies only to the roles created afterwards. If the roles already exist, for example after a failed first installation, also give their rights to the account:

```sql
GRANT <KEYCLOAK_DB_USER>, app_user, shared_user, analytics_support, analytics_read_access
  TO <DB_ADMIN_USER> WITH SET TRUE, INHERIT TRUE;
```

Use the role names of your values if you changed them, and add `ai_svc_user` if the AI features are enabled. Then run `helm upgrade` again for the chart whose job failed: the jobs run at each upgrade.

> [!caution] On managed services (Amazon RDS, Azure Database for PostgreSQL, Google Cloud SQL), the administration account is not a superuser and cannot own `template1` and `postgres`: the job `analytics-update-db` fails. Contact Radiant Logic before choosing a managed service.

## Shared Services values

Add to `shared.values.yaml`:

```yaml
database:
  type: external
  external:
    host: "<DB_HOST>"
    port: 5432
    database: "keycloak_db"
    username: "<KEYCLOAK_DB_USER>"
    password: "<KEYCLOAK_DB_PASSWORD>"
    autoInit: true
    adminUser: "<DB_ADMIN_USER>"
    adminPassword: "<DB_ADMIN_PASSWORD>"
    ssl:
      enabled: false
keycloak:
  service:
    password: "<KEYCLOAK_SERVICE_PASSWORD>"
```

> [!warning] `database.external.username` and `password` are required. They are the account Keycloak connects with, created by the initialisation job. If they are missing, the job `init-external-keycloak-db` fails with `syntax error at or near "WITH"`.

`keycloak.service.password` is also required with an external database. If you reinstall on an existing database, reuse the password of the first installation.

## Identity Analytics values

Add to `ida.values.yaml`:

```yaml
database:
  type: external
  external:
    host: "<DB_HOST>"
    port: 5432
    autoInit: true
    adminUser: "<DB_ADMIN_USER>"
    adminPassword: "<DB_ADMIN_PASSWORD>"
    ssl:
      enabled: false
```

> [!warning] `autoInit: true` is required in both values files. Creating the databases, roles and schemas yourself (`autoInit: false`) is not supported.

`port` is required. The job `external-db-init` creates the objects below on the first installation and checks them on each upgrade.

## What the initialisation creates

| Object | Name | Owner or rights |
| :----- | :--- | :-------------- |
| Database | `keycloak_db` (Shared Services) | Administration account. The schema `keycloak` belongs to the Keycloak account |
| Database | `analytics_db` | `app_user` |
| Database | `shared_db` | `shared_user` |
| Schemas of `analytics_db` | `igrc` (Identity Ledger), `activiti` (workflows), `auditlog` | `app_user` |
| Schemas of `shared_db` | `controller`, `configuration`, `gitsync` | `shared_user` |
| Login roles | `app_user`, `shared_user`, `analytics_support` | Passwords generated, or taken from your secrets |
| Group role | `analytics_read_access` | `SELECT` on the tables of the `analytics_db` schemas |

The database, role and schema names can be changed under `database.analytics` and `database.shared` in the Identity Analytics values. The schema `aida` and the role `ai_svc_user` are added only when the AI features are enabled.

## Encrypted connections

Enable TLS to the server in both values files:

```yaml
database:
  external:
    ssl:
      enabled: true
      mode: verify-full
      rootCertSecret: "<PG_CA_SECRET>"
```

`mode` accepts the PostgreSQL `sslmode` values. With `verify-ca` or `verify-full`, `rootCertSecret` is required: create it from the CA certificate of the server in each namespace (`kubectl create secret generic <PG_CA_SECRET> --from-file=ca.crt=<CA_FILE>`).

## Keeping administration passwords out of the values

Instead of `adminUser` / `adminPassword`, reference a secret holding the administration password with `adminExistingSecret` (key `password` by default, see `adminExistingSecretPasswordKey`), and keep `adminUser` for the login.
