---
title: "Prerequisites and sizing"
description: "Cluster, tools, network, DNS, TLS, registry and sizing requirements for Self-Managed Identity Analytics."
---

# Prerequisites and sizing

Before the installation, decide how the platform is exposed and where its database runs, then check the items below. Everything listed here is generic: the values that depend on your environment go into your values files.

## Decisions to make first

| Decision | Options | Impact |
| :------- | :------ | :----- |
| Database | Built-in PostgreSQL managed by CloudNativePG, or an external PostgreSQL server | CloudNativePG operator required or not; who backs up the data. See [External database](./04-external-database.md). |
| Number of instances | One Identity Analytics instance per namespace (for example production and acceptance) | Each instance adds its own pods, volumes and database |
| Exposure | One DNS name per cluster, one URL path per instance (`/ida`, `/ida-acc`…) | DNS record, certificate and ingress class |
| Project repository | The Radiant Logic reference repository (read only), a fork of it hosted by Radiant Logic, or your own Git server | Who can change the project, and which account the instance uses. See [Project repository](#project-repository). |

## Kubernetes cluster

- A conformant Kubernetes cluster, for example Amazon EKS, Azure Kubernetes Service, Google Kubernetes Engine, Red Hat OpenShift or RKE2. Use a Kubernetes version supported both by your provider and by the CloudNativePG version of the [compatibility table](../self-managed).
- A default storage class, or one you name in `global.storageClass`, providing `ReadWriteOnce` volumes (for example gp3 on AWS or managed-csi on Azure).
- An ingress controller with an ingress class (for example ingress-nginx or Traefik). The charts create standard Kubernetes `Ingress` objects.
- Permission to create namespaces, and cluster-wide permission to install the CloudNativePG operator and its CRDs if you use the built-in database.

## Tools on the administration workstation

- [kubectl](https://kubernetes.io/docs/reference/kubectl/) configured for the cluster.
- [Helm](https://helm.sh/) 3.8 or later (the charts are distributed as OCI artifacts). Helm 4 also works.

## Information and files from Radiant Logic

- The access to the `docker.io/radiantone` registry: either the image pull secret `ida-registry-credentials.yaml`, or a username and a token to create this secret yourself (see [Installation](./03-installation.md#1-prepare-the-namespaces-and-the-registry-secret)).
- The Identity Analytics license file.

If the cluster cannot reach Docker Hub, mirror the images to your own registry. For `ida-helm` 3.7.4 and `ida-shared-helm` 3.5.4, the images are:

| Chart | Images |
| :---- | :----- |
| `ida-shared-helm` 3.5.4 | `radiantone/ida-apisix:3.5.4`, `radiantone/ida-etcd:3.5.4`, `radiantone/ida-keycloak:3.5.4` (`ida-keycloak-fips` in FIPS mode), `radiantone/ida-tools:3.5.4`, `radiantone/kubectl:secure` |
| `ida-helm` 3.7.4 | `radiantone/ida-portal:3.7.4`, `radiantone/ida-batch-server:3.7.4`, `radiantone/ida-controller:3.7.4`, `radiantone/ida-config-server:3.7.4`, `radiantone/ida-git-server:3.7.4`, `radiantone/ida-extract:3.7.4`, `radiantone/ida-ops-manager:3.7.4`, `radiantone/ida-env-manager:3.7.4`, `radiantone/ida-init-db:3.7.4`, `radiantone/ida-tools:3.7.4`, `radiantone/kubectl:secure` |
| CloudNativePG (internal database) | `ghcr.io/cloudnative-pg/cloudnative-pg:1.29.1`, `ghcr.io/cloudnative-pg/postgresql:18.3-system-trixie` |

To pull the images from your mirror, add to both values files:

```yaml
image:
  registry: "<MIRROR_REGISTRY>"   # replaces docker.io/radiantone
imageCredentials:
  - existingSecret: <MIRROR_PULL_SECRET>
```

The `kubectl` image of the upgrade jobs has its own setting: `tools.kubectl.image.repository` in the Identity Analytics values and `preUpgradeChart.kubectl.image.repository` in the Shared Services values (for example `<MIRROR_REGISTRY>/kubectl`).

To list the images of another version, render the chart and filter the image lines:

```bash
helm template rlia oci://docker.io/radiantone/ida-helm --version <IDA_CHART_VERSION> \
  --set global.license=dummy --set 'imageCredentials[0].existingSecret=x' | grep 'image:' | sort -u
```

## Information from your organisation

- **DNS**: a name for the platform (`<IDA_DNS>`) pointing to the ingress controller.
- **TLS**: a certificate for that name, stored as a Kubernetes TLS secret, or issued by your certificate manager.
- **Git** (only if you host the project on your own Git server): the URL of the repository, the branch, and an account or token with read access. See [Project repository](#project-repository).
- **SMTP**: host, port, security mode and optional credentials of a mail relay, to send notifications.
- **Identity provider** (optional): the SAML or OIDC settings if users log in with your corporate identity provider.
- **External database** (optional): host, port and an administration account, see [External database](./04-external-database.md).

## Project repository

Each instance runs an Identity Analytics project: the configuration of the data collection, the data model, the rules and the portal. The instance reads it from a Git repository, in the folder `brainwave` at the root of the repository (`git.default.fromFolder`), and keeps a local copy of it (`git-server` pod).

Choose one of these options:

| Option | Repository and branch | Access | Use it when |
| :----- | :-------------------- | :----- | :---------- |
| Reference project | `https://repository.brainwavegrc.com/Brainwave/identityanalytics.git`, branch `release/<MAJOR.MINOR>` of your version (for example `release/3.7`) | Read only | You use the standard project without changes |
| Fork hosted by Radiant Logic | A fork of the reference project, in an organisation created for you on `repository.brainwavegrc.com` | Read and write | You adapt the project; you usually work on your own branches |
| Your own Git server | A copy of the reference project that you maintain | Your choice | Your organisation requires its own Git server; you handle the copy and its updates |

For the first two options, you need an account on `repository.brainwavegrc.com`, usually a service account, provided by Radiant Logic on request. Log in to `https://repository.brainwavegrc.com` with this account and generate an access token: the instance uses the account name as username and the token as password (`authType: basic`).

To change the project of a running instance, push to its branch: see [Project changes](./11-operations/05-project-changes.md).

> [!note] When you upgrade to a new minor version, switch the instance to the matching branch (`release/<MAJOR.MINOR>`), or merge it into your own branch if you adapted the project.

## Network flows

| From | To | Port | Purpose |
| :--- | :- | :--- | :------ |
| Users | Ingress controller | 443 | Portal, configuration interface, Keycloak login |
| Cluster nodes | Image registry (`docker.io`, `ghcr.io` or your mirror) | 443 | Pull the images |
| `git-server` pod | Git repository | 443 or 22 | Synchronise the project |
| `portal`, `batch-server`, `keycloak` pods | SMTP relay | 25, 465 or 587 | Send mail |
| `keycloak` pod | Identity provider | 443 | SAML or OIDC federation |
| Platform pods | External PostgreSQL (if used) | 5432 | Databases |

## Sizing

The default requests and limits of the charts amount to the figures below for Shared Services plus one Identity Analytics instance with the built-in database. Plan the nodes so that the requests fit with some headroom, and so that one node can host the largest pod (the database, 16 Gi limit by default).

| Scope | CPU requests | Memory requests | CPU limits | Memory limits | Persistent storage |
| :---- | -----------: | --------------: | ---------: | ------------: | -----------------: |
| Shared Services (APISIX, etcd, Keycloak, Keycloak database) | 1.5 | 1.5 Gi | 4.5 | 12.5 Gi | 96 Gi |
| One Identity Analytics instance (with its database) | 5.8 | 13.3 Gi | 12.2 | 33.6 Gi | 252 Gi |
| **Total** | **7.3** | **14.8 Gi** | **16.7** | **46.1 Gi** | **348 Gi** |

With an external database, remove the database pods (2.5 CPU and 4.5 Gi requested, 224 Gi of storage) from the cluster figures, and size the PostgreSQL server instead.

These defaults suit small and medium projects. Increase the portal, batch and database resources and the database volume with the number of identities, accounts and permissions loaded, the number of timeslots kept and the number of concurrent users, as shown in [Installation](./03-installation.md). The main drivers are:

| Component | Grows with |
| :-------- | :--------- |
| `shared-db` (CPU, memory, data volume) | Volume of the Identity Ledger and number of timeslots kept |
| `batch-server` (memory) | Size of the source files and complexity of the collectors |
| `portal` (CPU, memory) | Number of concurrent users and reports |
| `git-server` (volume) | Size of the project repository |
