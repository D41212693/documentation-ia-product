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

4. If no other application uses it, remove the CloudNativePG operator:

    ```bash
    helm uninstall cnpg --namespace cnpg-system
    ```
