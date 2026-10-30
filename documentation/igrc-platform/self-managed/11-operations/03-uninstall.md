---
title: "Uninstallation"
description: "Remove an Identity Analytics instance and Shared Services from the cluster."
---

# Uninstallation

Remove the Identity Analytics instances first, then Shared Services. Shared Services must stay installed as long as one instance remains.

> [!warning] Deleting the persistent volume claims deletes the data, including the built-in database. Take a [backup](./01-backup-and-restore.md) first if you may need the data again.

## Remove an Identity Analytics instance

1. Uninstall the release:

    ```bash
    helm uninstall rlia --namespace <IDA_NAMESPACE> --wait
    ```

2. List the remaining persistent volume claims, and delete them to free the storage:

    ```bash
    kubectl get pvc --namespace <IDA_NAMESPACE>
    kubectl delete pvc --namespace <IDA_NAMESPACE> --all
    ```

3. Delete the namespace, if it is no longer used:

    ```bash
    kubectl delete namespace <IDA_NAMESPACE>
    ```

## Remove Shared Services

1. Uninstall the release:

    ```bash
    helm uninstall rlss --namespace <SHARED_NAMESPACE> --wait
    ```

2. List the remaining persistent volume claims, and delete them:

    ```bash
    kubectl get pvc --namespace <SHARED_NAMESPACE>
    kubectl delete pvc --namespace <SHARED_NAMESPACE> --all
    ```

3. Delete the namespace:

    ```bash
    kubectl delete namespace <SHARED_NAMESPACE>
    ```

4. With the built-in database, if no other application uses it, remove the CloudNativePG operator:

    ```bash
    helm uninstall cnpg --namespace cnpg-system
    ```

## External database

With an external database, uninstalling the charts does not delete the databases: `analytics_db`, `shared_db` and `keycloak_db` and their roles stay on the PostgreSQL server. Only the realm of the instance is deleted from `keycloak_db` by the `helm uninstall` of the instance, with its users, role assignments and identity provider settings: a reinstallation creates a new realm.

To remove the platform completely, have your DBA drop these databases and roles. To reinstall on them, keep them: see [External database](../04-external-database.md).
