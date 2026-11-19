---
title: "Federal Information Processing Standards (FIPS) Mode"
description: "This document explains how Self-Managed Identity Analytics complies with FIPS and how to enable FIPS mode."
---

# Federal Information Processing Standards (FIPS) Mode

FIPS mode is a configuration setting designed to ensure adherence to a set of security standards established by the U.S. government for the use of cryptographic modules.

Starting with `ida-shared-helm` 3.6.0 and `ida-helm` 3.8.0, FIPS mode is enabled by default: Keycloak, APISIX, etcd, PostgreSQL, the portal, the batch server and the data recovery run FIPS images, whose cryptography uses only FIPS 140-3 validated modules. No additional configuration is needed. To upgrade an installation that runs the standard images (Shared Services 3.5.x and earlier), follow [Migrating to FIPS](../11-operations/04-upgrade/fips-migration.md): after the switch, Keycloak can no longer read the passwords stored by the standard Keycloak.

Identity Analytics leverages the following modules and services to comply with the standards.

## Safelogic CryptoComply GO Module

The Identity Analytics Configuration Service is FIPS certified using the Radiant Logic Cryptographic Module for Go. The certificate is available at: [FIPS Certificate](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5235).

## Keycloak FIPS

Identity Analytics uses Keycloak for authentication and authorization. Shared Services 3.6.0 deploy the FIPS build of Keycloak (`ida-keycloak-fips`), used for the certification of the Identity Provider (IDP) service.

Passwords must have 14 characters or more: the charts refuse shorter ones.

## APISIX

Identity Analytics uses APISIX as its gateway. The APISIX image of Shared Services 3.6.0 (`ida-apisix-fips:3.6.0`) is built with OpenSSL 3.6.4 and the Chainguard FIPS Provider 3.4.0, active by default.
