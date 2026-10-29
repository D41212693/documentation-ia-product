---
title: "Installation"
description: "Install CloudNativePG, Shared Services and Identity Analytics with Helm."
---

# Installation

An installation runs in four steps: prepare the namespaces, install the CloudNativePG operator, install Shared Services, then install one or more Identity Analytics instances. Check the [prerequisites](./02-prerequisites.md) first.

The values shown on this page are minimal and complete: they install a working platform as they are, once the placeholders are replaced. They have been validated with `ida-helm` 3.7.4 and `ida-shared-helm` 3.5.4.

> [!note] If you use a PostgreSQL server outside the cluster, skip step 2 and follow [External database](./04-external-database.md) for the database values.

## Placeholders used on this page

| Placeholder | Description | Example |
| :---------- | :---------- | :------ |
| `<SHARED_NAMESPACE>` | Namespace of Shared Services (one per cluster) | `ida-shared` |
| `<IDA_NAMESPACE>` | Namespace of an Identity Analytics instance. It is also the name of the Keycloak realm of this instance. | `ida` |
| `<IDA_DNS>` | DNS name users reach Identity Analytics on | `ida.example.com` |
| `<IDA_PATH>` | URL path of the instance, starting with `/` | `/ida` |
| `<INGRESS_CLASS>` | Ingress class of your ingress controller | `nginx`, `traefik` |
| `<TLS_SECRET>` | Kubernetes TLS secret holding the certificate of `<IDA_DNS>`, in `<SHARED_NAMESPACE>` | `ida-tls` |
| `<SHARED_CHART_VERSION>` / `<IDA_CHART_VERSION>` | Chart versions from the [compatibility table](../self-managed) | `3.5.4` / `3.7.4` |

## 1. Prepare the namespaces and the registry secret

Create the namespaces and apply the image pull secret received from Radiant Logic (`ida-registry-credentials.yaml`) in **both** namespaces:

```bash
kubectl create namespace <SHARED_NAMESPACE>
kubectl create namespace <IDA_NAMESPACE>
kubectl apply -n <SHARED_NAMESPACE> -f ida-registry-credentials.yaml
kubectl apply -n <IDA_NAMESPACE> -f ida-registry-credentials.yaml
```

If you received a username and a token instead of the file, create the secret in both namespaces. Keep the name `ida-registry-credentials`: the values files below refer to it.

```bash
for ns in <SHARED_NAMESPACE> <IDA_NAMESPACE>; do
  kubectl create secret docker-registry ida-registry-credentials -n "$ns" \n    --docker-server=docker.io --docker-username=<REGISTRY_USER> --docker-password=<REGISTRY_TOKEN>
done
```

If `helm` cannot pull the charts (`unauthorized`), log in to the registry with the same account first: `helm registry login registry-1.docker.io -u <REGISTRY_USER>`.

If you serve TLS from the ingress, create the certificate secret in the Shared Services namespace (the ingresses of both charts are created there):

```bash
kubectl create secret tls <TLS_SECRET> -n <SHARED_NAMESPACE> --cert=tls.crt --key=tls.key
```

## 2. Install the CloudNativePG operator

CloudNativePG (CNPG) creates and manages the PostgreSQL clusters used by Keycloak and Identity Analytics. It is required for the built-in (internal) database only.

1. Create `cnpg-custom-values.yaml`:

    ```yaml
    fullnameOverride: cnpg
    config:
      create: true
      name: cnpg-controller-manager-config
      data:
        INHERITED_LABELS: app.kubernetes.io/*, radiantlogic.io/*
        INHERITED_ANNOTATIONS: meta.helm.sh/*, helm.sh/*, prometheus.io/*
    ```

2. Install the operator:

    ```bash
    helm repo add cnpg https://cloudnative-pg.github.io/charts
    helm repo update
    helm upgrade --install cnpg cnpg/cloudnative-pg \
      --namespace cnpg-system --create-namespace \
      --version 0.28.2 \
      --values cnpg-custom-values.yaml \
      --wait
    ```

3. Check that the operator runs and that the `clusters.postgresql.cnpg.io`, `backups.postgresql.cnpg.io`, `scheduledbackups.postgresql.cnpg.io` and `imagecatalogs.postgresql.cnpg.io` CRDs exist:

    ```bash
    kubectl get pods -n cnpg-system
    kubectl get crds | grep postgresql.cnpg.io
    ```

## 3. Install Shared Services

Shared Services (Keycloak, APISIX gateway and its etcd store) are installed once per cluster.

1. Create `shared.values.yaml`:

    ```yaml
    global:
      externalDns: "<IDA_DNS>"
    imageCredentials:
      - existingSecret: ida-registry-credentials
    ingress:
      enabled: true
      className: "<INGRESS_CLASS>"
      tls:
        - secretName: <TLS_SECRET>
          hosts:
            - "<IDA_DNS>"
    keycloak:
      auth:
        adminPassword: "<KEYCLOAK_MASTER_ADMIN_PASSWORD>"
    ```

    > [!caution] `imageCredentials` is a list at the root of the file, not a key under `global`. A misplaced key makes the installation fail with `missing registry username`.

2. Install the chart:

    ```bash
    helm upgrade --install rlss oci://docker.io/radiantone/ida-shared-helm \
      --namespace <SHARED_NAMESPACE> \
      --version <SHARED_CHART_VERSION> \
      --values shared.values.yaml \
      --wait --timeout 20m
    ```

3. Check that `apisix`, `apisix-etcd-0`, `keycloak-0` and the `keycloak-db-1` database pod are running:

    ```bash
    kubectl get pods -n <SHARED_NAMESPACE>
    ```

    Keycloak can restart once or twice while its database starts: this is expected.

The password of the Keycloak `master-admin` account is the one set in `keycloak.auth.adminPassword`. If you left it empty, it is generated and stored in the `keycloak-master-admin` secret of `<SHARED_NAMESPACE>`.

## 4. Install Identity Analytics

Repeat this step for each instance, each in its own namespace.

1. Gather the inputs of the instance:
    - the license file;
    - the Git repository of the Identity Analytics project, a branch matching the product version (for example `release/3.7` for 3.7.x) and credentials with read access;
    - the SMTP settings, if the instance sends mail;
    - an email address and an initial password for the technical administrator (`setup`): at least 16 characters with upper case, lower case, digit and special character.

2. Create `ida.values.yaml`:

    ```yaml
    global:
      externalDns: "<IDA_DNS>"
      pathPrefix: "<IDA_PATH>"
      technicalAdmin:
        email: "<ADMIN_EMAIL>"
        password: "<ADMIN_INITIAL_PASSWORD>"
    imageCredentials:
      - existingSecret: ida-registry-credentials
    apisix:
      namespace: "<SHARED_NAMESPACE>"
    keycloakSettings:
      namespace: "<SHARED_NAMESPACE>"
    ingress:
      enabled: true
      className: "<INGRESS_CLASS>"
    git:
      default:
        authType: "basic"
        repository: "<GIT_REPOSITORY_URL>"
        branch: "<GIT_BRANCH>"
        fromFolder: "brainwave"
        username: "<GIT_USER>"
        password: "<GIT_PASSWORD_OR_TOKEN>"
    smtp:
      enabled: true
      host: "<SMTP_HOST>"
      port: 25
      auth:
        enabled: false
        username: ""
        password: ""
      ssl: false
      startTls: false
      from:
        mail: "<SMTP_FROM>"
        displayName: "<SMTP_FROM_DISPLAY_NAME>"
      replyTo:
        mail: "<SMTP_REPLY_TO>"
        displayName: "<SMTP_REPLY_TO_DISPLAY_NAME>"
    ```

    Points to check in this file:
    - `apisix.namespace` and `keycloakSettings.namespace` are the **Shared Services** namespace, not the instance namespace.
    - `ingress` is at the root of the file, not under `global`. The ingress is created in the Shared Services namespace and routes `<IDA_PATH>` to the `apisix-gateway` service.
    - Do not set `keycloakSettings.externalDns`: it is reserved for SaaS deployments.
    - `git.default.authType` also accepts `token`, `ssh`, `githubapp` and `none`.

3. Install the chart, passing the license file:

    ```bash
    helm upgrade --install rlia oci://docker.io/radiantone/ida-helm \
      --namespace <IDA_NAMESPACE> \
      --version <IDA_CHART_VERSION> \
      --set-file global.licenseFile=<PATH_TO_LICENSE_FILE> \
      --values ida.values.yaml \
      --timeout 30m
    ```

    The installation runs initialisation jobs (`config-store-init`, `analytics-update-db`, `environment-setup`) and takes a few minutes. Instead of `--set-file`, you can also put the license, base64-encoded, in `global.license`.

4. Continue with the [post-installation checks](./05-post-installation.md).

## Adjust resources and storage

The charts come with default requests, limits and volume sizes (see [Prerequisites and sizing](./02-prerequisites.md)). Override them in your values files, then run the same `helm upgrade --install` commands.

**Shared Services** (`shared.values.yaml`):

```yaml
bwapisix:
  resources:
    requests: { cpu: 500m, memory: 512Mi }
    limits: { cpu: "1", memory: 2Gi }
etcd:
  resources:
    limits: { cpu: 500m, memory: 512Mi }
  persistence:
    size: 32Gi
keycloak:
  resources:
    requests: { cpu: 500m, memory: 512Mi }
    limits: { cpu: "1", memory: 2Gi }
database:
  resources:
    requests: { cpu: 500m, memory: 512Mi }
    limits: { cpu: "2", memory: 8Gi }
  persistence:
    data: { size: 32Gi }
    wal: { size: 32Gi }
```

**Identity Analytics** (`ida.values.yaml`):

```yaml
portal:
  resources:
    requests: { cpu: "1", memory: 4Gi }
    limits: { cpu: "2", memory: 8Gi }
  pvc:
    logs: 16Gi
batch:
  resources:
    requests: { cpu: "1", memory: 4Gi }
    limits: { cpu: "2", memory: 6Gi }
  pvc:
    zipLogs: 16Gi
controller:
  resources:
    requests: { cpu: 250m, memory: 256Mi }
    limits: { cpu: "1", memory: 1Gi }
gitServer:
  pvc:
    gitSources: 32Gi
    logs: 16Gi
database:
  instances: 1
  resources:
    requests: { cpu: "2", memory: 4Gi }
    limits: { cpu: "4", memory: 16Gi }
  persistence:
    storageClass: ""
    data: { size: 128Gi }
    wal: { size: 32Gi }
```

To use a specific storage class for every volume of a chart, set `global.storageClass`.

### Pod placement

Node selectors, tolerations and affinities are set per component, or for the whole chart under `global`. See the [Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).

```yaml
# shared.values.yaml
global:
  nodeSelector: {}
  tolerations: []
bwapisix: { nodeSelector: {}, tolerations: [], affinity: {} }
etcd: { nodeSelector: {}, tolerations: [], affinity: {} }
keycloak: { nodeSelector: {}, tolerations: [], affinity: {} }

# ida.values.yaml
global:
  nodeSelector: {}
  tolerations: []
portal: { affinity: {}, tolerations: [] }
batch: { affinity: {}, tolerations: [] }
controller: { affinity: {}, tolerations: [] }
config: { affinity: {}, tolerations: [] }
extractor: { affinity: {}, tolerations: [] }
gitServer: { affinity: {}, tolerations: [] }
database: { tolerations: [] }
```

To give the platform pods a priority class, set `global.priorityClassName`.

### PostgreSQL parameters

The built-in database is tuned with `database.postgres.overrideConf` in `ida.values.yaml`. The defaults are:

```yaml
database:
  postgres:
    overrideConf:
      autovacuum_vacuum_scale_factor: "0.2"
      maintenance_work_mem: 1GB
      default_statistics_target: "200"
      min_parallel_table_scan_size: 10MB
      min_parallel_index_scan_size: 1MB
      max_connections: "500"
      max_wal_size: 1GB
      random_page_cost: "1.1"
      temp_buffers: 64MB
      work_mem: 12MB
      track_activity_query_size: "16384"
      pg_stat_statements.max: "10000"
      pg_stat_statements.track: all
      auto_explain.log_min_duration: "300s"
```

Keep them consistent with `database.resources`: `shared_buffers` is computed by CloudNativePG from the memory limit.
