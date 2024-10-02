# Install Self-Managed Identity Analytics Using Helm

This document serves as a guide for deploying Radiant Logic's Identity Analytics (IDA) using Helm charts on a Kubernetes cluster. It outlines prerequisites and provides detailed, step-by-step instructions for the deployment.

## Overview

You can deploy self-managed Identity Analytics on Amazon EKS or Azure Kubernetes Service. The installation process exclusively utilizes Helm, meaning you will use `helm install` or `helm upgrade` commands to install and upgrade the deployment.

## Design

The Identity Analytics Helm chart consists of two components:

1. **Ida-shared-helm**: This chart includes Shared Services necessary for Identity Analytics instances. It serves as a prerequisite for the Identity Analytics installation, and only one instance of Shared Services is required per cluster.
2. **Ida-helm**: This chart contains the Identity Analytics stack. You can deploy multiple instances of Identity Analytics if needed. If you do so, ensure that each instance resides in its own namespace.

> Prior to deploying these charts, you will need to deploy CloudNativePG. The installation details are covered in a later section.

The following diagram illustrates a typical deployment, showcasing the Shared Services alongside two instances of Identity Analytics, each in its own namespace:

![Deployment Diagram](./images/01-design.png)

## Prerequisites

- **Kubernetes Cluster**: Install a Kubernetes cluster of version **1.27** or higher.
- **Helm**: Install Helm version **3.0** or higher.
- **kubectl**: Install kubectl version **1.27** or higher and configure it to access your Kubernetes cluster.
- **Identity Data Analytics License Key**: Provided during onboarding.
- **Container Registry Access**: Ensure that you have saved the image pull credential file (ida-registry-credentials.yaml) provided during onboarding.
- **Storage Provisioners**: Ensure necessary storage provisioners and storage classes are configured for the Kubernetes cluster (e.g., gp2, Azure Disk).
- **Resource Estimation**: Estimate sufficient resources (CPU, memory, storage) for the deployment. Consult your Radiant Logic solutions engineer for guidance.

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
  --version 0.21.5 \
  --wait \
  --values cnpg-custom-values.yaml
```

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
helm upgrade --install $SHARED_HELM_RELEASE \
  oci://$REGISTRY/radiantone/ida-shared-helm \
  --namespace $SHARED_NAMESPACE \
  --create-namespace \
  --version $SHARED_CHART_VERSION \
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
helm upgrade --install $IDA_HELM_RELEASE \
  oci://$REGISTRY/radiantone/ida-helm \
  --namespace $IDA_NAMESPACE \
  --create-namespace \
  --version $IDA_CHART_VERSION \
  --set-file global.licenseFile=$IDA_LICENSE_FILE_PATH \
  --wait \
  --values env01.values.yaml
```

4. **Verify Deployment**:

```bash
kubectl get pod --namespace $IDA_NAMESPACE
```

Ensure that the pod named `portal-0` is up and running.

### Access Identity Analytics with Keycloak

After successful deployment of the Helm chart, you will find the following information in your terminal:

- URLs of services, initial username, and instructions on how to retrieve initial credentials.  

Open a browser and connect to the Identity Analytics service using the retrieved credentials.

### Update Resource Configuration

Customize resource requests and limits for deployed Shared Services based on your cluster's capacity.

#### Shared Services Resource Configuration

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
database:
  storage: 32Gi
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

#### Identity Analytics Resource Configuration

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

### Pod Assignments

You can define node selectors, tolerations, or affinities in both charts if needed in your values file. For detailed explanations, refer to the [Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/).

#### Example of Shared Services Node Configuration

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

#### Example of Identity Analytics Node Configuration

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

### Configure Database Parameters

Tune the Postgres database with appropriate parameters defined in the values file.

#### Example Configuration

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

### Updating a Deployment

Once you make necessary changes to your values files, update the Helm chart deployments to reflect these changes:

```bash
helm upgrade --install $SHARED_HELM_RELEASE \
 oci://$REGISTRY/radiantone/ida-shared-helm \
  --namespace $SHARED_NAMESPACE \
  --create-namespace \
  --version $SHARED_CHART_VERSION \
  --values shared-minimal.values.yaml
```

```bash
helm upgrade --install $IDA_HELM_RELEASE \
  oci://$REGISTRY/radiantone/ida-helm \
  --namespace $IDA_NAMESPACE \
  --create-namespace \
  --version $IDA_CHART_VERSION \
  --set-file global.licenseFile=$IDA_LICENSE_FILE_PATH \
  --wait \
  --values env01.values.yaml
```

### Helm Chart Values

To view the list of values for a specific Helm chart, execute the following command:

#### Shared Services Values

```bash
helm show values \
  oci://$REGISTRY/radiantone/ida-shared-helm \
  --version $SHARED_CHART_VERSION
```

#### Identity Analytics Values

```bash
helm show values \
  oci://$REGISTRY/radiantone/ida-helm \
  --version $IDA_CHART_VERSION
```

Note that even if there are additional values available, Radiant Logic doesn’t recommend modifying undocumented values as it may cause issues with your deployment.

### Uninstall Identity Analytics Chart

If you would like to uninstall Identity Analytics, you may do so by following these outlined steps. Ensure that Shared Services remain installed during the uninstallation of Identity Analytics.

1. **Uninstall Command**:

```bash
helm uninstall $IDA_HELM_RELEASE \
  --namespace $IDA_NAMESPACE \
  --ignore-not-found \
  --wait
```

2. **Check for Any Remaining Persistent Volume Claims (PVCs)**:

```bash
kubectl get pvc \
  --namespace $IDA_NAMESPACE \
  --selector app.kubernetes.io/instance=$IDA_HELM_RELEASE
```

Delete if necessary to free up space:

```bash
  kubectl delete pvc \
    --namespace $IDA_NAMESPACE \
    --selector app.kubernetes.io/instance=$IDA_HELM_RELEASE
```

3. **Delete Existing Namespace**:

You may choose to delete the namespace used for Identity Analytics:

```bash
kubectl delete namespace $IDA_NAMESPACE
```

### Uninstall Shared Services Chart

Do not uninstall Shared Services if the Identity Analytics instance is still deployed. Only uninstall Shared Services after uninstalling Identity Analytics.

1. **Uninstall Command**:

```bash
helm uninstall $SHARED_HELM_RELEASE \
  --namespace $SHARED_NAMESPACE \
  --ignore-not-found \
  --wait
```

2. **Check for Any Remaining PVCs**:

Depending on your PVC retention policy, persistent volumes may not be deleted. Verify by running:

```bash
kubectl get pvc \
  --namespace $SHARED_NAMESPACE \
  --selector app.kubernetes.io/instance=$SHARED_HELM_RELEASE
```

If any PVCs are present, delete them:

```bash
kubectl delete pvc --namespace $IDA_NAMESPACE $PVC_NAME
```

3. **Delete Namespace**:

You may choose to delete the namespace used for Shared Services:

```bash
kubectl delete namespace $SHARED_NAMESPACE
```

4. **Remove CRDs**:

Delete all CRDs installed by Shared Services:

```bash
helm show crds oci://$REGISTRY/radiantone/ida-shared-helm \
  --version $SHARED_CHART_VERSION | kubectl delete -f -
```

### Troubleshooting

If you encounter issues during the installation, consider the following:

- Check the logs of the pods for any error messages:

```bash
kubectl logs <pod-name> -n <namespace>
```

- Ensure that all required resources are available in your cluster.