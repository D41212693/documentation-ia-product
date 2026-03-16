# Migration Guide for version 3.5.1

## Portal API Route Changes

**Version**: 3.5.1+

### Overview

The Portal API APISIX routes have been restructured to expose additional backend endpoints. The previous catch-all `/api/*` route has been replaced by three specific routes.

### Breaking Change

The `/api/*` catch-all route no longer exists. Clients must update their API paths.

| Old Path          | New Path                           | Method | Backend                           |
| ----------------- | ---------------------------------- | ------ | --------------------------------- |
| `/api/{resource}` | `/api/views/{resource}`            | GET    | `/ws/results/{resource}`          |
| N/A               | `/api/compliancereport/{resource}` | GET    | `/ws/compliancereport/{resource}` |

### Migration Steps

Update all API client calls that use `/api/`:

**Before:**

```
GET /api/my-view-name
```

**After:**

```
GET /api/views/my-view-name
```

Authentication (bearer token), rate limiting, and upstream (portal:internal) remain unchanged.

---

## From `aiservice.*` to `aida.*` + `aiuar.*` + `litellmProxy.*`

**Version**: 3.5.1+

### Overview

The AI service configuration has been restructured to:

1. Introduce a **master flag** (`aida.enabled`) controlling all AI services
2. **Rename** `aiservice` to `aiuar` (AI User Access Review) to align with IDO naming
3. **Extract** LiteLLM Proxy configuration to a shared top-level `litellmProxy` section
4. Support **sidecar and external** LiteLLM Proxy modes

### Backward Compatibility

**`aiservice.enabled` is deprecated and IGNORED.** You must use `aida.enabled: true` instead. A deprecation warning is shown during `helm install/upgrade` if `aiservice.enabled` is detected.

Other legacy `aiservice.*` configuration values (models, litellm, image, etc.) continue to work with deprecation warnings. Migrate these at your convenience.

### Dev Overrides

You can selectively disable individual AI services during development while keeping `aida.enabled: true`:

```yaml
aida:
  enabled: true
  dev:
    aiuar:
      enabled: false # Disables AI UAR only
```

If `aida.dev.aiuar.enabled` is not set, AI UAR follows `aida.enabled`.

### Migration Steps

#### Step 1: Enable AIDA with the new master flag

**Before:**

```yaml
aiservice:
  enabled: true
```

**After:**

```yaml
aida:
  enabled: true
```

#### Step 2: Move AI service configuration to `aiuar.*`

**Before:**

```yaml
aiservice:
  image:
    baseName: aiservice
    repository: ""
    tag: ""
  logLevel: "INFO"
  env: {}
  models:
    task: "my-model"
    nl: "my-model"
  replicas: 1
  resources:
    limits:
      cpu: "1"
      memory: 1Gi
    requests:
      cpu: "1"
      memory: 512Mi
  pvc:
    logs: 16Gi
  podSecurityContext:
    enabled: true
    fsGroup: 1000
  containerSecurityContext:
    enabled: false
```

**After:**

```yaml
aiuar:
  image:
    baseName: aiuar
    repository: ""
    tag: ""
  logLevel: "INFO"
  env: {}
  models:
    task: "my-model"
    nl: "my-model"
  replicas: 1
  resources:
    limits:
      cpu: "1"
      memory: 1Gi
    requests:
      cpu: "1"
      memory: 512Mi
  pvc:
    logs: 16Gi
  podSecurityContext:
    enabled: true
    fsGroup: 1000
  containerSecurityContext:
    enabled: false
```

#### Step 3: Move LiteLLM configuration to `litellmProxy.*`

**Before:**

```yaml
aiservice:
  litellm:
    logLevel: "INFO"
    env: {}
    settings:
      general:
        master_key: os.environ/LITELLM_PROXY_API_KEY
      litellm:
        set_verbose: false
        json_logs: true
    masterKey:
      existingSecret:
        name: ""
        key: ""
      value: ""
    providers:
      aws:
        models:
          - name: my-model
            provider: bedrock/converse/my-model
        credentials:
          enabled: true
          accessKeyId: "AKID..."
          secretAccessKey: "SECRET..."
    image:
      baseName: litellm
      registry: docker.io/radiantone
      tag: main-stable
    resources:
      limits:
        cpu: 1
        memory: 1Gi
```

**After:**

```yaml
litellmProxy:
  logLevel: "INFO"
  env: {}
  settings:
    general:
      master_key: os.environ/LITELLM_PROXY_API_KEY
    litellm:
      set_verbose: false
      json_logs: true
  masterKey:
    existingSecret:
      name: ""
      key: ""
    value: ""
  sidecar:
    enabled: true
  providers:
    aws:
      models:
        - name: my-model
          provider: bedrock/converse/my-model
      credentials:
        enabled: true
        accessKeyId: "AKID..."
        secretAccessKey: "SECRET..."
  image:
    baseName: litellm
    registry: docker.io/radiantone
    tag: main-stable
  resources:
    limits:
      cpu: 1
      memory: 1Gi
```

#### Step 4 (Optional): Use external LiteLLM Proxy

To point to an existing LiteLLM Proxy instance instead of running a sidecar:

```yaml
litellmProxy:
  sidecar:
    enabled: false
  external:
    url: "http://litellm-proxy.shared-services.svc.cluster.local:4000"
```

### Key Mapping Reference

| Old Path                             | New Path                         | Notes                                                 |
| ------------------------------------ | -------------------------------- | ----------------------------------------------------- |
| `aiservice.enabled`                  | `aida.enabled`                   | Master flag for all AI services (deprecated, ignored) |
| `aiservice.image.*`                  | `aiuar.image.*`                  |                                                       |
| `aiservice.models.*`                 | `aiuar.models.*`                 |                                                       |
| `aiservice.logLevel`                 | `aiuar.logLevel`                 |                                                       |
| `aiservice.env`                      | `aiuar.env`                      |                                                       |
| `aiservice.resources`                | `aiuar.resources`                |                                                       |
| `aiservice.replicas`                 | `aiuar.replicas`                 |                                                       |
| `aiservice.pvc`                      | `aiuar.pvc`                      |                                                       |
| `aiservice.affinity`                 | `aiuar.affinity`                 |                                                       |
| `aiservice.tolerations`              | `aiuar.tolerations`              |                                                       |
| `aiservice.podSecurityContext`       | `aiuar.podSecurityContext`       |                                                       |
| `aiservice.containerSecurityContext` | `aiuar.containerSecurityContext` |                                                       |
| `aiservice.litellm.settings.*`       | `litellmProxy.settings.*`        |                                                       |
| `aiservice.litellm.masterKey.*`      | `litellmProxy.masterKey.*`       |                                                       |
| `aiservice.litellm.providers.*`      | `litellmProxy.providers.*`       |                                                       |
| `aiservice.litellm.image.*`          | `litellmProxy.image.*`           |                                                       |
| `aiservice.litellm.resources`        | `litellmProxy.resources`         |                                                       |
| `aiservice.litellm.logLevel`         | `litellmProxy.logLevel`          |                                                       |
| `aiservice.litellm.env`              | `litellmProxy.env`               |                                                       |
| N/A                                  | `litellmProxy.sidecar.enabled`   | New: sidecar mode toggle                              |
| N/A                                  | `litellmProxy.external.url`      | New: external proxy URL                               |
| N/A                                  | `aiuar.subPath`                  | New: APISIX route sub-path (default: `aiservice`)     |

### Template Directory Changes

| Old Path                                      | New Path                                            |
| --------------------------------------------- | --------------------------------------------------- |
| `templates/aida/statefulset.yaml`             | `templates/ai-uar/statefulset.yaml`                 |
| `templates/aida/service.yaml`                 | `templates/ai-uar/service.yaml`                     |
| `templates/aida/service-headless.yaml`        | `templates/ai-uar/service-headless.yaml`            |
| `templates/aida/litellm-config.yaml`          | `templates/litellm-proxy/config.yaml`               |
| `templates/aida/litellm-api-key.yaml`         | `templates/litellm-proxy/api-key.yaml`              |
| `templates/aida/ai-provider-credentials.yaml` | `templates/litellm-proxy/provider-credentials.yaml` |

### What Stays the Same

The following are deliberately unchanged for backward compatibility:

- **Kubernetes resource names**: `aiservice`, `aiservice-headless` (StatefulSet, Service)
- **PVC name**: `aiservice-logs`
- **Secret names**: `litellm-api-key`, `ai-provider-credentials`, `analytics-aiservice-credentials`
- **APISIX route path**: `/aiservice/*`
- **Database schema/user references**: `database.analytics.services.aiservice.*`, `database.analytics.schemas.aiservice`
- **Kubernetes labels**: `app.kubernetes.io/component: aiservice`
