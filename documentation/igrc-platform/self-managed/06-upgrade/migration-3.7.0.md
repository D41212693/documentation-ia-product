# Migration Guide for version 3.7.0

## `database.port` Deprecated and Ignored

**Version**: 3.7.0  
**Category**: DEPRECATED / CHANGED

### Overview

The chart no longer uses a common `database.port` value. Internal
CloudNativePG databases always listen on PostgreSQL port `5432`, and external
PostgreSQL databases must declare their connection port explicitly with
`database.external.port`.

### Old Format

```yaml
database:
  type: external
  port: 15433
  external:
    host: postgres.example.com
```

### New Format

```yaml
database:
  type: external
  external:
    host: postgres.example.com
    port: 15433
```

### Migration Steps

1. Move any external PostgreSQL port from `database.port` to
   `database.external.port`.
2. Remove `database.port` from your values file. If left in place, the chart
   prints a non-blocking deprecation warning and ignores the value.
3. Do not configure an internal database port. CNPG manages the internal
   PostgreSQL listener on `5432`.

### Timeline

`database.port` is ignored starting in version 3.7.0 and will be removed from
the documented configuration surface in a future release.
