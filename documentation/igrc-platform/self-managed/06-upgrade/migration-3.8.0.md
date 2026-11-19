# Migration Guide for version 3.8.0

## Upgrade Procedure

> [!warning]
> Before upgrading from 3.7.x, read the [FIPS migration guide](./fips-migration.md): it covers the password resets, the values to clean up and the backup to take.

- Upgrade Shared Services to 3.6.0
- Upgrade Identity Analytics to 3.8.0

The changes below are listed from the most recent to the oldest. Check the ones marked BREAKING before upgrading.

## [ADD-012] Keycloak Realm Password Hash Algorithm

**Version**: 3.8.0
**Category**: ADDED / CHANGED

### Overview

`keycloakSettings.passwordHashAlgorithm` (default `pbkdf2-sha512`) is
rendered as `password.hashAlgorithm` in `context.yaml`. Env-manager writes
`hashAlgorithm(<value>)` into the tenant realm `passwordPolicy` when the
realm is created.

On a later `environment-setup` run Keycloak stays authoritative. The patch
does not replace length, history, charset, or an existing hash algorithm.
If the live policy has no `hashAlgorithm(...)` clause, env-manager appends
the configured value and keeps every other rule. Empty
`passwordHashAlgorithm` omits the clause on create and does not add one on
Day-2.

Existing password hashes are not rewritten. Users already stored as argon2
still need a login, admin reset, or the Shared Services FIPS rehash path.

Chart and env-manager ship on the same release train.

### Migration Steps

1. No values change is required. A new realm is created with
   `hashAlgorithm(pbkdf2-sha512)`.
2. After upgrade, the next `environment-setup` Job adds that clause only
   when the live tenant policy has no `hashAlgorithm(...)`. Admin edits
   (algorithm, length, history) are kept.
3. To skip adding a hash clause on create and on Day-2, set
   `keycloakSettings.passwordHashAlgorithm: ""`.
4. Plan password resets for users still on argon2 before a FIPS-strict
   Keycloak switch.

## [CHG-022] Keycloak `passwordMinLength` Must Be at Least 14

**Version**: 3.8.0
**Category**: BREAKING

### Overview

IDA 3.8.0 runs Keycloak in FIPS 140-3 approved-only mode. PBKDF2 rejects
passwords shorter than 14 ASCII characters (112 bits). A realm policy of
`length(8)` lets the UI accept a short password, then hashing fails.

The chart now **refuses to render** when
`keycloakSettings.passwordMinLength` is below 14, so the failure happens at
`helm upgrade` instead of after a user submits a password. The chart default
is already `"16"`. Empty or non-numeric values fail as well. Helm does not
rewrite the value to 14.

### Who is affected

| Case | What happens | Action |
|------|----------------|--------|
| Chart default (`passwordMinLength: "16"`) | Nothing | None |
| Explicit override of `"14"` or higher | Nothing | None |
| Explicit override below 14 (for example `"8"`) | `helm upgrade` fails to render. The cluster is unchanged. | Set `keycloakSettings.passwordMinLength` to `"14"` or higher |
| Live tenant realm already has `length()` below 14 | Raising Helm values does **not** change the live policy | Edit the realm in Keycloak, or delete and recreate the realm |

Day-2 environment setup does not overwrite a live `length()` clause. That is
the same rule as the rest of `passwordPolicy` except `hashAlgorithm(...)`
when it is missing ([ADD-012](#add-012-keycloak-realm-password-hash-algorithm)).

Demo sample user import already generates passwords of
`max(policy length, 14)`, so samples keep working even when the live realm
`length()` is still below the floor.

### Symptom

```
Error: execution error at (igrcanalytics/templates/environment-init/defaults-config.yaml:1:4):
keycloakSettings.passwordMinLength must be at least 14 (got 8): IDA 3.8.0
runs FIPS 140-3 approved-only, and Keycloak PBKDF2 rejects shorter passwords.
Set keycloakSettings.passwordMinLength to "14" or higher (chart default is "16").
```

### Migration Steps

1. Search your values for `keycloakSettings.passwordMinLength`. If it is
   missing, you already get `"16"` and nothing changes.
2. If it is below 14, set it to `"14"` or higher (or remove the key).
3. Dry-run before upgrade:

```bash
helm upgrade --dry-run <release> ./igrcanalytics -n <ns> -f <your-values>
```

4. After a successful upgrade, check the live tenant `passwordPolicy` in
   Keycloak. If it still contains `length(8)` (or any value below 14), raise
   it in the Admin Console, or delete the realm and let environment setup
   recreate it. Changing Helm values alone will not rewrite `length()`.

### Timeline

Ships in 3.8.0. There is no deprecation window: a value below 14 fails at
render.

## [ADD-011] Chart-Wide Container Security Context

**Version**: 3.8.0
**Category**: ADDED / Security

### Overview

`global.security.containerSecurityContext` is a chart-wide container security
context with the same shape as the per-component
`<component>.containerSecurityContext` blocks. When it is enabled, its keys
apply to every container the chart renders, including the containers whose
component block is disabled.

It ships disabled, so **rendered output is unchanged** until you turn it on.
Nothing needs to be migrated to stay on the current behaviour.

| Setting | Default | Effect |
|---------|---------|--------|
| `global.security.containerSecurityContext.enabled` | `false` | when `true`, the block below applies to every container |
| `global.security.containerSecurityContext.allowPrivilegeEscalation` | `false` | the only key set by default |

### Which keys belong here

Only keys that need nothing from the image. `allowPrivilegeEscalation` is one:
it is a kernel flag, not an image property.

`runAsNonRoot`, `runAsUser` and `readOnlyRootFilesystem` are **not**. They
require an image that runs non-root and writes only to its volumes. Portal and
the dedicated workflow still run Tomcat as root
([CHG-019](#chg-019-portal-and-workflow-log-volume-fsgroup-65532)), so setting
those keys globally stops those pods from starting. They belong in the
component blocks, which graduate one component at a time as its image is
rebuilt - see [CHG-008](#chg-008-batch-server-defaults-to-non-root-65532-and-read-only-root)
and [CHG-016](#chg-016-data-recovery-defaults-to-fips-image-and-uid-65532).

### Precedence

The global block and the component block are merged, each contributing only
when it is enabled, and the component wins field by field:

| Global | Component | Result |
|--------|-----------|--------|
| disabled | disabled | no `securityContext` is rendered |
| enabled | disabled | the global keys alone |
| disabled | enabled | the component keys alone |
| enabled | enabled | both, with the component's value for any shared key |

### New Format

```yaml
global:
  security:
    containerSecurityContext:
      enabled: true
      allowPrivilegeEscalation: false
```

Per-container opt-out, where an image needs a setuid binary:

```yaml
portal:
  containerSecurityContext:
    enabled: true
    allowPrivilegeEscalation: true
```

### Migration Steps

1. Render with the block on and review the diff:
   `helm template ... --set global.security.containerSecurityContext.enabled=true`
2. Deploy to a non-production environment and confirm every pod starts.
   `allowPrivilegeEscalation: false` sets `no_new_privs`, which disables setuid
   binaries - a container that calls `sudo` or `su` will fail.
3. Turn it on in production values.

### Timeline

The default may flip to `enabled: true` in a later release once it has been
exercised in the field.

## [CHG-021] Batch Templates Embedded in the Image

**Version**: 3.8.0
**Category**: CHANGED / ADDED

### Overview

The batch-server pod no longer runs the `batch-server-setup` init-container
from `tools.image`. The batch image embeds the seven
`batch-server-properties` templates and renders them itself (IDO `prop`
escape on `.properties`; `igrc_setenv.sh` is unescaped). Startup waits for
config-store health, `eoc_initialized`, and git-server `/probes/readyz`,
then writes `igrc_setenv.sh` to `/tmp/analytics-bin`. The image replaces
`IGRC_HOME/igrc_setenv.sh` with a symlink to that path.

The chart no longer ships ConfigMap `analytics-batch-templates`, ConfigMap
`igrcanalytics-conf-scripts`, `batch-server-setup.sh`, or the
`batch-server-properties/` templates. Batch does **not** use Environment
Manager.

`batch.technicalConfiguration.configOverride` and `projectOverride` are
now applied by the batch image, after each ingestion or purge render of
`config.properties` and `project.properties`. Keys the templates already
emit are skipped with a warning; the run continues. NOTES prints
WARN-005 when either override map is non-empty. The chart stores
them in ConfigMap `igrcanalytics-batch-overrides`, mounted at
`/etc/config/batch-overrides.yaml`.

### Old Format

```yaml
# Init used tools.image + /scripts/batch-server-setup.sh
# Helm mounted /templates and a subPath over IGRC_HOME/igrc_setenv.sh
```

### New Format

```yaml
batch:
  image:
    repository: ""   # must be an image that embeds the templates, the setenv symlink, and override merge
    tag: ""          # defaults to Chart.appVersion
  technicalConfiguration:
    name: docker
    configOverride: {}
    projectOverride: {}
```

`batch.technicalConfig` moved to `batch.technicalConfiguration.name`. See [DEPR-010](#depr-010-batchtechnicalconfig-moved-to-batchtechnicalconfigurationname).

### Migration Steps

1. Upgrade the chart **together with a compatible batch image** (embed +
   symlink `/tmp/analytics-bin/igrc_setenv.sh` + property-override merge).
   An older batch image still expects the Helm templates mount and the
   setup init, and ignores `batch.technicalConfiguration`.
2. The upgrade is **in-place**. The batch pod rolls. **Do not delete
   PVCs** (`batch-logs` is unchanged).
3. Remove unused customizations of `tools.image` that existed only for
   the batch init. Git-server, controller, and database jobs still use
   `tools.image`.
4. Optional: set `batch.technicalConfiguration.configOverride` /
   `projectOverride` for extra Analytics properties. Already-rendered keys
   are skipped (warning in the batch log). An image that does not yet
   merge overrides ignores the file.

### Upgrade impact

- In-place upgrade. No PVC change.
- Compatible batch image is **required**.
- ConfigMaps `analytics-batch-templates` and `igrcanalytics-conf-scripts`
  are removed. ConfigMap `igrcanalytics-batch-overrides` is added.

### Rollback

Revert the chart and the previous batch image together.

### Timeline

Shipped in chart version 3.8.0.

## [DEPR-010] `batch.technicalConfig` Moved to `batch.technicalConfiguration.name`

**Version**: 3.8.0
**Category**: DEPRECATED

### Overview

The config-store batch technical-configuration profile name is
`batch.technicalConfiguration.name` (default `docker`). The sibling
`batch.technicalConfig` key is deprecated.

A leftover `batch.technicalConfig` still seeds the store when
`technicalConfiguration.name` is left at `docker`. Setting
`technicalConfiguration.name` to any other value wins. NOTES prints
DEPR-010 while the old key is present.

### Old Format

```yaml
batch:
  technicalConfig: docker
```

### New Format

```yaml
batch:
  technicalConfiguration:
    name: docker
    configOverride: {}
    projectOverride: {}
```

### Migration Steps

1. Move `batch.technicalConfig` to `batch.technicalConfiguration.name`.
2. Remove `batch.technicalConfig` from your values file.
3. Verify the warning is gone:
   ```bash
   helm template <release> ./igrcanalytics -f my-values.yaml | grep DEPR-010
   ```
   No match is the expected result.

### Timeline

Deprecated in chart version 3.8.0. The old key will be removed in a
future release.

## [CHG-020] Portal and Workflow Init Use Environment Manager

**Version**: 3.8.0
**Category**: CHANGED / ADDED

### Overview

The portal and dedicated workflow-engine init-container no longer runs
`portal-setup.sh` from the `ida-tools` image. It runs
`python /opt/env-init/portal_setup.py` from `envManager.image`. Jinja2
templates for Tomcat properties live in that image. The chart no longer
ships `igrcanalytics-portal-templates` or `portal-setup.sh`.

The `environment-setup` Job is unchanged: it still only provisions
Keycloak and APISIX with `setup.py`. Batch-server no longer uses an
init-container for templates; see [CHG-021](#chg-021-batch-templates-embedded-in-the-image).

`portal.technicalConfiguration.configOverride` and `projectOverride`
are now applied. Keys the templates already emit are skipped with a
warning; the pod keeps running. NOTES prints WARN-005 when either
override map is non-empty.

`DB_SSL_PARAMS` on portal and workflow is the raw JDBC query string
(`?sslmode=verify-ca&sslrootcert=…`). Environment Manager escapes `&`
to `&amp;` when it writes `context.xml`. An image that still writes
values raw would double-escape if the chart kept the old `&amp;` env.

### Old Format

```yaml
# Init used tools.image + /scripts/portal-setup.sh
# portal.technicalConfiguration existed in values.yaml but was unused
```

### New Format

```yaml
envManager:
  image:
    repository: ""   # must be an image that contains portal_setup.py
    tag: ""          # defaults to Chart.appVersion

portal:
  technicalConfiguration:
    configOverride: {}
    projectOverride: {}
```

### Migration Steps

1. Upgrade the chart **together with a compatible Environment Manager
   image** (the image that contains `/opt/env-init/portal_setup.py`).
   An older env-manager image will make the portal init fail.
2. The upgrade is **in-place**. Portal and workflow pods roll. **Do not
   delete PVCs.**
3. Optional: set `portal.technicalConfiguration.configOverride` /
   `projectOverride` for extra Analytics properties. Already-rendered keys
   are skipped (warning in the init log).
4. Remove unused customizations of `tools.image` that existed only for
   the portal init. Batch-server no longer uses `tools.image`
   ([CHG-021](#chg-021-batch-templates-embedded-in-the-image)).

### Upgrade impact

- In-place upgrade. No PVC change.
- Compatible `envManager.image` is **required**.
- ConfigMap `igrcanalytics-portal-templates` is removed. ConfigMap
  `igrcanalytics-portal-overrides` is added.

### Rollback

Revert the chart and the previous Environment Manager image together.

### Timeline

Shipped in chart version 3.8.0.

## [CHG-019] Portal and Workflow Log Volume `fsGroup` 65532

**Version**: 3.8.0
**Category**: CHANGED / Security

### Overview

The portal and dedicated workflow pods now set `fsGroup: 65532` so the
Fluentd sidecar (`ida-fluentd`, uid 65532) can read
Tomcat log files. Tomcat writes those files as `0640` (`UMASK` 0027).
Without a shared group, Fluentd logged `Permission denied @ rb_sysopen`
on `/usr/local/tomcat/logs/igrcportal*.log`.

The portal and workflow **processes stay root**. Only the volume group
changes. This is the same remount pattern as batch `CHG-008`, without
moving the main container off root.

| Setting | Previous default | New default |
|---------|------------------|-------------|
| `portal.podSecurityContext.enabled` | `false` | `true` |
| `portal.podSecurityContext.fsGroup` | (unset) | `65532` |
| `workflow.podSecurityContext.enabled` | `false` | `true` |
| `workflow.podSecurityContext.fsGroup` | (unset) | `65532` |
| `portal.containerSecurityContext.enabled` | `false` | `false` (unchanged) |

### Old Format

```yaml
portal:
  podSecurityContext:
    enabled: false
workflow:
  podSecurityContext:
    enabled: false
```

### New Format

```yaml
portal:
  podSecurityContext:
    enabled: true
    fsGroup: 65532
    fsGroupChangePolicy: Always
workflow:
  podSecurityContext:
    enabled: true
    fsGroup: 65532
    fsGroupChangePolicy: Always
```

### Migration Steps

1. Upgrade the chart. The portal StatefulSet rolls and remounts the
   existing logs PVC. **Do not delete the PVC.**
2. kubelet applies `fsGroup: 65532` with `fsGroupChangePolicy: Always`.
   Files that were `root:root` `0640` become group `65532` and
   group-readable. Root can still write. New files inherit group `65532`
   from the setgid directory.
3. Dedicated workflow (`workflow.dedicated: true`) uses the same
   defaults on its logs volume.
4. On OpenShift restricted-v2, `global.compatibility.openshift.adaptSecurityContext: auto`
   still drops `fsGroup`; the SCC assigns the shared group.
5. To keep the previous empty pod securityContext, set
   `portal.podSecurityContext.enabled: false` (and the matching
   workflow key). Fluentd will not be able to read `0640` portal logs.

### Timeline

Shipped in chart version 3.8.0.

## [REM-001] SQL Tool / SQLPad Removed

**Version**: 3.8.0
**Category**: REMOVED / BREAKING

### Overview

The optional SQL Tool based on SQLPad has been removed from the chart and the
IDA Environment Manager. The `/sqlclient` endpoint is no longer available.

The following resources and configuration have been removed:

- The `sqlPad.*` values
- The `sql-tool` Deployment and Service
- The `sql-tool-configuration` ConfigMap and `sql-tool-connections` Secret
- The SQL Tool-specific database NetworkPolicy and CNPG operator namespace validation
- The APISIX SQL Tool routes and upstream
- The Keycloak SQL Tool authorization resource, policy, and permission

### Upgrade Steps

1. Remove the complete `sqlPad:` block from custom values files. Helm ignores
   this unknown block if it remains, but it has no effect.
2. Upgrade the chart and the `ida-env-manager` image to version 3.8.0 together.
3. Verify that the `sql-tool` workload and `/sqlclient` route are no longer present.

Helm removes the chart-managed Deployment, Service, ConfigMap, Secret, and
NetworkPolicy during upgrade. The Environment Manager rebuilds tenant APISIX
configuration without the SQL Tool routes or upstream. SQLPad used in-memory
state, so no data migration is required.

### Rollback

Rollback requires both the earlier chart and matching Environment Manager
image. Re-enable `sqlPad.enabled` in the earlier chart if the SQL Tool is
required after rollback.

## [CHG-018] Chart-Owned FIPS Tokens in `JAVA_OPTS` (BWIPUAR-3726)

**Version**: 3.8.0
**Category**: CHANGED

### Overview

`--add-modules` and `-DFIPS_ENABLED=true` are no longer set in
`portal.javaEclipseOptions` / `portal.javaOptions` (or the matching workflow
keys). The `java.options` helper always appends them to `JAVA_OPTS`. The
dedicated workflow uses the same tokens as portal.

| Component | Tokens |
|---|---|
| Batch | `--add-modules=org.bouncycastle.fips.pkix,org.bouncycastle.fips.pg` `-DFIPS_ENABLED=true` |
| Portal and dedicated workflow | `--add-modules=org.bouncycastle.fips.pkix` `-DFIPS_ENABLED=true` |

A leftover `batch.image.fips` or `portal.image.fips` map does not omit the
tokens. A custom `image.repository` does not change them.

### Migration Steps

1. Remove `--add-modules=org.bouncycastle.fips.pkix` and `-DFIPS_ENABLED=true`
   from `portal.javaOptions`, `portal.javaEclipseOptions`,
   `workflow.javaOptions`, and `workflow.javaEclipseOptions` if you copied
   them. The chart now injects them.
2. Extra JVM flags still go in `<component>.javaOptions`. Do not set
   `JAVA_TOOL_OPTIONS` or `JDK_JAVA_OPTIONS` in a component `env` map
   ([CHG-011](#chg-011-jdk_java_options-refused-in-component-env-maps)).

### Timeline

Shipped in chart version 3.8.0.

## [CHG-017] Fluentd Without Entrypoint and FIPS Image by Default

**Version**: 3.8.0  
**Category**: CHANGED

### Overview

Fluentd no longer polls `/usr/share/pod/done` inside the image. When `logs.fluentd.enabled` is true, the chart runs Fluentd:

- on portal, the dedicated workflow, git-server, batch-server and extractor: as a regular container, listed after the main container. The pod template sets `kubectl.kubernetes.io/default-container` to the main container, so `kubectl logs` and `kubectl exec` target it without `-c`.
- on the `analytics-update-db` Job: as a Kubernetes native sidecar (`initContainers` + `restartPolicy: Always`). Kubelet sends SIGTERM to Fluentd after the main container exits, so the Job completes. Kubernetes 1.29+ is required for native sidecars.

Long-running workloads do not use a native sidecar because ArgoCD 2.12 and older report such pods as still initializing.

Fluentd flushes its buffers on SIGTERM (`flush_at_shutdown`). The default image is `ida-fluentd` (FIPS build).

| Setting | Previous default | New default |
|---------|------------------|-------------|
| Fluentd placement (StatefulSets, Deployments) | first container | last container |
| Fluentd placement (`analytics-update-db` Job) | regular container | native sidecar init container |
| `logs.fluentd.waitBeforeExit` | used by `entrypoint.sh` | removed |
| Fluentd image | `ida-fluentd` | `ida-fluentd` (FIPS build) |

On a StatefulSet or Deployment, Fluentd receives SIGTERM together with the main container and stops once its buffers are flushed. Lines the main container writes after that stay in the logs volume. On a PVC (portal, workflow, git-server, batch-server), Fluentd ships them when the pod starts again. The extractor logs volume is an `emptyDir`, so its last lines are lost when the pod is deleted.

### New Format

```yaml
logs:
  fluentd:
    enabled: true
    image:
      baseName: fluentd
```

### Migration Steps

1. Upgrade the chart and the Fluentd image together. The new image has no `entrypoint.sh`.
2. When `logs.fluentd.enabled` is true, the cluster Kubernetes version must be 1.29 or newer (`analytics-update-db` Job).
3. Remove `logs.fluentd.waitBeforeExit` and `WAIT_BEFORE_EXIT` overrides; they have no effect.

### Timeline

Shipped in chart version 3.8.0.

## [CHG-016] Data Recovery Defaults to FIPS Image and UID 65532

**Version**: 3.8.0  
**Category**: CHANGED / Security

### Overview

The data-recovery image is a FIPS build (`ida-data-recovery`). The pod and the main containers (`data-recovery-setup` and `data-recovery`) run as Chainguard `nonroot` UID/GID `65532`. Ship **the chart and the FIPS data-recovery image together**.

The downloader sidecar is unchanged.

| Setting | Previous default | New default |
|---------|------------------|-------------|
| Data-recovery image | `ida-data-recovery` | `ida-data-recovery` (FIPS build) |
| `dataRecovery.podSecurityContext.enabled` | `false` | `true` |
| `dataRecovery.podSecurityContext.fsGroup` | (unset) | `65532` |
| `dataRecovery.podSecurityContext.fsGroupChangePolicy` | `Always` | `Always` |
| `dataRecovery.containerSecurityContext.enabled` | `false` | `true` |
| `dataRecovery.containerSecurityContext.runAsUser` / `runAsGroup` | (unset) | `65532` |
| `dataRecovery.containerSecurityContext.readOnlyRootFilesystem` | `true` (inert while disabled) | `true` |

A non-empty `dataRecovery.image.repository` is used as-is. A global `image.suffix` is still appended (for example `ida-data-recovery-mirror` with `image.suffix: mirror`).

### Old Format

```yaml
dataRecovery:
  image:
    baseName: data-recovery
  podSecurityContext:
    enabled: false
  containerSecurityContext:
    enabled: false
```

The previous image ran as UID `1001`. Existing upload PVCs were typically owned by `1001`.

### New Format

```yaml
dataRecovery:
  image:
    baseName: data-recovery
  podSecurityContext:
    enabled: true
    fsGroup: 65532
    fsGroupChangePolicy: Always
  containerSecurityContext:
    enabled: true
    runAsUser: 65532
    runAsGroup: 65532
    runAsNonRoot: true
    readOnlyRootFilesystem: true
```

### Migration Steps

1. Upgrade **chart 3.8.0 and the FIPS data-recovery image in the same release**. Mirror `ida-data-recovery` into a private registry before enabling `dataRecovery.enabled: true`.
2. Keep the existing upload PVC. **Do not delete it.** kubelet remounts it with `fsGroup: 65532` and `fsGroupChangePolicy: Always`.
3. Leave the downloader sidecar as it is. `fsGroup` on the pod also applies to `/work/upload`.

### Timeline

Shipped in chart version 3.8.0.

## [CHG-013] FIPS by Default for Batch, Portal, and Tools

**Version**: 3.8.0  
**Category**: CHANGED

### Overview

Batch, portal, and tools images are FIPS builds by default. Batch and portal keep the same names as 3.7.1 (`ida-batch-server`, `ida-portal`) and have no flavor opt-out. Tools is the only IDA image that still publishes two flavors: the default name is `ida-tools-fips`, and `tools.image.fips.disabled: true` pulls `ida-tools`. Data-recovery follows the same unsuffixed FIPS contract in [CHG-016](#chg-016-data-recovery-defaults-to-fips-image-and-uid-65532). Other IDA images (controller, extract, ops-manager, config, and so on) stay unsuffixed.

The dedicated workflow engine uses `portal.image`.

| Image | Default name | Opt out |
|---|---|---|
| `batch.image` | `ida-batch-server` | none |
| `portal.image` | `ida-portal` | none |
| `tools.image` | `ida-tools-fips` | `tools.image.fips.disabled: true` |

```yaml
tools:
  image:
    fips:
      disabled: false
```

Registry, prefix, suffix, and tag still compose the name (`image.registry`, `image.prefix`, `image.suffix`, and the image `tag`). A non-empty `image.repository` is used as-is. A leftover `batch.image.fips` or `portal.image.fips` map is ignored.

### What the FIPS tools image changes

- `curl`, `psql`, `pg_isready`, `nc`, and `python` use a hardened TLS policy: TLS 1.2 minimum, P-384/P-256 key exchange only, AES-GCM only, no CBC, DHE, or SHA-1. An endpoint that offers nothing else makes the init container fail.
- The OpenSSL legacy provider is disabled: `md5` is rejected, and PostgreSQL password authentication must be `scram-sha-256`.
- The tools user stays UID/GID `1001`.

### Upgrade impact

Pull the FIPS **content** for `ida-batch-server` and `ida-portal` (same names as 3.7.1). Only tools **rolls onto** `ida-tools-fips`. Mirror `ida-batch-server`, `ida-portal`, and `ida-tools-fips` into a private registry before upgrading. A global `image.suffix` (for example `mirror`) is appended after composition (`ida-batch-server-mirror`, `ida-tools-fips-mirror`).

### Rollback

Tools can set `tools.image.fips.disabled: true`, then `helm upgrade`. Batch and portal cannot toggle flavor: pin `image.repository` or roll back the chart.

## [CHG-014] Internal PostgreSQL Pinned to FIPS 18.6

**Version**: 3.8.0  
**Category**: CHANGED

### Overview

An internal database (`database.type: internal`) now sets `spec.imageName` to `rl-postgres-cloudnative-pg-fips:18.6` (composed with `image.registry` and the global `image.suffix`). PostgreSQL stays on the FIPS image. There is no `database.postgres.image.fips.disabled`.

`database.postgres.image.from`:

| Value | Effect |
|---|---|
| `standard` (default) | `spec.imageName` |
| `catalog` | `spec.imageCatalogRef` |
| `""` | omit both; the CloudNativePG operator image is used |

A global `image.suffix` is not applied to this pin (`suffix: ""` on the image block).

### Upgrade impact

The first upgrade that adds `spec.imageName` rolls the CloudNativePG cluster. That roll is the operator's normal image change. It is not the pre-upgrade patch. The patch runs only when switching between `imageName` and `imageCatalogRef`, which cannot be set together.

### Rollback

Set `database.postgres.image.from: ""` to return to the operator default, or pin another image with `standard.version` as a full image string.

## [CHG-015] Pre-upgrade Job Uses the Tools Image

**Version**: 3.8.0  
**Category**: CHANGED

### Overview

The pre-upgrade Job runs `tools.image` (`ida-tools-fips` by default). It no longer uses `tools.kubectl.image`. The script still calls `kubectl`, `jq`, and `yq` inside the tools image.

`tools.kubectl` in your values is ignored. See [DEPR-009](#depr-009-toolskubectl-ignored).

## [ADD-010] Namespaced CNPG ImageCatalog and Image Migration

**Version**: 3.8.0  
**Category**: ADDED

### Overview

Set `database.postgres.image.from: catalog` to point the cluster at an ImageCatalog. When `catalog.kind` is `ImageCatalog`, the chart creates a namespaced catalog named `<namespace>-image-catalog`. The chart never creates a `ClusterImageCatalog`. To use one that already exists, set `kind: ClusterImageCatalog` plus `catalog.name` and `catalog.major`.

`preUpgradeChart.cnpgImageMigration` (default on, internal database only) patches a live cluster when the image source changes between `imageName` and `imageCatalogRef`, then waits until the cluster is healthy (`timeoutSeconds: 900`).

See CNPG ImageCatalog.

## [DEPR-009] `tools.kubectl` Ignored

**Version**: 3.8.0  
**Category**: DEPRECATED

### Overview

`tools.kubectl` is ignored. The pre-upgrade Job uses `tools.image`. A leftover `tools.kubectl` block prints a NOTES warning. The key is not in the chart defaults.

### Old format

```yaml
tools:
  kubectl:
    image:
      repository: docker.io/radiantone/kubectl
      tag: secure
```

Remove that block. The Job image is `tools.image` (`ida-tools-fips` unless `tools.image.fips.disabled` is true).

## [ADD-008] Git Server APIs through Operations Manager

**Version**: 3.8.0  
**Category**: ADDED

### Overview

Operations Manager now proxies Git Server sync, job and latest-archive
download calls under `/ops/git/...`, with the same JWT / ServiceAccount
authentication as the other `/ops/*` façades.

**No action is required on upgrade.** The machine API ships unpublished.
`GET /ops/git/download/project/{git-hash}` is not served.

### New values

| Value | Default | Effect |
|---|---|---|
| `operationsManager.git.api.enabled` | `false` | Publishes the six `/ops/git` paths at the gateway |
| `operationsManager.git.api.limits.request` | `enabled: true`, `rate: 2`, `burst: 5` | APISIX `limit-req` when that plugin is on |
| `operationsManager.git.api.limits.count` | `enabled: false`, `max: 600`, `timeWindow: 3600` | APISIX `limit-count` when that plugin is on |
| `operationsManager.git.api.limits.concurrency` | `enabled: false` | APISIX `limit-conn` when that plugin is on |

The Deployment always sets `GIT_SERVER_API_URL` to the in-cluster Git Server
Service. There is no URL or timeout value; `GIT_SERVER_TIMEOUT` defaults to
120s in the service.

### If you want to drive it from a script

```yaml
operationsManager:
  git:
    api:
      enabled: true
```

Off means *not published*: with the default, the gateway has no route while
the ops-manager Service still serves the paths inside the cluster. Data
recovery hides the gateway route. Allowed on `database.type: external`.

### Verification

```bash
# no token → 401
curl -k https://ops.localtest.me/ops/git/sync/latest
# with an operator token, the latest archive is a zip
curl -k -H "Authorization: Bearer $TOKEN" -o project.zip \
  https://ops.localtest.me/ops/git/download/project
file project.zip
# the by-hash path is 404
curl -k -o /dev/null -w "%{http_code}\n" \
  -H "Authorization: Bearer $TOKEN" \
  https://ops.localtest.me/ops/git/download/project/abc123
```

## [CHG-012] Database Passwords Must Be at Least 14 Characters

**Version**: 3.8.0  
**Category**: BREAKING

### Overview

IDA 3.8.0 runs the portal and batch JVMs in FIPS 140-3 approved-only mode.
PostgreSQL SCRAM-SHA-256 uses PBKDF2. A password shorter than 14 characters
is rejected at connection time. The chart now **refuses to render** when a
clear-text database password in values is shorter than 14 characters, so the
failure happens at `helm upgrade` instead of after the pods start.

Generated passwords are already 32 characters (`common.secrets.passwords.manage`
`length: 32`). Default installs that never set a password are unaffected.

### Who is affected

| Case | What happens | Action |
|------|----------------|--------|
| Default install (empty password keys; chart generates 32-character secrets) | Nothing | None |
| A short password in values (`database.postgres.password`, `database.analytics.password`, `database.shared.password`, `database.analytics.services.aiservice.password`, `database.analytics.services.support.password`, or `database.external.adminPassword` when `autoInit` is on) | `helm upgrade` fails to render. The cluster is unchanged. | Set a password of at least 14 characters, or clear the key so the chart generates one |
| A short password already stored in a Secret (`existingSecret`, `adminExistingSecret`, or a Secret edited by hand) while values stay empty | The upgrade **succeeds**. Portal, batch and the other FIPS clients then fail at runtime. | Rotate the Secret **before** the 3.8.0 upgrade (see below) |

The chart cannot see passwords that live only in Kubernetes Secrets. Those
must be checked with `kubectl` (the loop in the next section prints lengths
only, never the password).

### Symptom

**Render** (short password in values):

```
Error: execution error at (igrcanalytics/templates/database/db-secrets.yaml:7:4):
database.postgres.password must be at least 14 characters (got 13): IDA 3.8.0
runs FIPS 140-3 approved-only, and PostgreSQL SCRAM-SHA-256 (PBKDF2) rejects
shorter passwords. Use a longer password, or leave it empty to let the chart
generate one.
```

**Runtime** (short password already in a Secret): the portal and the batch
JVM fail the PostgreSQL handshake. The exact exception was **inferred**
(not reproduced in this session): BouncyCastle FIPS approved-only rejects
PBKDF2 parameters used by pgJDBC SCRAM-SHA-256. Look for `SCRAM`,
`PBKDF2` or `FipsUnapproved` in the portal and batch logs.

### Before the upgrade: checks (read-only)

Replace `<release>` and `<ns>` with your Helm release and namespace.

1. Look at the live values for the keys listed above:

```bash
helm get values <release> -n <ns>
```

2. Measure the length of every database password the cluster already has.
   This prints `OK` or `TOO SHORT (n)` and never the password itself:

```bash
for s in postgres-credentials analytics-credentials shared-credentials \
         analytics-aiservice-credentials analytics-support-credentials \
         external-db-admin-credentials; do
  if kubectl get secret "$s" -n <ns> >/dev/null 2>&1; then
    n=$(kubectl get secret "$s" -n <ns> -o jsonpath='{.data.password}' | base64 -d | wc -c | tr -d ' ')
    if [ "$n" -ge 14 ]; then echo "$s: OK ($n)"; else echo "$s: TOO SHORT ($n)"; fi
  fi
done
# Add any name from database.external.*.existingSecret or adminExistingSecret.
```

3. Dry-run the 3.8.0 chart so a short password in values fails before you
   touch the cluster:

```bash
helm upgrade --dry-run <release> ./igrcanalytics -n <ns> -f <your-values>
```

### Before the upgrade: rotate on the current (pre-FIPS) release

Do this on the chart version that is already running, so the platform keeps
working while you lengthen the passwords. Stop scheduled execution plans
for the window: between the Secret change and the pod restarts, new
connections from running pods fail (CNPG applies the new password to
PostgreSQL immediately; portal, batch, controller and config do not restart
on their own — they have no checksum on `db-secrets.yaml`).

**Internal database (CNPG)**

1. Set each short key to a password of at least 14 characters. **Removing
   the key from values is not enough**: the chart reuses the existing Secret.
2. `helm upgrade` with the **current** chart version.
3. Wait for CNPG to reconcile the managed roles:

```bash
kubectl get cluster shared-db -n <ns> -o jsonpath='{.status.managedRolesStatus}{"\n"}'
```

4. Restart the workloads that mount the database Secrets:

```bash
kubectl -n <ns> rollout restart sts/portal sts/batch-server sts/controller sts/config
kubectl -n <ns> rollout restart deploy/ops-manager deploy/git-server
# if present:
kubectl -n <ns> rollout restart sts/workflow sts/ai-uar
```

**External database without `autoInit`**

1. `ALTER ROLE … WITH PASSWORD '…'` on the server **first** (14+ characters).
2. Update values or the `existingSecret`.
3. Restart the same workloads.

**External database with `autoInit`**

1. Update values to 14+ character passwords.
2. `helm upgrade` with the current chart. Confirm the init Job ran
   (`kubectl get job -n <ns>`). The init script issues `ALTER USER … WITH
   PASSWORD` for existing users. If the Job did not run, fall back to a
   manual `ALTER ROLE`.
3. Restart the same workloads.

### Upgrade

Upgrade to chart 3.8.0 as usual.

### After the upgrade: verification

- All IDA pods are Ready.
- Portal login succeeds.
- One execution plan or a purge completes.
- Portal and batch logs contain no `SCRAM`, `PBKDF2` or `FipsUnapproved` line.

### Rollback

`helm rollback` to the previous chart works with the **new** (longer)
passwords. Longer passwords are valid on a non-FIPS JVM. Do not revert the
passwords.

### Limitation

A short password that already lives in a Secret, with empty values, is not
detected at render time. Use the `kubectl` length loop above. A later chart
change may add a `lookup`-based NOTES warning for that case.

### Timeline

Shipped in chart version 3.8.0.

## [CHG-011] `JDK_JAVA_OPTIONS` Refused in Component env Maps

**Version**: 3.8.0  
**Category**: CHANGED

### Overview

The FIPS batch, portal and workflow images set `JDK_JAVA_OPTIONS` in the
container: BouncyCastle FIPS add-exports, `-Djavax.net.ssl.trustStoreType=FIPS`,
and, on the batch, `@/usr/local/igrcanalytics/osgi-system-packages.args` so
Equinox has an explicit system-package list without `org.bouncycastle.*`.
The `java` launcher reads that variable on its own. A value set through
`<component>.env` would **replace** the image value, disable FIPS settings
and, on the batch, make Eclipse exit 13 (OSGi uses-constraint violation on
`org.bouncycastle.asn1`).

The chart already refused `JAVA_OPTS`, `JAVA_TOOL_OPTIONS` and `_JAVA_OPTIONS`
from a component `env` map ([CHG-005](#chg-005-jvm-heap-options-set-outside-jvmmemory-are-now-ignored)).
`JDK_JAVA_OPTIONS` is now refused the same way and reported by `WARN-001`.

`batch.javaOptions` and `batch.javaEclipseOptions` are unchanged. Extra JVM
flags still go there: they are appended to `JAVA_OPTS` and combine with the
image `JDK_JAVA_OPTIONS`. Do not copy the portal `--add-modules` /
`--add-opens` flags onto the batch; they do not fix the OSGi clash.

### What Changed

| Before | Now |
|--------|-----|
| `batch.env.JDK_JAVA_OPTIONS` was rendered last and replaced the image value | the key is dropped from the container env and named in `WARN-001` |
| `WARN-001` read as heap-only | the box also covers the four JVM env keys and the FIPS image setup |

### Migration Steps

1. Search your values files for `JDK_JAVA_OPTIONS` under `batch.env`,
   `portal.env` or `workflow.env`.
2. Move any extra flags to `<component>.javaOptions` (or
   `<component>.javaEclipseOptions`). Heap sizing stays in
   `<component>.jvm.memory`.
3. Run a `helm upgrade` and read the `WARN-001` box if the key was dropped.

Deployments that never set `JDK_JAVA_OPTIONS` in an env map — which is the
default configuration — are unaffected.

### Timeline

Shipped in chart version 3.8.0.

## [CHG-010] Portal and Workflow JVM Exit on the First Thread OutOfMemoryError

**Version**: 3.8.0  
**Category**: CHANGED

### Overview

The same OutOfMemoryError policy that [CHG-009](#chg-009-batch-jvm-exits-on-the-first-thread-outofmemoryerror)
gave the batch Eclipse JVM now applies to the portal Tomcat and to a
dedicated workflow engine. The first worker-thread
`java.lang.OutOfMemoryError` terminates the process. Kubernetes restarts
the pod.

The policy is a first-class value, not tokens in `javaOptions`. The chart
emits **exactly one** of Exit or Crash.

### New default

```yaml
portal:                    # same shape under workflow when dedicated
  jvm:
    outOfMemory:
      onError: exit          # -XX:+ExitOnOutOfMemoryError
      heapDump:
        enabled: false
```

`ErrorFile` is always injected at
`/usr/share/tomcat/logs/hs_err_pid%p.log`. In the image,
`/usr/local/tomcat` is a symlink to `/usr/share/tomcat`; the existing
`logs` PVC (mounted at `/usr/local/tomcat/logs`) is that directory. Exit
typically does not write the file for this OOM; `onError: crash` does.
The path also covers other HotSpot fatals (SIGSEGV, JNI).

When `workflow.dedicated` is `false`, only `portal.jvm.outOfMemory`
applies (one Tomcat). A dedicated workflow engine uses
`workflow.jvm.outOfMemory`.

| `onError` | JVM flag | Process | Diagnostic |
|-----------|----------|---------|------------|
| `exit` (default) | `-XX:+ExitOnOutOfMemoryError` | `os::exit(3)` | stdout `Terminating due to java.lang.OutOfMemoryError` |
| `crash` | `-XX:+CrashOnOutOfMemoryError` | SIGABRT / 134 | `hs_err_pid*.log` (allocating thread + stack) |

Heap dump stays **off**. A `.hprof` is the size of the live heap and
`portal.pvc.logs` / `workflow.pvc.logs` default to 16Gi. Enable it only
with `onError: exit` and a volume that can hold at least one full dump.
NOTES prints `WARN-002` when it is on. `heapDump.enabled: true` with
`onError: crash` fails the render.

Do not put `-XX:+ExitOnOutOfMemoryError`, `-XX:+CrashOnOutOfMemoryError`,
`-XX:ErrorFile` or `-XX:+HeapDumpOnOutOfMemoryError` in `javaOptions`.
Those tokens are dropped (`WARN-003`); configure
`<component>.jvm.outOfMemory`. An override of `javaOptions` no longer
removes the policy.

None of these flags save a pod whose container hits the cgroup memory
limit (`OOMKilled`). Keep the heap under `resources.limits.memory` — see
JVM Memory Sizing.

The rotating GC log is the same key under `portal.jvm.gcLog` /
`workflow.jvm.gcLog` ([ADD-007](#add-007-rotating-gc-log)).

### Rollback

There is no supported return to “Tomcat stays alive after a thread OOM”.
`onError: crash` only changes the diagnostic (writes `hs_err`). Upgrade
in place; no PVC or secret change.

## [CHG-009] Batch JVM Exits on the First Thread OutOfMemoryError

**Version**: 3.8.0  
**Category**: CHANGED

### Overview

The batch Eclipse JVM used to stay alive after a worker thread died with
`java.lang.OutOfMemoryError`. The Go supervisor kept heartbeating and the
controller stayed Running (client run 51). The chart now makes that first
OOM fatal for the process, which `Wait()` already treats as failure.

The policy is a first-class value, not tokens in `javaOptions`. The chart
emits **exactly one** of Exit or Crash — they are two responses to the same
event, and combining them is not a documented HotSpot contract.

### New default

```yaml
batch:
  jvm:
    outOfMemory:
      onError: exit          # -XX:+ExitOnOutOfMemoryError
      heapDump:
        enabled: false
```

`ErrorFile` is always injected at
`/var/log/igrcanalytics/batch/current/hs_err_pid%p.log` (archived at
teardown). Exit typically does not write that file for this OOM; `onError:
crash` does. The path also covers other HotSpot fatals (SIGSEGV, JNI).

| `onError` | JVM flag | Process | Diagnostic |
|-----------|----------|---------|------------|
| `exit` (default) | `-XX:+ExitOnOutOfMemoryError` | `os::exit(3)` | stdout `Terminating due to java.lang.OutOfMemoryError` |
| `crash` | `-XX:+CrashOnOutOfMemoryError` | SIGABRT / 134 | `hs_err_pid*.log` (allocating thread + stack) |

Heap dump stays **off**. A `.hprof` is the size of the live heap (often
tens of Gi) and `batch.pvc.zipLogs` defaults to 16Gi. Enable it only with
`onError: exit` and a volume that can hold at least one full dump plus
existing archives. NOTES prints `WARN-002` when it is on. `heapDump.enabled:
true` with `onError: crash` fails the render.

Applies to every script that reads `JAVA_OPTS` (`igrc_batch.sh`,
`igrc_purge.sh`, `igrc_cli.sh`, `igrc_view.sh`). A secondary-thread OOM now
stops the whole plan, including setup and recon.

Do not put `-XX:+ExitOnOutOfMemoryError`, `-XX:+CrashOnOutOfMemoryError`,
`-XX:ErrorFile` or `-XX:+HeapDumpOnOutOfMemoryError` in `javaOptions`.
Those tokens are dropped (`WARN-003`); configure `batch.jvm.outOfMemory`.
An override of `javaOptions` no longer removes the policy.

None of these flags save a run whose container hits the cgroup memory
limit (`OOMKilled`). Keep the heap under `resources.limits.memory` — see
JVM Memory Sizing.

### Rollback

There is no supported return to “the JVM stays alive after a thread OOM”:
that hang is the bug this change closes. `onError: crash` only changes the
diagnostic (writes `hs_err`). Upgrade in place; no PVC or secret change.

If an earlier 3.8.0 draft put Exit/Crash/`ErrorFile` tokens in
`batch.javaOptions`, remove them — the chart now injects the policy from
`batch.jvm.outOfMemory` and drops those tokens (`WARN-003`).

## [ADD-007] Rotating GC Log

**Version**: 3.8.0  
**Category**: ADDED

Opt-in GC + safepoint log for a deep heap diagnosis. Sibling of
`<service>.jvm.outOfMemory`, not a field of `heapDump`: the file is
written throughout the process and capped at 5 × 20 Mi. Off by default.
See
JVM Memory Sizing.

```yaml
batch:
  jvm:
    gcLog:
      enabled: true
      fileCount: 5          # default
      fileSize: "20M"       # default

portal:                    # same key under workflow when dedicated
  jvm:
    gcLog:
      enabled: true
```

| Component | File |
|-----------|------|
| `batch` | `/var/log/igrcanalytics/batch/current/gc-%p.log` |
| `portal`, `workflow` | `/usr/share/tomcat/logs/gc-%p.log` |

The chart injects
`-Xlog:gc*,safepoint:file=<path>:time,level,tags:filecount=<fileCount>,filesize=<fileSize>`.
Tokens starting with `-Xlog:gc` in `javaOptions` are dropped (`WARN-004`).

No PVC or secret change. Disable the key to roll back.

## [ADD-006] Standalone Timeslot Purge and Post-Plan Opt-Out

**Version**: 3.8.0  
**Category**: ADDED

### Overview

A purge is now a run of its own. Until 3.8.0 timeslots were deleted only as the
tail of a **scheduled** execution plan; 3.8.0 adds a purge that an operator, a
cron or an authenticated script can start at any time, and makes the two
post-plan steps optional. Full operator manual:
Timeslot Purge.

**No action is required on upgrade.** The shipped defaults reproduce 3.7
behaviour exactly: a scheduled execution plan still removes hidden timeslots and
then applies the retention policy, a plan started by hand still purges nothing,
and no purge runs on a schedule. The machine API ships unpublished.

### New values

| Value | Default | Effect |
|---|---|---|
| `purge.mode` | `policy` | What a standalone purge runs: `policy` or `legacy` |
| `purge.legacy.unit` | `timeslots` | `timeslots`, `days` or `states` |
| `purge.legacy.value` | `"10"` | Count to keep, or states to delete (`I` `W` `A` `D`) |
| `purge.afterExecutionPlan.hidden` | `true` | A scheduled plan removes hidden timeslots |
| `purge.afterExecutionPlan.policy` | `true` | A scheduled plan applies the retention policy |
| `scheduling.purge.cron` | `""` | Standalone purge schedule, evaluated in **UTC** |
| `operationsManager.purge.api.enabled` | `false` | Publishes `/ops/purge/runs` at the gateway |
| `operationsManager.executionPlan.api.enabled` | `false` | Publishes `/ops/execution-plan/runs` |

The five `purge.*` values are written to the configuration store **at install
time only**. An upgrade never overwrites them, so an operator who changes them
in the Config UI keeps their choice across upgrades — and setting them in
`values.yaml` after the first install has no effect. Change them from the Config
UI **Purge** page instead.

### What changes without any action

- The Controller's *Execution plans* page gains a **Type** column, and its rows
  now name who asked for the run.
- The Config UI gains a **Purge** page and a fourth switch on **Scheduling**.
- A run stopped by somebody is reported as **Aborted** and sends no mail; a run
  that died on its own is reported as **Error**. Before 3.8.0 both read `Error`.
- An execution plan and a purge share one slot: whichever arrives second is
  refused with a message naming the holder.

### If you want a purge on a schedule

```yaml
scheduling:
  purge:
    cron: "0 2 * * *"   # 02:00 UTC
```

The cron is evaluated in **UTC**, like the three schedules that already existed.
The Config UI prints both clocks under each schedule.

### If you want to drive it from a script

```yaml
operationsManager:
  purge:
    api:
      enabled: true
      limits:
        count: { enabled: true, max: 200, timeWindow: 3600 }
```

Off means *not published*: with the default, the gateway answers `403` while the
ops-manager Service still serves the route inside the cluster. Raise
`limits.count.max` on environments used for integration work — the limit counts
refused calls as well as accepted ones.

### Verification

```bash
# the values the chart will seed
helm template <release> ./igrcanalytics -f my-values.yaml | grep -A6 igrc_purge

# what is in force after the upgrade
kubectl -n <ns> exec config-0 -c config -- \
  wget -qO- http://localhost:8080/api/get_config/igrc_purge
```

Expected on an upgrade from 3.7 with no action: `igrc_purge_after_plan_hidden`
and `igrc_purge_after_plan_policy` both `true`.

## [CHG-008] Batch-Server Defaults to Non-Root 65532 and Read-Only Root

**Version**: 3.8.0  
**Category**: CHANGED / Security

### Overview

Chart 3.8.0 turns on batch-server security contexts by default and always
mounts a writable `/home/java`. This matches the 3.8.0 batch image
(`USER 65532`, Chainguard `java`). Ship **the chart and that image
together**.

| Setting | Previous default | New default |
|---------|------------------|-------------|
| `batch.podSecurityContext.enabled` | `false` | `true` |
| `batch.podSecurityContext.fsGroup` | (unset) | `65532` |
| `batch.podSecurityContext.fsGroupChangePolicy` | `Always` | `Always` |
| `batch.containerSecurityContext.enabled` | `false` | `true` |
| `batch.containerSecurityContext.runAsUser` / `runAsGroup` | (unset) | `65532` |
| `batch.containerSecurityContext.readOnlyRootFilesystem` | `true` (inert while disabled) | `true` |

`batch.containerSecurityContext` applies to the `batch-server` container.
Fluentd is unchanged and has no security context from this block. Chart
3.8.0 later removed the `batch-server-setup` init
([CHG-021](#chg-021-batch-templates-embedded-in-the-image)).

`/home/java` is always rendered on `batch-server` from the existing `temp`
emptyDir (`subPath: java-home`). A values-only change cannot add that mount
on chart 3.7.2; upgrading the chart version is required.

### Old Format

```yaml
batch:
  podSecurityContext:
    enabled: false
  containerSecurityContext:
    enabled: false
```

The 3.7.2 image ran as root. Existing `batch-logs` volumes were typically
`root:root` `0755`.

### New Format

```yaml
batch:
  podSecurityContext:
    enabled: true
    fsGroup: 65532
    fsGroupChangePolicy: Always
  containerSecurityContext:
    enabled: true
    runAsUser: 65532
    runAsGroup: 65532
    runAsNonRoot: true
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    seccompProfile:
      type: RuntimeDefault
```

### Migration Steps

1. Upgrade **chart 3.8.0 and batch image 3.8.0 in the same release**. Do not
   run a Helm upgrade while a batch plan is in progress — the StatefulSet
   roll recreates `batch-server-0`.
2. Keep the existing `batch-logs` PVC. **Do not delete it.** kubelet remounts
   gp3/ext4 with `fsGroup: 65532` and `fsGroupChangePolicy: Always`. A volume
   that was `root:root` `0755` becomes `root:java` setgid (`drwxrwsr-x`).
   The `java` user can write. This volume is RWO gp3, not EFS.
3. If you upgrade the chart without the 3.8.0 image, set
   `batch.podSecurityContext.enabled: false` and
   `batch.containerSecurityContext.enabled: false` (or stay on 3.7.2 until
   both move).
4. Leave fluentd unrestricted. Empty DSM on import is a business miss, not a
   security-context failure.

### Timeline

Shipped in chart version 3.8.0.

## [CHG-007] DB Check and DB Maintenance Unavailable on External Databases

**Version**: 3.8.0  
**Category**: CHANGED

### Overview

DB Check and DB Maintenance run only against the built-in CloudNativePG
cluster (`database.type: internal`, the default). When `database.type` is
`external`, those capabilities are **off**. There is no operator override and
no new Helm value. `operationsManager.enabled` remains ignored ([DEPR-008](#depr-008-operationsmanagerenabled-deprecated-and-ignored)).

On a fresh external install:

- ops-manager does not register `/ops/db/check*` or `/ops/db/maintenance*`
  (ClusterIP returns 404)
- Helm does not render Job or catalogue ConfigMaps
- Environment Manager does not create Keycloak `ida-ops-api` (client, `operator`
  role, grants, admin user role, `ida-ops-manager` audience mapper)
- Environment Manager does not create APISIX ops UI routes, M2M routes, or the
  ops-manager upstream
- Config UI hides the Database section
- After a data-recovery import, `start_maintenance.sh` prints a DBA message
  and does not start a Job

Helm refuses to render if `operationsManager.dbCheck.api.enabled` or
`operationsManager.dbMaintenance.api.enabled` is `true` on an external
database. Those flags default to `false`.

Switching an existing install from internal to external does not delete a
leftover `ida-ops-api` client. That cleanup is out of scope.

This chart version requires matching images for ops-manager, env-manager,
config-server, configui, and data-recovery so each component honours
`database.type`.

### Operator action

**None** when you already use the built-in database. If you use an external
PostgreSQL server, do not provision `ops` / `pg_monitor` / SQL helper functions
/ `ida-ops-api` for DB Check. Ask the DBA to run `VACUUM ANALYZE` (and
optionally `VACUUM FULL` / `REINDEX`) when you need maintenance.

## [CHG-006] OIDC Client Secrets Persist Across Helm Upgrades

**Version**: 3.8.0  
**Category**: CHANGED / Fixed

### Overview

On every `helm upgrade`, the chart regenerated OIDC client secrets for APISIX,
the extractor, and the Portal API (and the APISIX OIDC session secret). The
helpers looked up a Secret named `keycloak-app-realm-credentials`, which the
chart never creates. The live Secret is `keycloak-app-realm-admin-credentials`.
The empty lookup fell through to `randAlphaNum`, so Keycloak clients and APISIX
OIDC sessions were rotated unintentionally.

The ops-manager M2M client was already looking up the correct Secret and was
not affected.

`keycloakSettings.portalAPI.clientSecret` was also ignored: the portal-api
helper wrongly fell back to `keycloakSettings.extractor.clientSecret`. NOTES
told operators to read the non-existent Secret name for the initial admin
password.

### Operator action

**None required.** After upgrading to a chart that includes this fix, the next
`helm upgrade` reuses the values already stored in
`keycloak-app-realm-admin-credentials`. Subsequent upgrades keep those same
secrets.

To retrieve the initial technical admin password, use the corrected Secret
name (also shown in NOTES):

```bash
kubectl get secret -n <namespace> keycloak-app-realm-admin-credentials \
  -o jsonpath="{.data.technical_admin_initial_password}" | base64 -d
```

## [DEPR-008] `operationsManager.enabled` Deprecated and Ignored

**Version**: 3.8.0  
**Category**: DEPRECATED / CHANGED

### Overview

Operations Manager is a required component (portal lifecycle, database
maintenance, DB Check, environment cleanup). The `operationsManager.enabled`
flag no longer gates it: the service is always deployed. The key is ignored
and prints a non-blocking warning at install/upgrade time.

`enabled: false` does **not** omit the component. To stop the pod, use
`global.hibernate: true` (replicas set to 0), the same as the other always-on
services.

### Old Format

```yaml
operationsManager:
  enabled: true   # or false — both ignored
```

### New Format

```yaml
operationsManager:
  image:
    baseName: ops-manager
  # no enabled key
```

### Migration Steps

1. Remove `operationsManager.enabled` from your values file.
2. If you used `enabled: false` to keep the component off, that is no longer
   supported. Scale to zero with `global.hibernate: true` if you need the pod
   stopped; otherwise leave the service running.
3. Verify the warning is gone:
   ```bash
   helm template <release> ./igrcanalytics -f my-values.yaml | grep DEPR-008
   ```
   No match is the expected result.

### Timeline

Deprecated and ignored in chart version 3.8.0. The key will be removed from
the documented configuration surface in a future release.

## [ADD-005] `ops` Schema in `shared_db` for DB Check

**Version**: 3.8.0  
**Category**: ADDED

### Overview

DB Check and DB Maintenance store their audit trails and report artifacts in a
new `ops` schema in `shared_db`, the fourth schema-per-component entry after
`controller`, `configuration` and `gitsync`. One migration file
(`0001_ops_schema.sql`) creates `ops.db_check_run` (including `sections_failed`)
and `ops.db_maintenance_run`. There is no separate ADD-006.

**No action is required on upgrade.** An `init-db-schema` init container on the
ops-manager Deployment creates the schema on every rollout, and
`CREATE SCHEMA IF NOT EXISTS` is idempotent, so a fresh install and an existing
one converge on the same path. ops-manager then creates its own tables through
versioned migrations. On an external database with
`database.external.autoInit: true`, `ops` is also part of the shared-schema
loop in the init Job. **DB Check and DB Maintenance are not available on an
external database** ([CHG-007](#chg-007-db-check-and-db-maintenance-unavailable-on-external-databases)):
you do not need `ops`, `pg_monitor`, SQL helper functions, or Keycloak
`ida-ops-api` for those features. A leftover `ops` schema from init is unused
for product screens.

### Configuration

```yaml
database:
  shared:
    schemas:
      ops: ops
```

### Two behaviours worth knowing

**The init container is best effort.** Unlike the equivalent container on
config, git-server and controller, it waits for the database for a bounded time
and then exits successfully whatever happened. ops-manager also serves portal
restart and environment cleanup, so an unreachable `shared_db` must not hold the
pod in `Init`. If the schema could not be created, DB Check and DB Maintenance report
unavailable and the rest of ops-manager keeps working; it recovers on its own
once the schema exists, without a pod restart.

**`shared_user`'s default `search_path` is not extended on an existing
installation.** The statement that sets it runs only when the CNPG cluster is
first created, so an upgraded installation keeps
`search_path=controller, configuration, gitsync`. This does not affect the
product: ops-manager pins `search_path` on its own connections. It affects only
a manual `psql` session as `shared_user`, which has to qualify `ops.`
explicitly:

```sql
SELECT id, status, started_at FROM ops.db_check_run ORDER BY started_at DESC;
```

### Migration Steps

1. Upgrade normally. No values change is required.

   On an **internal** database the upgrade grants `pg_monitor` to the analytics
   user, through the `Cluster` managed roles, so DB Check reports full settings,
   statistics and query text instead of flagging the privilege as missing in every
   report's preflight. Know what it widens: `pg_monitor` is instance-wide and that
   user is the application's, so it gains read access to settings, statistics and
   query text in every database on the server. To decline it, remove the check
   user's `inRoles` from `templates/database/cluster.yaml`; the report then keeps
   working and keeps saying which sections are thin.
2. On an **external** database, skip DB Check setup. The capabilities are not
   available ([CHG-007](#chg-007-db-check-and-db-maintenance-unavailable-on-external-databases)).
   You do not need `ops`, `pg_monitor`, helper functions, or `ida-ops-api`.
3. If your **internal** check user holds no `CREATE` on the ledger schema, install the five
   helper functions once per analytics database. Where it does hold `CREATE`, the
   Job installs them itself before each run. An installation upgraded from an
   earlier version usually already has them, left by the previous monolithic
   check script; the preflight section at the top of any report says which are
   missing.
4. If DB Check or DB Maintenance reports itself unavailable on an **internal**
   database, check the
   `init-db-schema` init container logs on the ops-manager pod and confirm the
   schema exists in `shared_db`.
