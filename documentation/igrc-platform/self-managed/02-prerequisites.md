---
title: "Prerequisites and sizing"
description: "Kubernetes, tools, network, DNS, TLS, registry, and resource requirements for Self-Managed Identity Analytics."
***

# Prerequisites and sizing

Before installing, decide how users will access the platform and where the databases will run. Then review the requirements below and add your environment-specific settings to your values files.

## Deployment decisions

| Decision | Options | Impact |
| :------- | :------ | :----- |
| Database | Built-in PostgreSQL managed by CloudNativePG, or an external PostgreSQL server | Determines whether the CloudNativePG operator is required and who manages database backups. See [External database](./04-external-database.md). |
| Number of instances | One Identity Analytics instance per namespace, such as production and acceptance | Each instance requires its own pods, volumes, and database. |
| Exposure | One DNS name per cluster, with a separate URL path for each instance, such as `/ida` or `/ida-acc` | Determines the DNS record, TLS certificate, and ingress class to configure. |
| Project repository | The read-only Radiant Logic reference repository, a fork hosted by Radiant Logic, or your own Git server | Determines who can modify the project and which credentials the instance uses. See [Project repository](#project-repository). |

## Kubernetes cluster

Your cluster must meet the following requirements:

- A conformant Kubernetes distribution, such as Amazon EKS, Azure Kubernetes Service, Google Kubernetes Engine, Red Hat OpenShift, or RKE2. Use a Kubernetes version supported by both your provider and the CloudNativePG version listed in the [compatibility table](../self-managed).
- A default storage class, or a storage class specified in `global.storageClass`, that provides `ReadWriteOnce` volumes. Examples include `gp3` on AWS and `managed-csi` on Azure.
- An ingress controller with an ingress class, such as ingress-nginx or Traefik. The charts create standard Kubernetes `Ingress` resources.
- Permission to create namespaces. If you use the built-in database, you also need cluster-wide permissions to install the CloudNativePG operator and its custom resource definitions (CRDs).

## Administration workstation

Install the following tools:

- [kubectl](https://kubernetes.io/docs/reference/kubectl/), configured to connect to your cluster. [kubernetes](https://kubernetes.io/docs/reference/kubectl/)
- [Helm](https://helm.sh/) 3.8 or later. The charts are distributed as OCI artifacts. Helm 4 is also supported.

## Information and files from Radiant Logic

Obtain the following before installing:

- Access to the `docker.io/radiantone` registry, provided as either:
  - The `ida-registry-credentials.yaml` image pull secret.
  - A username and token that you can use to create the secret. See [Installation](./03-installation.md#1-prepare-the-namespaces-and-the-registry-secret).
- An Identity Analytics license file.

### Use a registry mirror

If the cluster cannot access Docker Hub, mirror the required images to your own registry.

For `ida-helm` 3.8.0 and `ida-shared-helm` 3.6.0, mirror the following images:

| Chart | Images |
| :---- | :----- |
| `ida-shared-helm` 3.6.0 | `radiantone/ida-apisix-fips:3.6.0`, `radiantone/ida-etcd-fips:3.6.0`, `radiantone/ida-keycloak-fips:3.6.0`, `radiantone/ida-tools-fips:3.6.0`, `radiantone/cgr-postgres-cloudnative-pg-fips:18.4` (built-in database) |
| `ida-helm` 3.8.0 | `radiantone/ida-portal:3.8.0`, `radiantone/ida-batch-server:3.8.0`, `radiantone/ida-controller:3.8.0`, `radiantone/ida-config-server:3.8.0`, `radiantone/ida-git-server:3.8.0`, `radiantone/ida-extract:3.8.0`, `radiantone/ida-ops-manager:3.8.0`, `radiantone/ida-env-manager:3.8.0`, `radiantone/ida-init-db:3.8.0`, `radiantone/ida-tools-fips:3.8.0`, `radiantone/rl-postgres-cloudnative-pg-fips:18.6` (built-in database) |
| CloudNativePG operator (built-in database) | `radiantone/rl-cloudnative-pg-fips:1.30` |

To pull images from your mirror, add the following settings to both values files:

```
image:
  registry: "<MIRROR_REGISTRY>"   # Replaces docker.io/radiantone
imageCredentials:
  - existingSecret: <MIRROR_PULL_SECRET>
```

For the CloudNativePG operator, set `image.repository` to `<MIRROR_REGISTRY>/rl-cloudnative-pg-fips` in `cnpg-custom-values.yaml`. See [Installation](./03-installation.md#2-install-the-cloudnativepg-operator).

To list the images used by another chart version, render the chart and filter the image entries:

```
helm template rlia oci://docker.io/radiantone/ida-helm --version <IDA_CHART_VERSION> \
  --set global.license=dummy --set 'imageCredentials[0].existingSecret=x' | grep 'image:' | sort -u
```

## Information from your organization

Gather the following information and resources:

| Item | Requirements |
| :--- | :----------- |
| DNS | A platform hostname (`<IDA_DNS>`) that points to the ingress controller. |
| TLS | A certificate for the platform hostname, stored in a Kubernetes TLS secret or issued by your certificate manager. |
| Git | If you host the project on your own Git server, the repository URL, branch, and an account or token with read access. See [Project repository](#project-repository). |
| SMTP | The mail relay's hostname, port, security mode, and credentials, if required, for sending notifications. |
| Identity provider (optional) | SAML or OIDC settings if users will sign in through your corporate identity provider. |
| External database (optional) | The PostgreSQL hostname, port, and an administration account. See [External database](./04-external-database.md). |

## Project repository

Each instance runs an Identity Analytics project that defines data collection, the data model, rules, and the portal. The instance reads the project from the `brainwave` folder at the root of a Git repository, as specified by `git.default.fromFolder`, and maintains a local copy in the `git-server` pod.

Choose one of the following repository options:

| Option | Repository and branch | Access | Use when |
| :----- | :-------------------- | :----- | :------- |
| Reference project | `https://repository.brainwavegrc.com/Brainwave/identityanalytics.git`, using the `release/<MAJOR.MINOR>` branch that matches your version, such as `release/3.8` | Read-only | You use the standard project without modifications. |
| Fork hosted by Radiant Logic | A fork of the reference project in an organisation created for you on `repository.brainwavegrc.com` | Read and write | You customise the project, typically on your own branches. |
| Your own Git server | A project that you maintain, based on the Self-Managed reference project | Managed by your organisation | Your organisation requires its own Git server and manages the project and its updates. |

### Repository credentials

All options require an account on `repository.brainwavegrc.com`, typically a service account. Radiant Logic provides this account on request.

How you use the account depends on where the project is hosted:

- Reference project or Radiant Logic-hosted fork: The instance uses this account to read the project. Sign in to `https://repository.brainwavegrc.com` and generate an access token. Configure the instance with the account name as the username and the token as the password, using `authType: basic`.
- Your own Git server: Use the Radiant Logic account to retrieve the reference project. Configure the instance to read from your server using credentials for that server.

### Requirements for your own Git server

Your project must include the Self-Managed reference project for your version. It provides the technical configuration, data source management, and connectors required by a Self-Managed instance. An existing on-premises project alone is not sufficient.

Either:

- Start with the reference project when creating a new project.
- Merge the reference project into your existing project, such as when migrating an on-premises installation. See [Project migration](./10-data-migration/project-migration.md).

To update the project used by a running instance, push your changes to the configured branch. See [Project changes](./11-operations/05-project-changes.md).

> [!note] When upgrading to a new minor version, switch the instance to the matching `release/<MAJOR.MINOR>` branch. If you maintain a customised project, merge that release branch into your own branch instead.

## Network flows

Allow the following network connections:

| From | To | Port | Purpose |
| :--- | :- | :--- | :------ |
| Users | Ingress controller | 443 | Access the portal, configuration interface, and Keycloak sign-in page. |
| Cluster nodes | Image registry (`docker.io`, `ghcr.io`, or your mirror) | 443 | Pull container images. |
| `git-server` pod | Git repository | 443 or 22 | Synchronise the project. |
| `portal`, `batch-server`, and `keycloak` pods | SMTP relay | 25, 465, or 587 | Send email. |
| `keycloak` pod | Identity provider | 443 | Support SAML or OIDC federation. |
| Platform pods | External PostgreSQL server, if used | 5432 | Connect to the databases. |

## Sizing

The following totals reflect the charts' default resource requests and limits for Shared Services and one Identity Analytics instance using the built-in database.

Provide enough node capacity to accommodate the resource requests, with additional headroom. Also ensure that an individual node can accommodate the largest pod: by default, a database pod with a 16 Gi memory limit.

| Scope | CPU requests | Memory requests | CPU limits | Memory limits | Persistent storage |
| :---- | -----------: | --------------: | ---------: | ------------: | -----------------: |
| Shared Services: APISIX, etcd, Keycloak, and the Keycloak database | 1.5 | 1.5 Gi | 4.5 | 12.5 Gi | 96 Gi |
| One Identity Analytics instance, including its database | 5.8 | 13.3 Gi | 12.2 | 33.6 Gi | 252 Gi |
| Total | 7.3 | 14.8 Gi | 16.7 | 46.1 Gi | 348 Gi |

If you use an external database, subtract the database pods' requirements from these cluster totals: 2.5 CPU requested, 4.5 Gi memory requested, and 224 Gi of persistent storage. Size the external PostgreSQL server separately.

These defaults are suitable for small and medium projects. Increase resources for the portal, batch server, and database, along with database storage, as your workload grows. Consider the number of identities, accounts, and permissions loaded, the number of retained timeslots, and concurrent user activity. See [Installation](./03-installation.md) for configuration details.

The main sizing factors are:

| Component and resources | Main sizing factors |
| :---------------------- | :------------------ |
| `shared-db`: CPU, memory, and data volume | Identity Ledger size and number of retained timeslots. |
| `batch-server`: memory | Source file sizes and collector complexity. |
| `portal`: CPU and memory | Number of concurrent users and reports. |
| `git-server`: volume | Project repository size. |
