---
title: "Self-Managed deployment guide"
description: "Deploy, configure and operate Identity Analytics on your own Kubernetes cluster with Helm."
---

# Self-Managed Identity Analytics Deployment

Self-Managed Identity Analytics runs on your own Kubernetes cluster and is installed and upgraded exclusively with Helm. This guide covers the whole lifecycle of a deployment: planning, installation, configuration, data migration from an on-premises installation, operations and security.

## Supported versions

Install the chart versions that match your Identity Analytics release. Always pass the target version explicitly (`--version`) to the `helm` commands of this guide.

| Release | `ida-helm` chart | `ida-shared-helm` chart | CloudNativePG chart (internal database) |
| :------ | :--------------: | :---------------------: | :-------------------------------------: |
| 3.7     |      3.7.4       |          3.5.4          |       0.28.2 (operator 1.29.1)          |
| 3.6     |      3.6.2       |          3.4.2          |                    -                    |
| 3.5     |      3.5.3       |          3.3.2          |                    -                    |
| 3.4     |      3.4.3       |          3.2.2          |                    -                    |
| 3.3     |      3.3.2       |          3.2.0          |                    -                    |
| 3.2     |      3.2.0       |          3.1.1          |                    -                    |
| 3.1     |      3.1.0       |          3.1.0          |                    -                    |

The charts are published in the `docker.io/radiantone` OCI registry (`oci://docker.io/radiantone/ida-helm` and `oci://docker.io/radiantone/ida-shared-helm`).

## How this guide is organised

| Step | Page | What you will find |
| :--- | :--- | :----------------- |
| Understand | [Architecture](./self-managed/01-architecture.md) | Components deployed in the cluster, traffic flow, storage and limits |
| Plan | [Prerequisites and sizing](./self-managed/02-prerequisites.md) | Cluster, tools, network, DNS, TLS, registry, sizing |
| Install | [Installation](./self-managed/03-installation.md) | Step-by-step installation of CloudNativePG, Shared Services and Identity Analytics |
| Install | [External database](./self-managed/04-external-database.md) | Use a PostgreSQL server outside the cluster instead of the built-in database |
| Verify | [Post-installation](./self-managed/05-post-installation.md) | Checks, first login, users and roles |
| Configure | Configuration, integrations, extractors, components | Day-2 configuration of the platform |
| Migrate | Data migration | Move an on-premises installation to Self-Managed |
| Operate | Operations | Backup and restore, troubleshooting, uninstallation, upgrades, project changes |
| Secure | Security | FIPS mode and vulnerability reports |
| Reference | [Glossary](./self-managed/14-glossary.md) | Product terms used in this guide |

> [!note] The same Helm charts are used for all customers. Everything specific to your environment (DNS names, namespaces, storage class, ingress class, passwords) is set in your own values files and is written as `<PLACEHOLDER>` in this guide.
