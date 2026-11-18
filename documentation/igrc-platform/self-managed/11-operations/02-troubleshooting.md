---
title: "Troubleshooting"
description: "Diagnose pods and jobs that fail to start in Self-Managed Identity Analytics."
***

# Troubleshooting

Most installation issues appear as pods that are not `Running` or jobs that are not `Complete`. Start by reviewing warning events, then check the logs of the failing container.

## Diagnostic commands

1. Check the status of pods and jobs in both namespaces:

    ```
    kubectl get pods -n <SHARED_NAMESPACE>
    kubectl get pods,jobs -n <IDA_NAMESPACE>
    ```

2. Review recent warning events:

    ```
    kubectl get events -n <IDA_NAMESPACE> --field-selector type=Warning --sort-by=.lastTimestamp
    ```

3. Inspect the failing pod:

    ```
    kubectl describe pod <POD_NAME> -n <IDA_NAMESPACE>
    ```

4. Read the logs of the failing container. Specify the container name explicitly, including for init containers, whose logs are not shown by default. Use `--previous` to view logs from the previous container instance.

    ```
    kubectl logs <POD_NAME> -n <IDA_NAMESPACE> -c <CONTAINER_NAME>
    kubectl logs <POD_NAME> -n <IDA_NAMESPACE> -c <CONTAINER_NAME> --previous
    ```

For resources in Shared Services, replace `<IDA_NAMESPACE>` with `<SHARED_NAMESPACE>`.

## Common symptoms

| Symptom | Likely cause | Resolution |
| :------ | :----------- | :--------- |
| Helm fails with `missing registry username` | `imageCredentials` is not defined as a list at the root of the values file. | Correct the values file. See [Installation](../03-installation.md). |
| Pods remain in `ImagePullBackOff` | The registry secret is missing from the namespace. | Apply `ida-registry-credentials.yaml` in both namespaces. |
| `helm --wait` fails with `context deadline exceeded` | Initialisation jobs exceed the timeout. | Run the command again with a longer `--timeout`, or omit `--wait` and monitor the pods. |
| The IDA `helm upgrade --wait` command fails with `resource Job/<IDA_NAMESPACE>/config-store-init-... not ready. status: NotFound`, and the release status is `failed` | Chart jobs are deleted after the period specified by `jobs.ttlSecondsAfterFinished`, which defaults to 600 seconds (10 minutes). If pods take longer to restart, Helm 4 looks for a job that has already been deleted. | The changes have been applied. Verify that the pods are running, and run the IDA chart without `--wait`, as described in [Installation](../03-installation.md). The next successful `helm upgrade` returns the release to `deployed`. |
| The portal returns HTTP 404 immediately after an installation or upgrade, although all pods are running | The `environment-setup` job failed, leaving the APISIX routes unconfigured. | Review the `environment-setup` logs and resolve the cause, which is often a Keycloak or APISIX restart. Then run `helm upgrade` for the IDA release again. |
| The portal never becomes available | The technical administrator password does not meet the password policy. | Use a password with at least 16 characters, including an uppercase letter, a lowercase letter, a digit, and a special character. |
| `keycloak-0` restarts several times during installation | Keycloak starts before its database is ready. | No action is required. Keycloak stabilises when the database becomes available. |
| The `init-external-keycloak-db` job fails with `syntax error at or near "WITH"` | `database.external.username` or `database.external.password` is missing from the Shared Services values for the external database. | Configure the external database credentials. See [External database](../04-external-database.md). |
| Login fails with `The account 'x' is unknown on this platform` | The current timeslot contains no account with that login. | Load the required data, or log in with an account present in the Identity Ledger for the current timeslot. |
| Login fails with `HTTP Status 401 – User not authenticated or not authorized` | The Keycloak user has no role for the `identity-analytics` client. | Assign a client role to the user. See [Post-installation](../05-post-installation.md). |
| Execution plans end in Error with `an issue occured with your license`, and the batch logs contain `A valid license file named license.lic is mandatory` | The license has expired or is not valid for this installation. The portal still starts because it checks only that a license file is present. | Request a license from Radiant Logic that is valid for a Self-Managed installation. Then run `helm upgrade` for the instance with your usual values and `--set global.license="$(base64 -w0 <NEW_LICENSE_FILE>)"`. The `global.license` setting takes precedence over `global.licenseFile`: if your values file contains the old license in `global.license`, changing `global.licenseFile` alone does not replace it. The portal and batch restart and load the new license. Start a new execution plan after the restart. |
