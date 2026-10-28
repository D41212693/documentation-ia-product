# Released notes

This page covers release notes for self-managed Identity Analytics. For more information on the features and changes, refer to the Identity Analytics user guides.

See [vulnerability fixes](./12-security/02-vulnerability-fixes.md) for the complete list of patch versions that include vulnerability fixes in the images.

## Version 3.7.4

Release date: 2026-10-12

This release contains vulnerability fixes only; it introduces no additional feature changes.

> Recommended versions:
>
> - `Shared Services`: 3.5.4
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

See [vulnerability fixes](./12-security/02-vulnerability-fixes.md) for additional details.

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

See [vulnerability fixes](./12-security/02-vulnerability-fixes.md) for additional details.

## Version 3.7.2

Release date: 2026-09-08

This release contains vulnerability fixes only; it introduces no additional feature changes.

> Recommended versions:
>
> - `Shared Services`: 3.5.2
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

See [vulnerability fixes](./12-security/02-vulnerability-fixes.md) for additional details.

## Version 3.7.1

Release date: 2026 August 17

In addition to vulnerability fixes, this release includes the following changes:

- Depreciation of flat JVM Heap Keys. These are now replaced by `jvm.memory.*`
- JVM Heap options that are set outside `jvm.memory` are now ignored.
- Relaxed the batch-server liveness probe defaults
- Corrected the default value of `batch.archiveLogs.maxAge` to use Go duration format.

> See [Migration 3.7.1](./11-operations/04-upgrade/migration-3.7.1.md) for details

> Recommended versions:
>
> - `Shared Services`: 3.5.1
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

## Version 3.7.0

Release date: 2026 July 29

> See [Migration 3.7.0](./11-operations/04-upgrade/migration-3.7.0.md)

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

See [vulnerability fixes](./12-security/02-vulnerability-fixes.md).

## Version 3.6.2

Release date: 2026 June 30

Only includes [vulnerability fixes](./12-security/02-vulnerability-fixes.md).

> Recommended versions:
>
> - `Shared Services`: 3.4.2
> - `CNPG`: 1.29
> - `PostgreSQL`: 18.3

## Version 3.6.1

Release date: 2026 June 9

Only includes [vulnerability fixes](./12-security/02-vulnerability-fixes.md).

## Version 3.6.0

Release date: 2026 May 19

> See [Migration 3.6.0](./11-operations/04-upgrade/migration-3.6.0.md)

- **BWIPUAR-2627** - Updated batch log cleanup policy.
- **BWIPUAR-2563** - Added support for automatically redirecting users from the login page to a preconfigured Identity Provider (IdP) to improve the authentication experience.

## Version 3.5.3

Release date: 2026 March 27

> See [Migration 3.5.3](./11-operations/04-upgrade/migration-3.5.3.md)

Only includes [vulnerability fixes](./12-security/02-vulnerability-fixes.md).

## Version 3.5.1

Release date: 2026 March 13

> See [Migration 3.5.1](./11-operations/04-upgrade/migration-3.5.1.md)

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
