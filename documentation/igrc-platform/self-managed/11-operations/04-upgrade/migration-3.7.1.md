# Migration Guide for version 3.7.1

## [DEPR-007] Flat JVM Heap Keys Replaced by `jvm.memory.*`

**Version**: 3.7.1  
**Category**: DEPRECATED

### Overview

JVM heap sizing now has a single entry point per component: `<component>.jvm.memory`. The flat keys `minRAMPercentage` and `maxRAMPercentage` still work as a fallback but are deprecated. Applies to `batch`, `portal`, and `workflow`.

The rename fixes a long-standing mismatch: `minRAMPercentage` always produced `-XX:InitialRAMPercentage`, never `-XX:MinRAMPercentage`. Those are two different JVM flags — `MinRAMPercentage` is a ceiling substitute that applies only to containers of a few hundred megabytes, and the chart never emitted it. The new key name says which flag it sets. Values and behavior are unchanged.

### Old Format

```yaml
batch:
  minRAMPercentage: "50.0"   # → -XX:InitialRAMPercentage (misleading name)
  maxRAMPercentage: "80.0"   # → -XX:MaxRAMPercentage
```

### New Format

```yaml
batch:
  jvm:
    memory:
      initialRAMPercentage: "50.0"   # → -XX:InitialRAMPercentage
      maxRAMPercentage: "80.0"       # → -XX:MaxRAMPercentage
```

Resolution order, most specific first:

```
jvm.memory.initialRAMPercentage  →  minRAMPercentage (deprecated)  →  "50.0"
jvm.memory.maxRAMPercentage      →  maxRAMPercentage (deprecated)  →  "80.0"
```

When both are set, `jvm.memory.*` wins.

### Migration Steps

1. No action required to keep working: the flat keys are still honored, and `DEPR-007` is printed at install/upgrade time while they are in use.
2. Move each `minRAMPercentage` to `jvm.memory.initialRAMPercentage` and each `maxRAMPercentage` to `jvm.memory.maxRAMPercentage` for every component you configure.
3. Verify the rendered result:
   ```bash
   helm template <release> ./igrcanalytics -f my-values.yaml \
     -s templates/data-ingestion/statefulset.yaml | grep -A1 "name: JAVA_OPTS"
   ```

Note: `values.yaml` intentionally ships `jvm: memory: {}` with the effective defaults applied by the chart helper. Shipping them as real values would make the new keys always present and prevent the deprecated flat keys from taking effect.

### Timeline

Deprecated in chart version 3.7.1. The flat keys will be removed in a future version.

## [CHG-005] JVM Heap Options Set Outside `jvm.memory` Are Now Ignored

**Version**: 3.7.1  
**Category**: CHANGED

### Overview

The JVM heap is derived from `resources.limits.memory`. Heap options placed anywhere else are now **dropped from the rendered `JAVA_OPTS`** and reported as `WARN-001` at install/upgrade time.

This closes three defects in the previous behavior:

| Before | Now |
|--------|-----|
| `javaOptions` was filtered, `javaEclipseOptions` was **not** — a heap option placed there survived and produced a `JAVA_OPTS` with two contradictory `-XX:MaxRAMPercentage` values (HotSpot keeps the last) | both lists are filtered identically |
| the filter matched the substring `RAMPercentage` anywhere, silently dropping unrelated options such as `-Dmy.MaxRAMPercentage=1` | matching is on the option prefix, and only heap-sizing options are affected |
| options were dropped with no message at all | every dropped option is named in `WARN-001`, with its values key |

### What Changed

Dropped from `javaOptions` and `javaEclipseOptions`: `-Xmx`, `-Xms`, `-XX:MaxHeapSize`, `-XX:InitialHeapSize`, `-XX:MinHeapSize`, `-XX:MaxRAM*`, `-XX:MinRAM*`, `-XX:InitialRAM*` (including the deprecated `Fraction` forms) and `-XX:±UseContainerSupport`.

Dropped from any component `env` map: `JAVA_OPTS`, `JAVA_TOOL_OPTIONS`, `_JAVA_OPTIONS` — the JVM reads the last two on its own, bypassing the chart.

**Not** affected, because they are not total-heap sizing: `-Xss`, `-Xmn`, `-XX:NewSize`, `-XX:MaxNewSize`, `-XX:MetaspaceSize`, `-XX:MaxDirectMemorySize`, `-XX:ReservedCodeCacheSize`.

There is no opt-out. To reproduce a specific heap, reproduce the pair `resources.limits.memory` + percentage — it reproduces the whole container budget, not just the heap ceiling.

### Migration Steps

1. Search your values files for `-Xmx`, `-Xms` or any `RAMPercentage` inside `javaOptions` / `javaEclipseOptions` / `env`.
2. Move the intent to `<component>.jvm.memory.*` (see `DEPR-007`).
3. Run a `helm upgrade` and read the `WARN-001` box if any option was dropped.

Deployments that never set heap options outside `jvm.memory` — which is the default configuration — are unaffected.

### Also Added

`batch.env`, a last-resort environment escape hatch for the batch-server, matching the existing `gitServer.env` and `operationsManager.env`:

```yaml
batch:
  env:
    MY_VAR: "value"
```

Rendered after the chart-managed variables, so a colliding name wins.

`GOMEMLIMIT` is deliberately **not** guarded — it sizes the Go supervisor, not the JVM — so `batch.env.GOMEMLIMIT` is a supported way to cap the Go runtime.

### Timeline

Shipped in chart version 3.7.1.

## [CHG-004] Batch-Server Liveness Probe Defaults Relaxed

**Version**: 3.7.1  
**Category**: CHANGED

### Overview

Batch-server liveness and readiness probe timings are now configurable via `batch.livenessProbe` and `batch.readinessProbe` (ops-manager pattern: timings from values; `httpGet` path/port remain in the template).

Default **liveness** tolerances are relaxed to reduce false-kill loops under CPU starvation during heavy ingestion plans (SQ-1771):

| Field | Previous (hardcoded) | New default |
|-------|----------------------|-------------|
| `timeoutSeconds` | `1` | `5` |
| `failureThreshold` | `3` | `6` |

Readiness defaults are unchanged (`initialDelaySeconds: 30`, `periodSeconds: 10`, `failureThreshold: 5`, `timeoutSeconds: 1`).

### Old Format

Probes were not configurable; timing was fixed in the chart template.

### New Format

```yaml
batch:
  livenessProbe:
    enabled: true
    initialDelaySeconds: 10
    periodSeconds: 10
    failureThreshold: 6
    timeoutSeconds: 5
  readinessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    failureThreshold: 5
    timeoutSeconds: 1
```

Set `enabled: false` on any probe to omit it from the pod spec. The `enabled` key is chart-only and is never rendered into the Kubernetes probe object.

### Migration Steps

1. Most deployments need no values change — upgraded charts pick up the relaxed liveness defaults automatically.
2. To restore the previous aggressive liveness behavior, set `batch.livenessProbe.timeoutSeconds: 1` and `batch.livenessProbe.failureThreshold: 3`.
3. To disable a probe entirely, set `*.livenessProbe.enabled: false` (or readiness/startup).
4. **Do not run a Helm upgrade while a batch plan is in progress** — changing the StatefulSet recreates `batch-server-0` and interrupts the running plan.

### Timeline

Shipped in chart version 3.7.1.

## [CHG-003] `batch.archiveLogs.maxAge` Default Corrected to Go Duration

**Version**: 3.7.1  
**Category**: CHANGED / FIXED

### Overview

The chart default for `batch.archiveLogs.maxAge` was `7d`. Go `time.ParseDuration` (used by the batch server for `LOG_ARCHIVE_MAX_AGE`) does not accept a day unit, so the invalid default silently disabled age-based archive purge on every deployment that relied on chart defaults (warning: `invalid LOG_ARCHIVE_MAX_AGE '7d', archive age purge disabled`).

The shipped default is now `168h` (7 days), which is a valid Go duration.

### Old Format

```yaml
batch:
  archiveLogs:
    maxAge: 7d
```

### New Format

```yaml
batch:
  archiveLogs:
    maxAge: 168h
```

### Migration Steps

1. If you use chart defaults (no override), upgrade to 3.7.1+ — no values change is required; age-based purge becomes effective automatically.
2. If you copied `7d` (or any `*d` value) into your own values, replace it with a valid Go duration such as `168h` (7 days) or `720h` (30 days). Until you do, age-based purge remains disabled.
3. Day units (`d`) remain unsupported until a companion batch-service parser change lands; prefer hours (`h`) explicitly.
4. Confirm the policy the batch server actually applied:
   ```bash
   kubectl logs -n <namespace> batch-server-0 -c batch-server | grep "log archive retention policy"
   ```
   The `maxAge` field must read `keep younger than 168h0m0s`, not `disabled`.

### Timeline

Default corrected in chart version 3.7.1. Custom `7d` overrides stay invalid until the operator updates them or a future batch-service parser accepts them.
