---
title: "Troubleshooting"
description: "Find why a Self-Managed Identity Analytics pod or job does not start."
---

# Troubleshooting

Most installation issues show up as a pod that is not `Running` or a job that is not `Complete`. Look at events first, then at the logs of the container that fails.

## Commands

1. List what is not running, in both namespaces:

    ```bash
    kubectl get pods -n <SHARED_NAMESPACE>
    kubectl get pods,jobs -n <IDA_NAMESPACE>
    ```

2. Read the recent warning events:

    ```bash
    kubectl get events -n <IDA_NAMESPACE> --field-selector type=Warning --sort-by=.lastTimestamp
    ```

3. Describe the pod that fails:

    ```bash
    kubectl describe pod <POD_NAME> -n <IDA_NAMESPACE>
    ```

4. Read the logs of the failing container. Always name the container: init containers do not show their logs by default.

    ```bash
    kubectl logs <POD_NAME> -n <IDA_NAMESPACE> -c <CONTAINER_NAME>
    kubectl logs <POD_NAME> -n <IDA_NAMESPACE> -c <CONTAINER_NAME> --previous
    ```

## Common symptoms

| Symptom | Likely cause | What to do |
| :------ | :----------- | :--------- |
| `helm` fails with `missing registry username` | `imageCredentials` is not a list at the root of the values file | See [Installation](../03-installation.md) |
| Pods stay in `ImagePullBackOff` | Registry secret missing in the namespace | Apply `ida-registry-credentials.yaml` in both namespaces |
| `helm --wait` ends with `context deadline exceeded` | Initialisation jobs take longer than the timeout | Run the command again with a longer `--timeout`, or without `--wait`, and watch the pods |
| IDA `helm upgrade --wait` ends with `resource Job/<IDA_NAMESPACE>/config-store-init-... not ready. status: NotFound`, and the release is `failed` | The jobs of the chart are deleted 10 minutes after they finish (`jobs.ttlSecondsAfterFinished`, 600 by default). When the pods take longer to restart, Helm 4 looks for a job that no longer exists | The changes are applied: check that the pods are running. Run the IDA chart without `--wait`, as in [Installation](../03-installation.md). The next `helm upgrade` marks the release `deployed` again |
| Portal answers 404 right after an install or upgrade, all pods running | Job `environment-setup` failed, so the APISIX routes are missing | Read the logs of `environment-setup`, fix the cause (often Keycloak or APISIX restarting), then run the IDA `helm upgrade` again |
| Portal never becomes available | Technical administrator password does not meet the password policy | Use at least 16 characters with upper case, lower case, digit and special character |
| `keycloak-0` restarts a few times at installation | Keycloak starts before its database | No action: it stabilises once the database is ready |
| Job `init-external-keycloak-db` fails with `syntax error at or near "WITH"` | External database without `database.external.username` / `password` in the Shared Services values | See [External database](../04-external-database.md) |
| `The account 'x' is unknown on this platform` at login | No account with this login in the current timeslot | Load data, or log in with an account present in the Identity Ledger |
| `HTTP Status 401 – User not authenticated or not authorized` | The Keycloak user has no client role of `identity-analytics` | Assign a role, see [Post-installation](../05-post-installation.md) |
| Execution plans end in **Error** with the message `an issue occured with your license`; the batch logs `A valid license file named license.lic is mandatory` | The license is expired or not valid for this installation. The portal starts anyway: it only checks that a license file is present | Ask Radiant Logic for a new license, valid for a Self-Managed installation, then run the `helm upgrade` of the instance with your usual values and `--set global.license="$(base64 -w0 <NEW_LICENSE_FILE>)"`. `global.license` wins over `global.licenseFile`: if a values file still contains the old license in `global.license`, the new one is ignored. The portal and the batch restart and load it; start a new execution plan |
