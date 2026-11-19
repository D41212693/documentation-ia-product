---
title: "Self-Managed deployment guide"
description: "Deploy, configure, and operate Identity Analytics on your own Kubernetes cluster using Helm."
---

# Self-Managed Identity Analytics deployment

Self-Managed Identity Analytics runs on your own Kubernetes cluster. Installation and upgrades are performed exclusively through Helm. This guide covers the deployment lifecycle, including planning, installation, configuration, migration from an on-premises installation, operations, and security.

## Supported versions

Use the chart versions listed for your Identity Analytics release. Always specify the target chart version with `--version` when running the Helm commands included in the documents.

| Release | `ida-helm` chart | `ida-shared-helm` chart | CloudNativePG chart (internal database) |
| :------ | :--------------: | :---------------------: | :-------------------------------------: |
| 3.8     |      3.8.0       |          3.6.0          |         0.29.1 (operator 1.30)          |
| 3.7     |      3.7.5       |          3.5.5          |       0.28.2 (operator 1.29.1)          |
| 3.6     |      3.6.2       |          3.4.2          |                    -                    |
| 3.5     |      3.5.3       |          3.3.2          |                    -                    |
| 3.4     |      3.4.3       |          3.2.2          |                    -                    |
| 3.3     |      3.3.2       |          3.2.0          |                    -                    |
| 3.2     |      3.2.0       |          3.1.1          |                    -                    |
| 3.1     |      3.1.0       |          3.1.0          |                    -                    |

The Identity Analytics and Shared Services charts are published in the `docker.io/radiantone` OCI registry:

- Identity Analytics: `oci://docker.io/radiantone/ida-helm`
- Shared Services: `oci://docker.io/radiantone/ida-shared-helm`

## Documents structure

| Step | Page | Topics |
| :--- | :--- | :----- |
| Understand | [Architecture](./self-managed/01-architecture.md) | Cluster components, traffic flow, storage, and limits |
| Plan | [Prerequisites and sizing](./self-managed/02-prerequisites.md) | Cluster requirements, tools, networking, DNS, TLS, registry access, and sizing |
| Install | [Installation](./self-managed/03-installation.md) | Step-by-step installation of CloudNativePG, Shared Services, and Identity Analytics |
| Install | [External database](./self-managed/04-external-database.md) | Configuration of a PostgreSQL server outside the cluster as an alternative to the built-in database |
| Verify | [Post-installation](./self-managed/05-post-installation.md) | Deployment verification, initial login, users, and roles |
| Configure | Configuration, integrations, extractors, components | Post-installation platform configuration |
| Migrate | Data migration | Migration from an on-premises installation to Self-Managed Identity Analytics |
| Operate | Operations | Backup and restore, troubleshooting, uninstallation, upgrades, and project changes |
| Secure | Security | FIPS mode and vulnerability reports |
| Reference | [Glossary](./self-managed/14-glossary.md) | Product-specific terminology used in this guide |

> [!note]
> All customers use the same Helm charts. Configure environment-specific settings, such as DNS names, namespaces, storage classes, ingress classes, and passwords, in your own values files. This guide represents these values as `<PLACEHOLDER>`.
