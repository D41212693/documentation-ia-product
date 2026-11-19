# Release notes

This page covers release notes for self-managed Identity Analytics. For more information on the features and changes, refer to the Identity Analytics user guides.

See [vulnerability fixes](./08-vulnerability-fixes.md) for the complete list of patch versions that include vulnerability fixes in the images.

## Version 3.8.0

Release date: 2026-11-10

> [!warning]
> **Upgrading from 3.7.x: follow the [FIPS migration guide](./06-upgrade/fips-migration.md) before you upgrade.** Shared Services 3.6.0 and Identity Analytics 3.8.0 run FIPS images only. After the switch, Keycloak can no longer read the passwords stored by the standard Keycloak: users must reset their password unless you prepare the upgrade as described in the guide. Back up your databases first.

Identity Analytics 3.8 runs on a new platform (Eclipse, Java 21 and Tomcat 11) and uses FIPS 140-3 compliant images by default. See the [Identity Analytics 3.8 release notes](../../../iap-3.8/identity-analytics/iap-release/04-iap-3.X-release-notes/) for the product changes.

> See [Migration 3.8.0](./06-upgrade/migration-3.8.0.md) for details: several changes are BREAKING (Keycloak and database password length, SQL Tools removal).

> Recommended versions:
>
> - `Shared Services`: 3.6.0
> - `CNPG`: 1.30
> - `PostgreSQL`: 18.6

### New Features

- **BWIPUAR-2956, BWIPUAR-3618** — Images are FIPS 140-3 compliant by default

  All the Identity Analytics images now use FIPS-compliant base images, and the batch server runs in FIPS 140-3 mode.

- **BWIPUAR-3952** — `-fips` suffix removed from the Identity Analytics image names

  The Identity Analytics images only come in one flavor now, so the `-fips` suffix is gone from their names. The Shared Services images keep it (`ida-apisix-fips`, `ida-etcd-fips`, `ida-keycloak-fips`, `ida-tools-fips`). If you copy the images to a private registry, update your image list.

- **BWIPUAR-3958** — Shared Services: upgrade from non-FIPS to FIPS Keycloak

  Upgrading Shared Services from a standard Keycloak to the FIPS Keycloak of Shared Services 3.6 keeps the admin accounts of the chart working. The upgrade stops if it can't do it safely, and lists the application users whose password must be reset.

- **BWIPUAR-2558** — Trigger an execution plan from an external tool, through an authenticated endpoint.
- **BWIPUAR-2561** — Trigger a purge at will, from the config page or the external endpoint, and schedule it separately from the execution plan.
- **BWIPUAR-2560** — Update the project from Git and restart the portal from the config page.
- **BWIPUAR-2570** — Run a DB Check at will from the config page and download its results.
- **BWIPUAR-2569** — Mark a stuck upload or execution job as completed in the controller admin page.
- **BWIPUAR-2667** — The controller starts normally when its volumes and its external database are out of sync (e.g. after a cluster rebuild).
- **BWIPUAR-3043** — PostgreSQL 17 supported for the external database.
- **BWIPUAR-2543** — Istio support.
- **BWIPUAR-3355** — Kubernetes probes can be tuned or disabled from the Helm values, and the batch server liveness probe is more tolerant by default.
- **BWIPUAR-3356** — The JVM heap of the batch server, portal and workflow is set from a single place, based on the container memory limit.
- **BWIPUAR-3359** — The logs of an execution plan are kept when the batch server stops in the middle of it, and the controller marks the plan as failed right away.
- **BWIPUAR-3075** — Containers no longer allow privilege escalation, wherever possible.
- **BWIPUAR-3715** — SQL Tools removed.

### Bug Fixes

- **BWIPUAR-3454** — OIDC client secrets were regenerated on every Helm upgrade, which could break sessions. They are now kept.
- **BWIPUAR-2755, BWIPUAR-3354** — Keycloak SMTP settings were lost or only partly updated after an upgrade. They are now kept.
- **BWIPUAR-3353** — Data recovery: the DB maintenance script didn't work.

See [vulnerability fixes](./08-vulnerability-fixes.md) for additional details.

Software Bill of Materials (CycloneDX 1.5 JSON, all IDA 3.8.0 and Shared Services 3.6.0 container images): [IDA-3.8.0-SBOM.zip](./assets/sbom/IDA-3.8.0-SBOM.zip)

## Version 3.7.5

Release date: 2026-10-29

This release contains vulnerability fixes only; it introduces no additional feature changes.

> See [Migration 3.7.5](./06-upgrade/migration-3.7.5.md) for details.

> Recommended versions:
>
> - `Shared Services`: 3.5.5
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

See [vulnerability fixes](./08-vulnerability-fixes.md) for additional details.

Software Bill of Materials (CycloneDX 1.5 JSON, all IDA 3.7.5 and Shared Services 3.5.5 container images): [IDA-3.7.5-SBOM.zip](./assets/sbom/IDA-3.7.5-SBOM.zip)

## Version 3.7.4

Release date: 2026-10-12

This release contains vulnerability fixes only; it introduces no additional feature changes.

> Recommended versions:
>
> - `Shared Services`: 3.5.4
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

See [vulnerability fixes](./08-vulnerability-fixes.md) for additional details.

Software Bill of Materials (CycloneDX 1.5 JSON, all IDA 3.7.4 and Shared Services 3.5.4 container images): [IDA-3.7.4-SBOM.zip](./assets/sbom/IDA-3.7.4-SBOM.zip)

## Version 3.7.3

Release date: 2026-09-23

In addition to vulnerability fixes, this release includes the following change:

- Fixed email notifications not being sent when STARTTLS is enabled.

> Recommended versions:
>
> - `Shared Services`: 3.5.3
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

See [vulnerability fixes](./08-vulnerability-fixes.md) for additional details.

## Version 3.7.2

Release date: 2026-09-08

This release contains vulnerability fixes only; it introduces no additional feature changes.

> Recommended versions:
>
> - `Shared Services`: 3.5.2
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

See [vulnerability fixes](./08-vulnerability-fixes.md) for additional details.

## Version 3.7.1

Release date: 2026 August 17

In addition to vulnerability fixes, this release includes the following changes:

- Depreciation of flat JVM Heap Keys. These are now replaced by `jvm.memory.*`
- JVM Heap options that are set outside `jvm.memory` are now ignored.
- Relaxed the batch-server liveness probe defaults
- Corrected the default value of `batch.archiveLogs.maxAge` to use Go duration format.

> See [Migration 3.7.1](./06-upgrade/migration-3.7.1.md) for details

> Recommended versions:
>
> - `Shared Services`: 3.5.1
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

## Version 3.7.0

Release date: 2026 July 29

> See [Migration 3.7.0](./06-upgrade/migration-3.7.0.md)

> Recommended versions:
>
> - `Shared Services`: 3.5.0
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

### New Features

- **BWIPUAR-2532** — Allow use of an external PostgreSQL database

> Only the `autoinit` option is supported!

### Bug Fixes

- **BWIPUAR-2531** — Data-recovery pod issues after 3.3.1

### Vulnerability fixes

See [vulnerability fixes](./08-vulnerability-fixes.md).

## Version 3.6.2

Release date: 2026 June 30

Only includes [vulnerability fixes](./08-vulnerability-fixes.md).

> Recommended versions:
>
> - `Shared Services`: 3.4.2
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

## Version 3.6.1

Release date: 2026 June 9

Only includes [vulnerability fixes](./08-vulnerability-fixes.md).

## Version 3.6.0

Release date: 2026 May 19

> See [Migration 3.6.0](./06-upgrade/migration-3.6.0.md)

- **BWIPUAR-2627** - Updated batch log cleanup policy.
- **BWIPUAR-2563** - Added support for automatically redirecting users from the login page to a preconfigured Identity Provider (IdP) to improve the authentication experience.

## Version 3.5.3

Release date: 2026 March 27

> See [Migration 3.5.3](./06-upgrade/migration-3.5.3.md)

Only includes [vulnerability fixes](./08-vulnerability-fixes.md).

## Version 3.5.1

Release date: 2026 March 13

> See [Migration 3.5.1](./06-upgrade/migration-3.5.1.md)

- **BWIPUAR-2563** – [Self-Managed] Allow to configure login page to directly redirect to a preconfigured IDP
- **BWIPUAR-2562** – [Self-Managed] Access to /ws through authenticated client does not work as of 3.4.1
- **BWIPUAR-2548** – [Self-Managed] Allow users to easily expand the data-recovery pod with a sidecar dedicated to downloading the export archive
- **BWIPUAR-2581** – Add AIDA enabled flag
- **BWIPUAR-2531** – [Self-Managed] data-recovery pod issues after 3.3.1

## Version 3.5

Release date: 2026 February 04

- **BWIPUAR-2419** – Updated Git Sync in self-managed mode to let users select alternative authentication methods.
- **BWIPUAR-2433** – Resolved an issue preventing Identity Data Management credentials from being saved in configuration.
- **BWIPUAR-2446** – Added support for adding custom labels to all ida-helm components in self-managed deployments.
- **BWIPUAR-2452** – Corrected an issue that prevented users from renaming the technical configuration on the batch configuration page.
- **BWIPUAR-2507** – Enhanced self-managed deployments to allow greater customization of Ingress parameters.
- **BWIPUAR-2523** – Fixed a problem where sample installations on OpenShift could fail.
- **BWIPUAR-2522** – Added support for bearer token authentication when connecting to AWS Bedrock.

## Version 3.4

Release date: 2025 September 30

- **BWIPUAR-2278:** [Packaging] Saving Project Mashup Dashboard in SaaS
- **BWIPUAR-2249:** [Packaging] Activate SDC for communication from SaaS to IDDM on-premise
- **BWIPUAR-2373:** [Extractors] IdentityNow (IDN) Extractor and collect available in IDA self-managed and SaaS
- **BWIPUAR-2391:** [Extractors] IdentityIQ (IIQ) Extractor and collect available in IDA self-managed and SaaS
- **BWIPUAR-2156:** [Extractors] ServiceNow extractor: support "refresh ID token"
- **BWIPUAR-2382:** [Extractors] Update Entra Id script to be compatible with Powershell 7
- **BWIPUAR-2382:** [AIDA] support of AIDA deployments for OpenShift

## Version 3.3.2

- **BWIPUAR-2264:** [Packaging] Link to the login page in the logout when SaaS deployment
- **BWIPUAR-2424:** [AIDA] Identity Analytics Self-Managed with AWS Bedrock provided by the customer supported
- **BWIPUAR-2425:** [AIDA] Identity Analytics Self-Managed with Azure AI Foundry provided by the customer
- **BWIPUAR-2426:** [AIDA] Identity Analytics Self-Managed with Google Vertex AI provided by the customer

## Version 3.3.1

Release date: 2025 June 17

- **PP-797:** [Extraction] Decoding issue with Entra ID connector through IDDM
