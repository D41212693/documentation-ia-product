---
title: "Glossary"
description: "Terms used in the Self-Managed Identity Analytics deployment guide."
---

# Glossary

This page explains the product terms used in this guide. Kubernetes and Helm terms are not repeated here.

## Platform

| Term | Meaning |
| :--- | :------ |
| Identity Analytics (IDA) | The Radiant Logic identity governance and analytics product deployed by this guide, also known as iGRC Analytics. |
| Self-Managed | The deployment mode where you install and operate Identity Analytics on your own Kubernetes cluster with the Helm charts `ida-shared-helm` and `ida-helm`. |
| Shared Services | The components installed once per cluster by `ida-shared-helm`: Keycloak, the APISIX gateway and its etcd store. All the instances of the cluster use them. |
| Instance | One Identity Analytics deployment installed by `ida-helm`, in its own namespace, with its own URL path (`<IDA_PATH>`), project, database and Keycloak realm. Production and acceptance are usually two instances. |
| Built-in database | The PostgreSQL database run in the cluster by the CloudNativePG operator (`database.type: internal`). |
| External database | A PostgreSQL server outside the cluster, operated by your organisation (`database.type: external`). See [External database](./04-external-database.md). |
| CloudNativePG (CNPG) | The Kubernetes operator that runs and backs up the built-in database. |
| APISIX | The API gateway of Shared Services. It checks the user session with Keycloak and routes requests to the instances. |
| Keycloak | The identity provider of Shared Services. It authenticates users, locally or through your identity provider. |
| Realm | A Keycloak tenant. Each instance has its own realm, named after its namespace (`<IDA_NAMESPACE>`). The `master` realm is reserved for the administration of Keycloak. |
| Technical administrator | The built-in administrator account of an instance realm, `setup` by default (`global.technicalAdmin.username`). |
| Built-in roles | The roles of the `identity-analytics` client of the instance realm: `user`, `designer`, `functionaladmin` and `technicaladmin`. See [Authentication and authorization configuration](./06-configuration/02-authentication-configuration.md). |

## Applications of an instance

| Term | Meaning |
| :--- | :------ |
| Portal | The web application used by end users and administrators (Governance Portal), at `https://<IDA_DNS><IDA_PATH>/portal`. |
| Configuration UI | The web application that holds the technical settings of an instance, at `<IDA_PATH>/config`. See [Configuration UI](./06-configuration/01-config-ui.md). |
| Controller | The web application that receives the uploaded files, lists the connectors and follows the execution plans. See [Controller](./09-components/controller.md). |
| Configuration store | The database where the instance keeps its settings and the secrets entered in the configuration UI, encrypted. |

## Project and data

| Term | Meaning |
| :--- | :------ |
| Project | The configuration of what the instance collects and computes: data sources, collectors, data model, rules, reports and portal pages. It is stored in a Git repository, in the folder `brainwave`. See [Project repository](./02-prerequisites.md#project-repository). |
| Reference project | The standard project published by Radiant Logic, with one branch per release (`release/<MAJOR.MINOR>`). |
| iGRC Studio | The desktop application used to edit a project. Changes are committed and pushed to the Git repository of the project. |
| Identity Ledger | The data model of Identity Analytics: identities, organisations, accounts, groups, applications, permissions and their links. |
| Timeslot | A dated snapshot of the Identity Ledger, produced by an execution plan. The portal shows the current timeslot. |
| Execution plan | The batch process that loads the import files into a new timeslot. It is started from the controller or the portal, or on a schedule. By default, the new timeslot is validated automatically at the end of a successful plan. |
| Validated timeslot | A timeslot made available in the portal after a successful execution plan. |
| Data source | A source of data declared in the project, for example an HR file or a directory. |
| Data Source Management (DSM) | The portal menu used to declare data sources, map their files to the Identity Ledger, upload files and launch an execution plan. |
| Collector | The part of the project that reads a source file and loads its content into the Identity Ledger. |
| Reconciliation | The link between an account and the identity that owns it. |
| Connector | A standard extraction declared in the portal (for example Entra ID or SailPoint). The platform runs it and adds its result to the import files. |
| Import files | The files used by the next execution plan, stored in `/data/importfiles` (project variable `importfiles`). Each data source reads its files from its own folder under it. |
| Upload | A set of files sent to the controller, from the portal, the PowerShell extractor or a connector. |
| PowerShell extractor | The Windows script (`bw_data_collector`) that extracts data from Active Directory and other sources and uploads it to the instance. See [PowerShell extraction](./06-configuration/03-powershell-extraction.md). |
| Descartes | The name of the on-premises Identity Analytics releases (R5, R6…), the starting point of a [data migration](./10-data-migration/). |
