---
title: "Federal Information Processing Standards (FIPS) Mode"
description: "This document explains how Self-Managed Identity Analytics complies with FIPS and how to enable FIPS mode."
---

# Federal Information Processing Standards (FIPS) Mode

FIPS mode is a configuration setting designed to ensure adherence to a set of security standards established by the U.S. government for the use of cryptographic modules.

Identity Analytics leverages the following modules and services to comply with the standards. The FIPS mode setting described below (`keycloak.fips.enabled`) applies to Keycloak only: it does not change the other components of the instance.

## Safelogic CryptoComply GO Module

The Identity Analytics Configuration Service is FIPS certified using the Radiant Logic Cryptographic Module for Go. The certificate is available at: [FIPS Certificate](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5235).

## Keycloak FIPS

Identity Analytics uses Keycloak for authentication and authorization. Shared Services provide a FIPS build of Keycloak, used for the certification of the Identity Provider (IDP) service.

With `ida-shared-helm` 3.5.x, FIPS mode of Keycloak is **disabled by default**. Enable it in the Shared Services values:

```yaml
keycloak:
  fips:
    enabled: true
```

The chart then deploys the `ida-keycloak-fips` image instead of `ida-keycloak`.

> [!caution] In FIPS mode, Keycloak cannot verify passwords stored with the non-FIPS default hashing algorithm. Before you enable FIPS mode on an existing installation, set the hashing algorithm of the realms to `pbkdf2-sha512` and have each account log in once, or plan a password reset for the accounts that did not.

To set the hashing algorithm, in the Keycloak administration console (`https://<IDA_DNS>/auth/admin/`), for each realm that holds accounts (the instance realm `<IDA_NAMESPACE>` and `master`):

1. Select the realm, then **Authentication**, tab **Policies**, sub-tab **Password policy**.
2. Click **Add policy** and select **Hashing algorithm**.
3. Enter `pbkdf2-sha512` and click **Save**.

The new algorithm applies to a password the next time it is set or used to log in.

## APISIX

Identity Analytics uses APISIX as its gateway. The APISIX image of Shared Services 3.5.4 (`ida-apisix:3.5.4`) is built with OpenSSL 3.6.3, and the FIPS provider of OpenSSL is not part of the image. If your organisation requires FIPS-validated cryptography for the TLS connections of the gateway, contact Radiant Logic.
