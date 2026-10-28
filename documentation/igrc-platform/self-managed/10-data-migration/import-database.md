# Import database

This new feature is to dedicated to data recovery from legacy deployment to IDA Self-Managed deployment. This document describes how to configure deployment to enable data recovery mode and steps to import data into self-managed Postgres database.

Data recovery mode deploys a special pod in IDA namespace and at the same time, disable data ingestion service, governance portal/AIDA and its routes, so that it avoids any data RW to ledger database schema. This pod contains Postgres client configured to connect to ledger database as well as shell/sql scripts to execute data recovery process based on provided extractions.

- `/etc/scripts`: contains data recovery shell scripts
- `/etc/sql`: contains SQL scripts used by shell scripts
- `/tmp`: ephemeral volume with RW permissions
- `/work`: ephemeral volume with RW permissions
- `/work/upload`: contains exported data archive from Oracle or MS SQL Server
- `/work/import`: contains uncompressed exported data from Oracle or MS SQL Server
- `/work/logs`: contains logs from scripts

> We advise to enable data recovery when you install IDA for the first time to avoid extra steps, if you intent to restore database from your legacy deployment.

## Checks

- [ ] For PostgreSQL database the import is only possible from a target PostgreSQL version of 17.4 and below.
- [ ] CNPG operator **must** be equal to version "0.23.2" when performing the data import so that the database images are on version 17.4 of PostgreSQL
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
    # allowed values: sqlserver, oracle, postgres
    type: null
    # schema name required only if type is postgres
    schema: null
```

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
>
> In addition please note that if the pod crashes for any reason, all data, logs, … are lost because the pod is designed has no persistency.

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
