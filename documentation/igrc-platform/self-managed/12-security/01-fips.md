---
title: "Federal Information Processing Standards (FIPS) Mode"
description: "This document explains how Self-Managed Identity Analytics complies with FIPS and how to enable FIPS mode."
---

# Federal Information Processing Standards (FIPS) Mode

FIPS mode is a configuration setting designed to ensure adherence to a set of security standards established by the U.S. government for the use of cryptographic modules.

Identity Analytics leverages the following modules and services to comply with the standards.

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

> [!caution] In FIPS mode, Keycloak cannot verify passwords stored with the non-FIPS default hashing algorithm. Before you enable FIPS mode on an existing installation, set the password policy of the realms to `pbkdf2-sha512` and have each account log in once, or plan a password reset for the accounts that did not.

## APISIX

Identity Analytics uses APISIX to provide communication services with OpenSSL version 3.0.9, ensuring FIPS compliance for secure communication.
