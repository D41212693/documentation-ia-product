---
title: "Import database"
description: "Import an exported Identity Ledger into the Self-Managed database with the data recovery mode."
---

# Import database

This new feature is to dedicated to data recovery from legacy deployment to IDA Self-Managed deployment. This document describes how to configure deployment to enable data recovery mode and steps to import data into self-managed Postgres database.

Data recovery mode deploys a special pod in IDA namespace and at the same time, disable data ingestion service, governance portal/AIDA and its routes, so that it avoids any data RW to ledger database schema. This pod contains Postgres client configured to connect to ledger database as well as shell/sql scripts to execute data recovery process based on provided extractions.

- `/etc/scripts`: contains data recovery shell scripts
- `/etc/sql`: contains SQL scripts used by shell scripts
- `/tmp`: ephemeral volume with RW permissions
- `/work`: persistent volume (10 Gi by default, `dataRecovery.persistence.size`) with RW permissions
- `/work/upload`: contains exported data archive from Oracle or MS SQL Server
- `/work/import`: contains uncompressed exported data from Oracle or MS SQL Server
- `/work/logs`: contains logs from scripts

> We advise to enable data recovery when you install IDA for the first time to avoid extra steps, if you intent to restore database from your legacy deployment.

## Checks

- [ ] For a PostgreSQL source, the version of the source server must not be newer than the PostgreSQL version of the target database (18.3 with the CloudNativePG chart 0.28.2 of the [compatibility table](../../self-managed)). Do not downgrade the CloudNativePG operator of an existing cluster to match an older source: CloudNativePG does not support operator downgrades.
- [ ] The `/work` volume can hold the archive and its extracted content: set `dataRecovery.persistence.size` to at least twice the size of the uncompressed export.
- [ ] You must have access to your target Kubernetes cluster as well as network connectivity to upload your exported data files to the data recovery pod
- [ ] You own the exported data files package
- [ ] Ensure you correctly tune the self-managed database to import data from legacy database such as resources requests / limits, PVCs size, Postgres settings consistent with resources, …
- [ ] Also tune the data recovery section to your custom Values.yaml file. Here is the default stanza for it.

```yaml
dataRecovery:
  resources:
    limits:
      memory: 256Mi
      cpu: 1
    requests:
      memory: 256Mi
      cpu: 500m
```

## Enable data recovery mode

In your custom `values.yaml` file, add or update the stanza to enable data recovery.

```yaml
dataRecovery:
  enabled: true
  from:
    # required: sqlserver, oracle or postgres
    type: oracle
    # required only when type is postgres: name of the exported ledger schema
    # schema: igrc
  persistence:
    size: 10Gi
```

`from.type` is required: the installation fails with `dataRecovery.from.type must be one of: sqlserver, oracle, postgres` when it is empty.

Then proceed to the IDA deployment with this custom values file.

> Ensure that all pods are healthy before next steps.
>
> Remember that some pods such as portal or data ingestion service are not present in that mode.

## Find the data recovery pod name

You will need to find the data recovery pod name corresponding to your deployment. Execute the following command to get the pod name.

```sh
kubectl get pod --no-headers -o custom-columns=":metadata.name" \
  --field-selector=status.phase=Running \
  --selector app.kubernetes.io/component=data-recovery \
  --namespace <ida_namespace>
```

> [!warning] The name of the pod changes each time a new one is created.

## Transfer exported data files archive

It is necessary to create an archive named `export.tgz`, in `tar` `gzip` format, that contains all exported data files in a directory in the archive.

The structure should be:

```txt
-> export.tgz
----> folder
--------> file 1
--------> file 2
...
--------> file n
```

Execute the command (`bash` example) to upload the archive. This command will copy the `/path/to/export.tgz` archive to the pod name `<pod_name>`, container named data-recovery deployed in the namespace `<ida_namespace>`, with the same name in /work/upload.  

```sh
kubectl cp --namespace=<ida_namespace> --container data-recovery --request-timeout=10m \
  /path/to/export.tgz <pod_name>:/work/upload
```

> [!warning] Adapt the timeout, especially with low bandwidth and huge archive.

The archive and the logs are kept on the persistent `/work` volume if the pod restarts. The volume is removed with the data recovery deployment when you disable data recovery.

### Alternative: download the archive from object storage

If you cannot copy files to the pod, the pod can download `export.tgz` itself from Amazon S3, Azure Blob Storage or Google Cloud Storage when it starts. Add a `downloader` block to the data recovery values:

```yaml
dataRecovery:
  downloader:
    enabled: true
    # aws, azure, gcp, or other (custom image and command)
    provider: aws
    credentials:
      # secret holding the cloud credentials (keys AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY,
      # AZURE_STORAGE_ACCOUNT / AZURE_STORAGE_KEY, or GOOGLE_APPLICATION_CREDENTIALS)
      existingSecret: "<DOWNLOAD_SECRET>"
      aws:
        region: "<REGION>"
        # set to true to use the pod identity instead of a secret
        useIRSA: false
    storage:
      # bucket (S3, GCS) or container (Azure)
      containerName: "<BUCKET>"
      filePath: "migration/export.tgz"
```

`existingSecret` is not needed with `credentials.aws.useIRSA`, `credentials.azure.useWorkloadIdentity` or `credentials.gcp.useWorkloadIdentity`. The file is downloaded to `/work/upload/export.tgz`, ready for the recovery scripts.

## Connect to the pod

Use your favorite method to open a shell session on the pod. For instance, from your laptop you can use

```sh
kubectl exec --namespace=<ida_namespace> --container data-recovery -it <pod_name> -- bash
```

## Execute the recovery process

There are 4 main steps to execute based on provided scripts.

> Ensure you successfully completed the previous steps.
>
> Each script generates logs to console and generate files in `/work/logs`.

When connected to the pods:

- Execute `/etc/scripts/pre_import.sh`. The following operations are processed
  - Connect to Analytics database (self-managed instance), clean any database objects in analytics schema and create expected tables only, corresponding to IDA version
  - Exported data files archive is uncompressed (only tar gzip format is supported) in `/work/import` folder. All the exported data files should be at the root of `/work/import`. If it is not the case, fix it before continuing.
  - Update all SQL exported files to adapt to your environment
- Execute `/etc/scripts/import.sh`. It loads all exported data to Analytics tables.
- Execute `/etc/scripts/post_import.sh`. The following operations are processed
  - Apply expected SQL constraints, integrity relations, indexes creation and sequences initialization
  - Save in config store the ledger database version that has been restored
- Execute `/etc/scripts/start_maintenance.sh`. It execute a database maintenance plan (vacuum full and reindex). This operation is asynchronous. A Kubernetes job is executed, whose name is ops-db-maintenance. Note that only one job can be executed at a time. If you try to re-execute while a job is currently running, it will failed.

Once all step are executed:

- Exit from the pod
- Monitor the maintenance job using the command

```sh
kubectl logs --namespace <ida_namespace> jobs/ops-db-maintenance --ignore-errors --follow
```

- Retrieve all the log files for troubleshooting if needed. This command downloads the pod logs folder (`/work/logs`) to your laptop folder (`/local/path/to/logs`).

```sh
kubectl cp --namespace <ida_namespace> --container data-recovery --request-timeout=10m \
  <pod_name>:/work/logs /local/path/to/logs 
```

> If any steps failed, use logs to investigate. If you need to restart from the beginning, execute the script `/etc/scripts/rollback.sh` to start from scratch again.

## Validate the import

Before you switch users to the new platform:

- [ ] All four scripts ended without error (check `/work/logs`) and the `ops-db-maintenance` job is complete.
- [ ] The number of rows of the main Identity Ledger tables (identities, accounts, permissions, timeslots) matches the source database.
- [ ] After the finalisation below, the portal shows the last timeslot of the source, and a first execution plan runs on the new platform.
- [ ] Keep the source database and the export archive until the new platform is accepted: they are your way back.

## Finalize the recovery process

In your custom Values.yaml file, add or update the stanza to disable data recovery.

```yaml
dataRecovery:
  enabled: false
```

Then proceed to the IDA deployment with this custom values file.

> Ensure that all pods are healthy before trying to connect. A default user, whose login is ‘setup’, has been provisioned.
>
> Data recovery deployment will be deleted and all IDA pods such as portal or data ingestion service are deployed.
