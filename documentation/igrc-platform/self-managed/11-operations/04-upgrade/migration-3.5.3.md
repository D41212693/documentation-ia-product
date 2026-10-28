# Migration Guide for version 3.5.3

This procedure describes the removal of the ArgoCD operator previously used by IDA for maintenance operations such as portal restarts and database maintenance. The operator is deployed as part of the Shared Services Helm chart, which is required by IDA.
When upgrading the Shared Services chart to the latest version, the ArgoCD operator is automatically removed along with its associated Custom Resource Definitions (CRDs). Helm checks all resources in the existing release before applying changes. The EventSource object is already part of the deployed release which implies the CRD existed then and was later deleted, breaking the current application state and its upgrade.
The recommended upgrade procedure is detailed below:

## Upgrade Procedure

Perform the upgrades in the following order:

- Upgrade IDA to version 3.5.3.
- Check IDA is up and running
- Upgrade Shared Services to version 3.3.2.

# Post-Upgrade Validation

After completing both upgrades, verify that the cleanup has been successfully performed:

- Ensure that all IDA EventSource resources have been deleted.
- Confirm that the ArgoCD operator is no longer present in the Shared Services namespace.
- Verify that all related CRDs have been removed from the cluster.

# ArgoCD replacement in IDA 3.5.3

To address a CVE vulnerability introduced by the third-party ArgoCD component, IDA replaces ArgoCD with an internal **Service Ops Manager** starting in version **3.5.3**.

The Service Ops Manager now handles:

- Portal restart operations
- Database maintenance jobs
- Git synchronization operations

## Backward compatibility

For backward compatibility, IDA 3.5.3 can still use the previous ArgoCD-based behavior by setting:

```yaml
argoEvents.enabled: false
```

Using this fallback mode is **not recommended** for self-hosted deployments.

## Compatibility matrix

The table below shows compatibility between platform versions and the `argoEvents.enabled` flag value.

|          | IDA 3.5.2 | IDA 3.5.3         | IDA 3.6.0 |
| -------- | --------- | ----------------- | --------- |
| SS 3.3.1 | **X**     | `true` or `false` | **X**     |
| SS 3.3.2 | **X**     | `false` only      | **X**     |
| SS 3.4.0 | **X**     | `true` or `false` | **X**     |

- **IDA**: Identity Analytics
- **SS**: Shared Services
- **X**: Unsupported combination
