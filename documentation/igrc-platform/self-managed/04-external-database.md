---
title: "External database"
description: "Run Self-Managed Identity Analytics on a PostgreSQL server outside the Kubernetes cluster."
---

# External database

Both charts can use a PostgreSQL server you operate instead of the built-in CloudNativePG database: set `database.type: external` in the Shared Services values (Keycloak database) and in the Identity Analytics values (application databases). In this mode, the CloudNativePG operator is not needed, and backups, high availability and tuning of the database are your responsibility.

> [!note] This page completes [Installation](./03-installation.md): skip step 2 (CloudNativePG) and add the `database` and `keycloak.service` blocks below to your values files.

## Requirements

- PostgreSQL 17 or later (validated with 17), reachable from the cluster nodes on its port (5432 by default). Managed services (Amazon RDS, Azure Database for PostgreSQL, Google Cloud SQL) work like a self-hosted server. Do not use `localhost`: give an address or DNS name the pods can resolve.
- For `autoInit: true` (recommended): an administration account allowed to create databases and roles (`CREATEDB` and `CREATEROLE`).
- TLS: if the server requires it, the CA certificate of the server, stored in a secret (see [Encrypted connections](#encrypted-connections)).
- The `pg_stat_statements` extension is recommended for performance analysis. It is not created by the charts: ask your DBA to enable it (`shared_preload_libraries`) and to run `CREATE EXTENSION pg_stat_statements;` in `analytics_db`.

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

`port` is required. With `autoInit: true`, the job `external-db-init` creates the objects below on the first installation and checks them on each upgrade.

## What the initialisation creates

| Object | Name | Owner or rights |
| :----- | :--- | :-------------- |
| Database | `keycloak_db` (Shared Services) | Keycloak account, schema `keycloak` |
| Database | `analytics_db` | `app_user` |
| Database | `shared_db` | `shared_user` |
| Schemas of `analytics_db` | `igrc` (Identity Ledger), `activiti` (workflows), `auditlog` | `app_user` |
| Schemas of `shared_db` | `controller`, `configuration`, `gitsync` | `shared_user` |
| Login roles | `app_user`, `shared_user`, `analytics_support` | Passwords generated, or taken from your secrets |
| Group role | `analytics_read_access` | `SELECT` on the tables of the `analytics_db` schemas |

The database, role and schema names can be changed under `database.analytics` and `database.shared` in the Identity Analytics values. The schema `aida` and the role `ai_svc_user` are added only when the AI features are enabled.

## Creating the objects yourself

If your DBA does not grant `CREATEDB` and `CREATEROLE`, create the objects beforehand, set `autoInit: false`, and give the charts the application passwords:

```sql
-- Keycloak (Shared Services)
CREATE ROLE keycloak_user LOGIN PASSWORD '<KEYCLOAK_DB_PASSWORD>';
CREATE DATABASE keycloak_db OWNER keycloak_user ENCODING 'UTF8';
\c keycloak_db
CREATE SCHEMA keycloak AUTHORIZATION keycloak_user;
ALTER ROLE keycloak_user IN DATABASE keycloak_db SET search_path TO keycloak;

-- Identity Analytics
CREATE ROLE app_user LOGIN PASSWORD '<ANALYTICS_PASSWORD>';
CREATE ROLE shared_user LOGIN PASSWORD '<SHARED_PASSWORD>';
CREATE ROLE analytics_support LOGIN PASSWORD '<SUPPORT_PASSWORD>';
CREATE ROLE analytics_read_access NOLOGIN;
CREATE DATABASE analytics_db OWNER app_user ENCODING 'UTF8';
CREATE DATABASE shared_db OWNER shared_user ENCODING 'UTF8';
GRANT CONNECT ON DATABASE analytics_db TO analytics_support;

\c analytics_db
CREATE SCHEMA igrc AUTHORIZATION app_user;
CREATE SCHEMA activiti AUTHORIZATION app_user;
CREATE SCHEMA auditlog AUTHORIZATION app_user;
ALTER ROLE app_user IN DATABASE analytics_db SET search_path TO igrc, activiti, auditlog, public;
ALTER ROLE analytics_support IN DATABASE analytics_db SET search_path TO igrc, public;
GRANT USAGE ON SCHEMA igrc, activiti, auditlog TO analytics_read_access;
ALTER DEFAULT PRIVILEGES FOR ROLE app_user IN SCHEMA igrc GRANT SELECT ON TABLES TO analytics_read_access;
ALTER DEFAULT PRIVILEGES FOR ROLE app_user IN SCHEMA activiti GRANT SELECT ON TABLES TO analytics_read_access;
ALTER DEFAULT PRIVILEGES FOR ROLE app_user IN SCHEMA auditlog GRANT SELECT ON TABLES TO analytics_read_access;
GRANT USAGE ON SCHEMA igrc TO analytics_support;
ALTER DEFAULT PRIVILEGES FOR ROLE app_user IN SCHEMA igrc GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO analytics_support;

\c shared_db
CREATE SCHEMA controller AUTHORIZATION shared_user;
CREATE SCHEMA configuration AUTHORIZATION shared_user;
CREATE SCHEMA gitsync AUTHORIZATION shared_user;
ALTER ROLE shared_user IN DATABASE shared_db SET search_path TO controller, configuration, gitsync, public;
```

Then pass the application credentials, either in clear in the values (`database.analytics.password`, `database.shared.password`, `database.analytics.services.support.password`) or, preferably, through Kubernetes secrets with `username` and `password` keys:

```yaml
database:
  type: external
  external:
    host: "<DB_HOST>"
    port: 5432
    autoInit: false
    analytics:
      existingSecret: "<IDA_DB_SECRET>"
    shared:
      existingSecret: "<SHARED_DB_SECRET>"
```

On the Shared Services side, `database.external.existingSecret` holds the Keycloak account in the same way.

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
