---
title: "Post-installation"
description: "Check a new Self-Managed Identity Analytics installation, log in and give users access."
---

# Post-installation

Run these checks after each installation and after each upgrade. An installation is complete when every pod is running, every job is complete and the technical administrator can log in to the portal.

## 1. Check the platform

1. All pods are `Running` and all jobs `Complete`:

    ```bash
    kubectl get pods,jobs -n <SHARED_NAMESPACE>
    kubectl get pods,jobs -n <IDA_NAMESPACE>
    ```

    Expected pods: `apisix`, `apisix-etcd-0`, `keycloak-0` (Shared Services), `portal-0`, `batch-server-0`, `controller-0`, `config-0`, `git-server-0`, `extractor`, `ops-manager` (instance), plus `keycloak-db-1` and `shared-db-1` with the built-in database. The portal can take several minutes to become ready.

2. With the built-in database, both PostgreSQL clusters are healthy:

    ```bash
    kubectl get clusters.postgresql.cnpg.io -A
    ```

3. The ingresses exist in the Shared Services namespace and point to your DNS name:

    ```bash
    kubectl get ingress -n <SHARED_NAMESPACE>
    ```

4. The portal answers with a redirection to the Keycloak login page of the instance realm (`/auth/realms/<IDA_NAMESPACE>/…`):

    ```bash
    curl -sI https://<IDA_DNS><IDA_PATH>/portal | grep -i '^location'
    ```

If a check fails, see [Troubleshooting](./11-operations/02-troubleshooting.md).

## 2. Log in

| Interface | URL | Account |
| :-------- | :-- | :------ |
| Portal | `https://<IDA_DNS><IDA_PATH>/portal` | `setup`, with the password set in `global.technicalAdmin.password` |
| Configuration interface | `https://<IDA_DNS><IDA_PATH>/config` | `setup` |
| Controller | `https://<IDA_DNS><IDA_PATH>/controller` | `setup` |
| Keycloak, instance realm | `https://<IDA_DNS>/auth/admin/<IDA_NAMESPACE>/console/` | `setup` |
| Keycloak, master realm | `https://<IDA_DNS>/auth/` | `master-admin`, with `keycloak.auth.adminPassword` of the Shared Services values |

At the first login, Keycloak may ask the technical administrator to change the password and to verify the email address.

## 3. Check the project and the mail

- In the configuration interface, open **Git Project** and check that the repository and branch of the values are synchronised. To use another branch, change it here: this setting is kept across upgrades.
- If SMTP is enabled, check the mail server in the controller configuration (**Email server configuration**) and add the recipients of the execution plan notifications (**Recipients of notification emails**). Without recipients, the controller does not send these mails.

## 4. Give users access

Users log in through the Keycloak realm of the instance (named after `<IDA_NAMESPACE>`), either with local Keycloak accounts or through your identity provider. See [Authentication and authorization configuration](./06-configuration/02-authentication-configuration.md).

To create a local user:

1. Log in to the Keycloak console of the instance realm as `setup`.
2. Create the user under **Users**, then set a password under **Credentials**. Leave **Temporary** on to make the user choose a new password at the first login.
3. Under **Role mapping**, assign one or more client roles of the `identity-analytics` client, for example `user`, `functionaladmin`, `designer` or `technicaladmin`.

A user also needs an account with the same login in the Identity Ledger (the current timeslot) to see the portal. A user without any client role gets `HTTP Status 401 – User not authenticated or not authorized`.

## 5. Load the first timeslot

1. Upload the source files of the project in the controller (**Uploads**, listed in **Import files**), in the folder expected by the collectors.
2. Start the execution plan from the controller, then follow it and download its logs from the same page. See [Controller](./09-components/controller.md).
3. Once the timeslot is validated, the portal displays the data.
