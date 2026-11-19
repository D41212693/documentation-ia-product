---
title: Release Notes
description: Descartes R7 Release Notes
---

# Release Notes Descartes R7

> Release date: 2026-11-19

Descartes R7 moves iGRCanalytics to a new platform: a recent Eclipse platform, Java 21 and Tomcat 11.

## Before installing or upgrading

### Tomcat 11 is required

The web portal of Descartes R7 runs on Apache Tomcat 11 only. Tomcat 9 is no longer supported.

- Install Tomcat 11 with Java 21 before deploying the portal.
- Add the following java9 option to your Tomcat: `--add-opens=java.base/java.net=ALL-UNNAMED`.

### Identity Analytics (IAP) version

IAP 3.8 requires Descartes R7.

### SQL Server: use the latest JDBC driver

Use the latest version of the Microsoft JDBC Driver for SQL Server: we have seen instabilities with older versions. When you replace the driver, keep the file name `sqljdbc4.jar`. See [How-To install and use Microsoft SQL server official driver](../how-to/database/sqlserver/install-sqlserver-driver.md).

### Known vulnerability: CVE-2026-113730 (OGNL)

**BWIPUAR-4128** — Descartes R7 ships MyBatis 3.5.19, which embeds OGNL 3.4.4, so vulnerability scanners report CVE-2026-113730. It can't be exploited in iGRCanalytics. The fix (MyBatis 3.6.1 or newer) is planned for Descartes R7 SP1.

## New Features

- **BWIPUAR-2974** — Platform upgrade: Eclipse, Java 21 and Tomcat 11

  iGRCanalytics now runs on a recent Eclipse platform with Java 21 and Tomcat 11, replacing platform components that reached their end of life. The web portal framework (RAP) is updated too.

- **BWIPUAR-3071** — Role Mining: role candidates computed by the product

  The product computes role candidates by clustering identities with the same permissions, then ranks them by relevance, so the most useful candidates come first. They feed the Role Mining page of IAP 3.8.

- **BWIPUAR-3078** — Tickets & Activiti purge now includes IAP campaigns

  The tickets and Activiti purge now also removes the old IAP review campaigns, and its log counts the purged tickets correctly.

- **BWIPUAR-3060** — Option to keep metadata values unique

  A metadata target can now prevent inserting the same value several times. Duplicated values could slow some views down a lot.

- **BWIPUAR-3086** — Password parameters hidden in the portal logs

  When pages call views, parameters whose name contains "password" are no longer written in clear in the logs (it was already the case for workflows).

- **BWIPUAR-3059** — Faster loading of large page datasets

  The default `pages.dataset.batch-size` goes from 1,000 to 10,000, which speeds up pages that load large datasets.

## Improvements

- **BWIPUAR-3617, BWIPUAR-3702** — Performance: business view inner queries are limited on PostgreSQL

  On PostgreSQL, the SQL queries run inside business views now get a limit too, which can make them much faster. The behavior can be disabled.

- **BWIPUAR-3053** — Compact React header

  The React page header no longer repeats the timeslot selector shown in the navigation bar and gets compact on pages that need space.

## Bug fixes

- **BWIPUAR-3504** — The purge skipped old timeslots when an IAP campaign was active

  The timeslot purge found an active IAP campaign and stopped, so old timeslots were never purged. Fixed, together with IAP 3.8.

- **BWIPUAR-2927** — Workflows calling each other looped forever

  The workflow engine looped forever when workflow A called workflow B and B called A back. Fixed.

- **BWIPUAR-3052** — Activiti & tickets purge didn't work as expected

- **BWIPUAR-3057, SQ-1709** — Metadata computation could stay stuck after an error

  An error during the metadata computation could leave the execution plan stuck. It now ends in error.

- **BWIPUAR-3279** — "Out of Memory" errors on React pages in Chrome

  Chrome often showed "Out of Memory" errors on React pages and other portal pages. Fixed.

- **BWIPUAR-3085** — Rhino warnings in the portal log

  The portal log no longer fills with "RHINO USAGE WARNING: Missed Context.javaToJS() conversion" messages.

- **BWIPUAR-2984** — Missing slf4j logs in the portal

- **Platform upgrade** — Upgrading to a newer Eclispe version

- **BWIPUAR-2412** — Filters on page tables based on a business view

## FIPS Compliance

- **BWIPUAR-2237** — FIPS: credentials of the remediation configured from the IDA UI
- **BWIPUAR-2713** — FIPS: SMTP user and password

Software Bill of Materials of the iGRC components (CycloneDX 1.5 JSON and CSV): [Descartes-R7-SBOM.zip](./assets/sbom/Descartes-R7-SBOM.zip)
