# Released notes

Please find below the release notes for Identity Analytics in self-managed. For more information on the contents of the project please refer to Identity Analytics user guides.

See [vulnerability fixes](./08-vulnerability-fixes.md) for the list of fixed vulnerabilities in the images.

## Version 3.5.1

> See [Migration 3.5.1](./06-upgrade/migration-3.5.1.md)

- **BWIPUAR-2563** – [Self-Managed] Allow to configure login page to directly redirect to a preconfigured IDP
- **BWIPUAR-2562** – [Self-Managed] Access to /ws through authenticated client does not work as of 3.4.1
- **BWIPUAR-2548** – [Self-Managed] Allow users to easily expand the data-recovery pod with a sidecar dedicated to downloading the export archive
- **BWIPUAR-2581** – Add AIDA enabled flag
- **BWIPUAR-2531** – [Self-Managed] data-recovery pod issues after 3.3.1

## Version 3.5

- **BWIPUAR-2419** – Updated Git Sync in self-managed mode to let users select alternative authentication methods.
- **BWIPUAR-2433** – Resolved an issue preventing Identity Data Management credentials from being saved in configuration.
- **BWIPUAR-2446** – Added support for adding custom labels to all ida-helm components in self-managed deployments.
- **BWIPUAR-2452** – Corrected an issue that prevented users from renaming the technical configuration on the batch configuration page.
- **BWIPUAR-2507** – Enhanced self-managed deployments to allow greater customization of Ingress parameters.
- **BWIPUAR-2523** – Fixed a problem where sample installations on OpenShift could fail.
- **BWIPUAR-2522** – Added support for bearer token authentication when connecting to AWS Bedrock.

## Version 3.4

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

- **PP-797:** [Extraction] Decoding issue with Entra ID connector through IDDM
