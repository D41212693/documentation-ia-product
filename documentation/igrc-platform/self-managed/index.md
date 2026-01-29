# Self-Managed Identity Analytics Deployment

This document serves as a guide for deploying Radiant Logic's Identity Analytics (IDA) using Helm charts on a Kubernetes cluster. It outlines prerequisites and provides detailed, step-by-step instructions for the deployment.

## Overview

You can deploy self-managed Identity Analytics on Amazon EKS, Azure Kubernetes Service or Google Kubernetes Engine. The installation process exclusively utilizes Helm, meaning you will use `helm install` or `helm upgrade` commands to install and upgrade the deployment.

The table below shows the Helm chart versions to install for each Identity analytics release:

| Release | IDA_HELM chart version | IDA_SHARED_HELM chart version |
| :------ | :--------------------: | :---------------------------: |
| 3.4     |         3.4.3          |             3.2.2             |
| 3.3     |         3.3.2          |             3.2.0             |
| 3.2     |         3.2.0          |             3.1.1             |
| 3.1     |         3.1.0          |             3.1.0             |

Ensure that you specify your target version when running installation and update commands that are listed in this document.

## Design

The Identity Analytics Helm chart consists of two components:

1. **Ida-shared-helm**: This chart includes Shared Services necessary for Identity Analytics instances. It serves as a prerequisite for the Identity Analytics installation, and only one instance of Shared Services is required per cluster.
2. **Ida-helm**: This chart contains the Identity Analytics stack. You can deploy multiple instances of Identity Analytics if needed. If you do so, ensure that each instance resides in its own namespace.

> Prior to deploying these charts, you will need to deploy CloudNativePG. The installation details are covered in a later section.

The following diagram illustrates a typical deployment, showcasing the Shared Services alongside two instances of Identity Analytics, each in its own namespace:

![Deployment Diagram](./images/01-design.png)

## Prerequisites

- [Kubernetes cluster](https://kubernetes.io/docs/setup/) of version **1.27** or higher.
- Install [Helm](https://helm.sh/) version **3.0** or higher.
- Install [kubectl](https://kubernetes.io/docs/reference/kubectl/) version **1.27** or higher and configure it to access your Kubernetes cluster.
- An Identity Data Analytics license key, which will be provided to you during onboarding.
- Ensure that you have received Container Registry Access and image pull credentials named (ida-registry-credentials.yaml) from Radiant Logic during onboarding.
- Ensure that you have necessary storage provisioners and storage classes configured for the Kubernetes cluster. Some examples of supported storage classes are gp2/gp3, Azure disk, etc.
- Estimate sufficient resources (CPU, memory, storage) for the deployment. Your Radiant Logic solutions engineer may guide you with this depending on your use case.

## Steps for Deployment

### 1. Install Cloud Native PG (CNPG)

Installing Cloud Native PG for managing Postgres databases in Kubernetes is a prerequisite for Shared Services and Identity Analytics helm chart deployments. Follow these steps for the installation:

1. **Set kube-context**:

```bash
kubectl cluster-info
kubectl get nodes
```

2. **Create Configuration File**:

Create a file named `cnpg-custom-values.yaml` with the following properties and values:

```yaml
fullnameOverride: cnpg
config:
  create: true
  name: cnpg-controller-manager-config
  data:
    INHERITED_LABELS: app.kubernetes.io/*, radiantlogic.io/*
    INHERITED_ANNOTATIONS: meta.helm.sh/*, helm.sh/*, prometheus.io/*
```

3. **Add CNPG Helm Repository**:

```bash
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm repo update
```

4. **Deploy CNPG**:

```bash
helm upgrade --install cnpg \
  cnpg/cloudnative-pg \
  --namespace cnpg-system \
  --create-namespace \
  --version <version> \
  --wait \
  --values cnpg-custom-values.yaml
```

Please update the version of the operator to install according to the parameters in the table below

| CNPG Chart version | Postgres Version | status      |
| :----------------- | :--------------- | :---------- |
| 0.23.2             | postgres 17.4    | recommended |
| 0.21.5             | postgres 16.3    |             |

5. **Verify Pod Health**:

```bash
kubectl get pods -n cnpg-system
```

6. **Confirm CRD Installation**:

```bash
kubectl get crds
```

Expected CRDs include:

- `scheduledbackups.postgresql.cnpg.io`
- `backups.postgresql.cnpg.io`
- `imagecatalogs.postgresql.cnpg.io`
- `clusters.postgresql.cnpg.io`

### 2. Deploy Shared Services Chart

The `ida-shared-helm` chart must be deployed only once per cluster and is a prerequisite for deploying Identity Analytics instances. Follow these steps:

1. **Set kube-context**.
2. **Configure DNS for Identity Analytics endpoint**.
3. **Create Configuration File**:
   Create a file named `shared-minimal.values.yaml` with the following properties:

```yaml
global:
  externalProtocol: "https"
  externalDns: "<AUTHN_ENDPOINT>" # Provide the authentication DNS endpoint
  imageCredentials:
    private: true
    registry: docker.io
    existingSecret: ida-registry-credentials
ingress:
  enabled: true
  className: "nginx"
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/proxy-body-size: 8m
keycloak:
  auth:
    adminPassword: "" # Set a strong password
```

4. **Install Shared Services**:

```bash
helm upgrade --install rlss \
  oci://docker.io/radiantone/ida-shared-helm \
  --namespace <SHARED_NAMESPACE> \
  --create-namespace \
  --version <SHARED_CHART_VERSION> \
  --values shared-minimal.values.yaml
```

5. **Verify CRD Installation**:

```bash
kubectl get crds
```

Expected CRDs include:

- `eventbus.argoproj.io`
- `eventsources.argoproj.io`
- `sensors.argoproj.io`

### 3. Deploy the Identity Analytics Chart

Before installing the Identity Analytics chart, ensure that you have the following:

- Shared Services are installed and operational.
- OCI registry credentials for images and charts from Radiant Logic.
- Identity Analytics license file.
- GIT repository credentials from your infrastructure team.
- SMTP server settings (required for verification emails).
- DNS for Identity Analytics endpoint.

Follow these steps to deploy the Identity Analytics helm chart:

1. **Set kube-context**.
2. **Create Configuration File**:

Create a file named `env01.values.yaml` with the following properties:

```yaml
global:
  externalProtocol: "https"
  externalDns: "<IDA_ENDPOINT>"
  pathPrefix: "<IDA_URI>"
  ingress:
    enabled: true
    className: "<INGRESS_CLASS_NAME>"
  technicalAdmin:
    email: "<IDA_ADM_EMAIL>"
    password: "<IDA_ADM_INITIAL_PASSWORD>"
git:
  default:
    repository: "<GIT_DEFAULT_REPO>"
    branch: "<GIT_DEFAULT_BRANCH>"
    fromFolder: "brainwave"
    username: "<GIT_DEFAULT_USER>"
    password: "<GIT_DEFAULT_PASSWORD>"
keycloakSettings:
  namespace: "<SHARED_NAMESPACE>"
  externalDns: "<AUTHN_ENDPOINT>"
apisix:
  namespace: "<SHARED_NAMESPACE>"
smtp:
  enabled: true
  host: "<SMTP_HOST>"
  port: <SMTP_PORT>
  auth:
    enabled: false
    username: ""
    password: ""
  ssl: false
  startTls: false
  from:
    mail: "<SMTP_FROM>"
    displayName: "<SMTP_FROM_DISPLAY>"
  replyTo:
    mail: "<SMTP_TO>"
    displayName: "<SMTP_TO_DISPLAY>"
```

3. **Install Identity Analytics**:

```bash
helm upgrade --install rlia \
  oci://docker.io/radiantone/ida-helm \
  --namespace <IDA_NAMESPACE> \
  --create-namespace \
  --version <IDA_CHART_VERSION> \
  --set-file global.licenseFile=<IDA_LICENSE_FILE_PATH> \
  --wait \
  --values env01.values.yaml
```

4. **Verify Deployment**:

```bash
kubectl get pod --namespace <IDA_NAMESPACE>
```

Ensure that the pod named `portal-0` is up and running.

### 4. Access Identity Analytics with Keycloak

After successful deployment of the Helm chart, you will find the following information in your terminal:

- URLs of services, initial username, and instructions on how to retrieve initial credentials.

Open a browser and connect to the Identity Analytics service using the retrieved credentials.

### 5. Update resource configuration

Customize resource requests and limits for deployed Shared Services based on your cluster's capacity.

1. **Configure Shared Services resources**

Include the following properties and set appropriate values for these properties in your shared-minimal.values.yaml:

```yaml
bwapisix:
  resources:
    limits:
      memory: "2Gi"
      cpu: "1"
    requests:
      memory: "512Mi"
      cpu: "500m"
etcd:
  resources:
    limits:
      cpu: "500m"
      memory: 512Mi
    persistence:
      size: 32Gi
: storage: 32Gi
cnpg:
  resources:
    limits:
      memory: 8Gi
      cpu: "2"
    requests:
      memory: 512Mi
      cpu: "500m"
  walStorage: 32Gi
keycloak:
  resources:
    limits:
      memory: 2Gi
      cpu: "1000m"
    requests:
      memory: 512Mi
      cpu: "500m"
```

2. **Configure Identity Analytics resources**

Include the following properties and set appropriate values for these properties in your env01.values.yaml file:

```yaml
batch:
  resources:
    limits:
      cpu: "2"
      memory: 6Gi
    requests:
      cpu: "1"
      memory: 4Gi
  pvc:
    gitSources: 2Gi
    zipLogs: 16Gi
portal:
  resources:
    limits:
      cpu: "2"
      memory: 8Gi
    requests:
      cpu: "1"
      memory: 4Gi
  pvc:
    gitSources: 2Gi
    logs: 16Gi
config:
  resources:
    limits:
      cpu: "500m"
      memory: 512Mi
    requests:
      cpu: "250m"
      memory: "256Mi"
extractor:
  resources:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "1Gi"
      cpu: "500m"
gitServer:
  resources:
    requests:
      memory: "512Mi"
      cpu: "1"
    limits:
      memory: "1Gi"
      cpu: "2"
  pvc:
    gitSources: 2Gi
    logs: 16Gi
database:
  storage: 128Gi
cnpg:
  resources:
    requests:
      memory: 4Gi
```

3. **Configure pod assignments**

You can define node selectors, tolerations, or affinities in both charts if needed in your values file. For detailed explanations, refer to the [Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).

**Example of Shared Services node configuration**

These should be added to `shared-minimal.values.yaml`.

```yaml
global:
  nodeSelector: {}
  tolerations: []

argo-events:
  controller:
    nodeSelector: {}
    tolerations: []
    affinity: {}

bwapisix:
  nodeSelector: {}
  tolerations: []
  affinity: {}
  etcd:
    nodeSelector: {}
    tolerations: []
    affinity: {}

keycloak:
  nodeSelector: {}
  tolerations: []
  affinity: {}
```

**Example of Identity Analytics node configuration**

These should be added to `env01.values.yaml`.

```yaml
global:
  nodeSelector: {}
  tolerations: []

samples:
  affinity: {}
  tolerations: []

batch:
  affinity: {}
  tolerations: []

portal:
  affinity: {}
  tolerations: []

controller:
  affinity: {}
  tolerations: []

config:
  affinity: {}
  tolerations: []

extractor:
  affinity: {}
  tolerations: []

gitServer:
  affinity: {}
  tolerations: []

database:
  affinity: {}
  tolerations: []
```

4. **Configure database parameters**

Tune the Postgres database with appropriate parameters defined in the values file.

**Example of database configuration**

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

## Updating a deployment

Once you make necessary changes to your values files, update the Helm chart deployments to reflect these changes:

```bash
helm upgrade --install rlss \
 oci://docker.io/radiantone/ida-shared-helm \
  --namespace <SHARED_NAMESPACE> \
  --create-namespace \
  --version <SHARED_CHART_VERSION> \
  --values shared-minimal.values.yaml
```

> [!warning] when upgrading the shared service is can be required to perform additional actions BEFORE performing the `helm upgrade --install` command above:
>
> ```sh
> kubectl scale deploy --replicas 0 --namespace ${SHARED_NAMESPACE} --selector "app.kubernetes.io/instance=${SHARED_RELEASE_NAME}"
> kubectl delete deploy --namespace ${SHARED_NAMESPACE} --selector "app.kubernetes.io/instance=${SHARED_RELEASE_NAME}"
>
> kubectl scale sts --replicas 0 --namespace ${SHARED_NAMESPACE} --selector "app.kubernetes.io/instance=${SHARED_RELEASE_NAME}"
> kubectl delete sts --namespace ${SHARED_NAMESPACE} --selector "app.kubernetes.io/instance=${SHARED_RELEASE_NAME}"
> ```
>
> where:
>
> - `${SHARED_NAMESPACE}`: is the name of the namespace where the shared service are installed
> - `${SHARED_RELEASE_NAME}`: is the name of the helm release used for shared services. _e.g._ rlss in the helm command above
>
> These commands are required when migrating from 3.0.0 3.1.0 and 3.1.1

```bash
helm upgrade --install rlia \
  oci://docker.io/radiantone/ida-helm \
  --namespace <IDA_NAMESPACE> \
  --create-namespace \
  --version <IDA_CHART_VERSION> \
  --set-file global.licenseFile=<IDA_LICENSE_FILE_PATH> \
  --wait \
  --values env01.values.yaml
```

## Troubleshooting your Kubernetes environment

The steps listed here are meant to help you identify and troubleshoot issues related to pod deployments in your Kubernetes environment.

1. **Check events for deployment issues**

   This command lists events in the specified namespace, helping to identify any issues related to pod deployment.

   ```bash
   kubectl get events -n <namespace>
   ```

2. **Describe a specific pod**

   This command provides detailed information about the pod, including its status, conditions, and any errors that might be affecting its deployment.

   ```bash
   kubectl describe pods/fid-0 -n <namespace>
   ```

## Deleting Identity Analytics chart

If you would like to uninstall Identity Analytics, you may do so by following these outlined steps. Ensure that Shared Services remain installed during the uninstallation of Identity Analytics.

1. **Uninstall Identity Analytics**:

   ```bash
   helm uninstall rlia \
     --namespace <IDA_NAMESPACE> \
     --ignore-not-found \
     --wait
   ```

2. **Check for any remaining Persistent Volume Claims (PVCs)**:

   ```bash
   kubectl get pvc \
     --namespace <IDA_NAMESPACE> \
     --selector app.kubernetes.io/instance=rlia
   ```

Delete if necessary to free up space:

    ```bash
      kubectl delete pvc \
        --namespace <IDA_NAMESPACE> \
        --selector app.kubernetes.io/instance=rlia
    ```

3. **Delete existing namespace**:

   You may choose to delete the namespace used for Identity Analytics:

   ```bash
   kubectl delete namespace <IDA_NAMESPACE>
   ```

## Deleting Shared Services chart

Do not uninstall Shared Services if the Identity Analytics instance is still deployed. Only uninstall Shared Services after uninstalling Identity Analytics.

1. **Uninstall Shared Services**:

   ```bash
   helm uninstall rlss \
     --namespace <SHARED_NAMESPACE> \
     --ignore-not-found \
     --wait
   ```

2. **Check for any remaining PVCs**:

   Depending on your PVC retention policy, persistent volumes may not be deleted. Verify by running:

   ```bash
   kubectl get pvc \
     --namespace <SHARED_NAMESPACE> \
     --selector app.kubernetes.io/instance=rlss
   ```

   If any PVCs are present, delete them:

   ```bash
   kubectl delete pvc --namespace <IDA_NAMESPACE> <PVC_NAME>
   ```

3. **Delete namespace**:

   You may choose to delete the namespace used for Shared Services:

   ```bash
   kubectl delete namespace <SHARED_NAMESPACE>
   ```

4. **Remove CRDs**:

   Delete all CRDs installed by Shared Services:

   ```bash
   helm show crds oci://docker.io/radiantone/ida-shared-helm \
     --version <SHARED_CHART_VERSION> | kubectl delete -f -
   ```
