---
title: "Post-installation"
description: "Check a new Self-Managed Identity Analytics installation, log in and give users access."
---

# Post-installation

Run these checks after each installation and after each upgrade. They lead to two milestones:

| Milestone | Criteria | Steps |
| :-------- | :------- | :---- |
| **Platform installed** | All pods running and all jobs complete; the technical administrator logs in to the configuration interface and to the controller; the project is synchronised from Git | 1 to 3 |
| **Platform usable** | A first execution plan is completed and its timeslot is validated: users see their data in the portal | 4 and 5 |

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

At the first login, Keycloak asks the technical administrator to change the password. It may also ask to verify the email address.

## 3. Check the project and the mail

- In the configuration interface, open **Git Project** and check the repository and the branch. To use another branch, change it here: this setting is kept across upgrades.
- Check that the project is synchronised: the `portal-0` pod becomes ready only once a synchronised version of the project is available (otherwise its logs show `No Git hash is available yet`). In case of doubt, read the logs of the `git-server-0` pod. See [Project changes](./11-operations/05-project-changes.md).
- If SMTP is enabled, check the SMTP settings in the controller (**Configuration**, **Email server** tab, read only) and add the recipients of the execution plan notifications (**Configuration**, **Notifications** tab). Without recipients, the controller does not send these mails.

## 4. Give users access

Users log in through the Keycloak realm of the instance (named after `<IDA_NAMESPACE>`), either with local Keycloak accounts or through your identity provider. See [Authentication and authorization configuration](./06-configuration/02-authentication-configuration.md).

To create a local user:

1. Log in to the Keycloak console of the instance realm as `setup`.
2. Create the user under **Users**, then set a password under **Credentials**. Leave **Temporary** on to make the user choose a new password at the first login.
3. Under **Role mapping**, assign one or more client roles of the `identity-analytics` client, for example `user`, `functionaladmin`, `designer` or `technicaladmin`.

A user also needs an account with the same login in the Identity Ledger (the current timeslot) to see the portal. A user without any client role gets `HTTP Status 401 – User not authenticated or not authorized`.

## 5. Load the first timeslot

1. Send the source files of the project to the instance: from the portal (**Data source Management**, **Status**, **Upload data**), with the PowerShell extractor, or with the connectors. They are listed in **Import files** in the controller.
2. Start the execution plan: **Submit a new execution plan** in the controller (**Execution plans**), or **Launch an execution plan** in the portal. Follow it and download its logs from the controller. See [Controller](./09-components/controller.md).
3. By default, the timeslot is validated automatically at the end of a successful plan, and the portal displays the data: the platform is usable.

See [Data loading](./06-configuration/06-data-loading.md) for the details: data source declaration, folders of the import files, ways to send the files and to start a plan.
