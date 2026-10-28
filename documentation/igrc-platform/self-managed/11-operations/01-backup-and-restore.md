---
title: "Backup and restore"
description: "Back up and restore the databases and settings of Self-Managed Identity Analytics."
---

# Backup and restore

No backup is configured by default. With the built-in database, enable the CloudNativePG backups of both charts as soon as the platform holds data you need to keep. With an external database, backups are done by the team that operates the PostgreSQL server.

## What to back up

| Item | Where it lives | How to back it up |
| :--- | :------------- | :---------------- |
| Identity Ledger, workflows, audit logs, configuration store, controller data | `shared-db` cluster of each instance (databases `analytics_db` and `shared_db`) | CloudNativePG backups of the `ida-helm` chart |
| Users, roles, realms, identity provider settings | `keycloak-db` cluster of Shared Services | CloudNativePG backups of the `ida-shared-helm` chart |
| Project | Your Git repository | Already versioned outside the cluster |
| Source files uploaded to the controller | `controller` volumes | Keep them at the source, or use Velero (`backup.velero`) |
| Installation settings | Your values files, the license and the secrets you created | Store them in your configuration management, without plain passwords |

## Enable the database backups

CloudNativePG backs up the databases with Barman to an object store: Amazon S3 or an S3-compatible store, Azure Blob Storage or Google Cloud Storage. It takes scheduled base backups and archives the write-ahead logs continuously, which allows point-in-time restores.

Add the same kind of block to `ida.values.yaml` and to `shared.values.yaml`, with a different destination path for each:

```yaml
database:
  backup:
    enabled: true
    destinationPath: "<BUCKET_PATH>"
    s3:
      enabled: true
      existingSecret: "<S3_SECRET>"
    retentionPolicy: 30d
    schedule:
      enabled: true
      # six fields, seconds first: every day at 02:00
      expression: "<BACKUP_CRON>"
    onUpgrade:
      enabled: true
```

- `existingSecret` holds the `ACCESS_KEY_ID` and `ACCESS_SECRET_KEY` keys. On AWS you can use the pod identity instead, with `s3.inheritFromIAMRole: true`. For S3-compatible stores, set `s3.endpointURL` and, if needed, `s3.endpointCA`.
- For Azure, use the `azure` block (`connectionString`, storage account and key, or `inheritFromAzureAD`). For Google Cloud, use the `gcs` block (`applicationCredentials`, `existingSecret` or `gkeEnvironment`).
- `onUpgrade.enabled` takes a backup before each chart upgrade.
- `data.compression`, `wal.compression` and the `encryption` settings control how backups are stored.

Check the backups with:

```bash
kubectl get backups.postgresql.cnpg.io,scheduledbackups.postgresql.cnpg.io -n <IDA_NAMESPACE>
kubectl get backups.postgresql.cnpg.io,scheduledbackups.postgresql.cnpg.io -n <SHARED_NAMESPACE>
```

## Restore

A restore creates a new database cluster from a backup. Use it on a new installation (new cluster or new namespace), or after removing the release and its volumes.

1. Install the chart with the same values, plus a `recovery` block pointing to the backup:

    ```yaml
    database:
      recovery:
        enabled: true
        # name of the backed-up cluster: shared-db (ida-helm) or keycloak-db (ida-shared-helm)
        serverName: "shared-db"
        externalCluster:
          name: backup-source
          barmanObjectStore:
            destinationPath: "<BUCKET_PATH>"
            s3:
              enabled: true
              existingSecret: "<S3_SECRET>"
        # optional: restore to a point in time instead of the latest state
        recoveryTarget:
          targetTime: ""
    ```

    `serverName` is required: it is the name under which the backed-up cluster stored its backups in the object store.

2. Wait until the database cluster is healthy and the instance pods are running, then check the data in the portal.
3. Remove the `recovery` block and keep `backup` enabled, then run `helm upgrade` again: the restored cluster starts its own backup chain.

Restore Shared Services (Keycloak) and the instances from backups taken at about the same time, so that users and roles match the data.

> [!caution] Test the restore procedure on a non-production cluster before you rely on it.
