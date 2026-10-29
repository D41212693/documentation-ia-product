---
title: "Upgrades"
description: "Upgrade Self-Managed Identity Analytics to a new release."
---

# Self-Managed upgrades

An upgrade installs new chart versions with your existing values files. Always read the migration guide of each version between yours and the target first: it lists the values changes and, for some versions, a specific upgrade order.

## Procedure

1. Read the release notes and the migration guides below, and apply the values changes they list.
2. Take a backup of the databases (see [Backup and restore](../11-operations/01-backup-and-restore.md)), or enable `database.backup.onUpgrade`.
3. If the target is a new minor version (for example 3.6 to 3.7), merge the reference project of that version into your project and set `git.default.branch` to the matching branch.
4. Upgrade the two charts in the order given by the migration guide of the target version. When the guide gives no order, upgrade Shared Services first, wait until Keycloak is ready, then upgrade each Identity Analytics instance.

    ```bash
    helm upgrade rlss oci://docker.io/radiantone/ida-shared-helm \
      --namespace <SHARED_NAMESPACE> --version <SHARED_CHART_VERSION> \
      --values shared.values.yaml --wait --timeout 20m

    helm upgrade rlia oci://docker.io/radiantone/ida-helm \
      --namespace <IDA_NAMESPACE> --version <IDA_CHART_VERSION> \
      --set-file global.licenseFile=<PATH_TO_LICENSE_FILE> \
      --values ida.values.yaml --timeout 30m
    ```

    Always give a `--timeout`: pre-upgrade jobs can run for several minutes.

5. Run the [post-installation checks](../05-post-installation.md).

> [!warning] Downgrades are not supported: an upgrade migrates the database schemas. To go back, restore the backup taken before the upgrade.

Each upgrade runs the `config-store-init` job again. It applies the values (for example `smtp`) to the configuration store and overwrites the same settings changed by hand in the configuration interface: report such changes in your values files before you upgrade. The Git project chosen in the configuration interface is kept. See [Which setting wins](../06-configuration/01-config-ui.md#which-setting-wins).

## Version-specific guides

- [Version 3.7.1](./04-upgrade/migration-3.7.1.md)
- [Version 3.7.0](./04-upgrade/migration-3.7.0.md)
- [Version 3.6.0](./04-upgrade/migration-3.6.0.md) (upgrade Identity Analytics before Shared Services)
- [Version 3.5.3](./04-upgrade/migration-3.5.3.md) (upgrade Identity Analytics before Shared Services)
- [Version 3.5.1](./04-upgrade/migration-3.5.1.md)

Versions 3.7.2 to 3.7.4 (Shared Services 3.5.2 to 3.5.4) have no migration guide: upgrade with the procedure above, and see the release notes for their content.
