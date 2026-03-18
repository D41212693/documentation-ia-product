---
title: "Federal Information Processing Standards (FIPS) Mode"
description : "This document explains how Identity Analytics complies with FIPS."
---


# Federal Information Processing Standards (FIPS) Mode

FIPS mode is a configuration setting designed to ensure adherence to a set of security standards established by the U.S. government for the use of cryptographic modules.

FIPS mode is enabled by default in Identity Analytics. Identity Analytics leverages the following modules and services to comply with the standards:

## Safelogic CryptoComply GO Module

The Identity Analytics Configuration Service is FIPS certified using the Radiant Logic Cryptographic Module for Go. The certificate is available at: [FIPS Certificate](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4950). 

## KeyCloak FIPS

Identity Analytics uses KeyCloak for authentication and authorization. This is used for the certification of the Identity Provider (IDP) service.

## APISIX

Identity Analytics uses APISIX to provide communication services with OpenSSL version 3.0.9, ensuring FIPS compliance for secure communication.
