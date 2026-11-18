---
title: "Uninstallation"
description: "Remove an Identity Analytics instance and Shared Services from the cluster."
---

# Uninstallation

Uninstall all Identity Analytics instances before removing Shared Services. Shared Services must remain installed while any instance still uses them.

> [!warning] Deleting persistent volume claims deletes the associated data, including the built-in database. Create a [backup](./01-backup-and-restore.md) first if you might need to recover the data.

## Remove an Identity Analytics instance

1. Uninstall the Helm release:

    ```
    helm uninstall rlia --namespace <IDA_NAMESPACE> --wait
    ```

2. List the remaining persistent volume claims, then delete them to release the storage:

    ```
    kubectl get pvc --namespace <IDA_NAMESPACE>
    kubectl delete pvc --namespace <IDA_NAMESPACE> --all
    ```

3. Delete the namespace if it is no longer needed:

    ```
    kubectl delete namespace <IDA_NAMESPACE>
    ```

## Remove Shared Services

After removing all Identity Analytics instances:

1. Uninstall the Helm release:

    ```
    helm uninstall rlss --namespace <SHARED_NAMESPACE> --wait
    ```

2. List and delete the remaining persistent volume claims:

    ```
    kubectl get pvc --namespace <SHARED_NAMESPACE>
    kubectl delete pvc --namespace <SHARED_NAMESPACE> --all
    ```

3. Delete the namespace:

    ```
    kubectl delete namespace <SHARED_NAMESPACE>
    ```

4. If you used the built-in database and no other application requires the CloudNativePG operator, uninstall it:

    ```
    helm uninstall cnpg --namespace cnpg-system
    ```

    The chart retains the CloudNativePG custom resource definitions (CRDs). To delete them and the operator namespace, run:

    ```
    kubectl get crds -o name | grep postgresql.cnpg.io | xargs kubectl delete
    kubectl delete namespace cnpg-system
    ```

## External database

If you use an external database, uninstalling the charts leaves `analytics_db`, `shared_db`, `keycloak_db`, and their database roles on the PostgreSQL server.

However, uninstalling an Identity Analytics instance deletes its realm from `keycloak_db`, including the realm's users, role assignments, and identity provider settings. Reinstalling the instance creates a new realm.

To reuse the databases for a reinstallation, retain them and follow the guidance in [External database](../04-external-database.md).

To remove the platform completely, ask your DBA to run the following commands. These commands use the default database and role names; adjust them if your installation uses different names.

```
DROP DATABASE analytics_db;
DROP DATABASE shared_db;
DROP DATABASE keycloak_db;
DROP ROLE app_user, shared_user, analytics_support, analytics_read_access, <KEYCLOAK_DB_USER>;
```

Then remove the administration account or revoke its privileges if it is no longer needed.
# Uninstallation

Remove the Identity Analytics instances first, then Shared Services. Shared Services must stay installed as long as one instance remains.

> [!warning] Deleting the persistent volume claims deletes the data, including the built-in database. Take a [backup](./01-backup-and-restore.md) first if you may need the data again.

## Remove an Identity Analytics instance

1. Uninstall the release:

    ```
    helm uninstall rlia --namespace <IDA_NAMESPACE> --wait
    ```

2. List the remaining persistent volume claims, and delete them to free the storage:

    ```
    kubectl get pvc --namespace <IDA_NAMESPACE>
    kubectl delete pvc --namespace <IDA_NAMESPACE> --all
    ```

3. Delete the namespace, if it is no longer used:

    ```
    kubectl delete namespace <IDA_NAMESPACE>
    ```

## Remove Shared Services

1. Uninstall the release:

    ```
    helm uninstall rlss --namespace <SHARED_NAMESPACE> --wait
    ```

2. List the remaining persistent volume claims, and delete them:

    ```
    kubectl get pvc --namespace <SHARED_NAMESPACE>
    kubectl delete pvc --namespace <SHARED_NAMESPACE> --all
    ```

3. Delete the namespace:

    ```
    kubectl delete namespace <SHARED_NAMESPACE>
    ```

4. With the built-in database, if no other application uses it, remove the CloudNativePG operator:

    ```
    helm uninstall cnpg --namespace cnpg-system
    ```

    The chart keeps the CloudNativePG CRDs. To remove them and the namespace:

    ```
    kubectl get crds -o name | grep postgresql.cnpg.io | xargs kubectl delete
    kubectl delete namespace cnpg-system
    ```

## External database

With an external database, uninstalling the charts does not delete the databases: `analytics_db`, `shared_db` and `keycloak_db` and their roles stay on the PostgreSQL server. Only the realm of the instance is deleted from `keycloak_db` by the `helm uninstall` of the instance, with its users, role assignments and identity provider settings: a reinstallation creates a new realm.

To reinstall on them, keep them: see [External database](../04-external-database.md). To remove the platform completely, have your DBA run, with the default names:

```
DROP DATABASE analytics_db;
DROP DATABASE shared_db;
DROP DATABASE keycloak_db;
DROP ROLE app_user, shared_user, analytics_support, analytics_read_access, <KEYCLOAK_DB_USER>;
```

Then remove the administration account, or its privileges, if it is no longer needed.
