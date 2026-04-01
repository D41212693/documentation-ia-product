---
title: "Certified Operating Environment for Identity Analytics"
description: "Certified Operating Environment for Identity Analytics"
---

# Certified Operating Environment for Identity Analytics

This document covers a list of certified operating environments for Identity Analytics. The operating systems, databases, and web browsers that are listed in this page are supported across all versions of the Descartes major release.

## Operating Systems

Operating systems are third‑party components and are not included with Identity Analytics.
Identity Analytics is composed of the following main modules:

* Studio
* Batch
* Web Portal

The table below provides the operating system support matrix for each module:

| **OS**         | **Version**     | **Studio** | **Batch** | **Webportal** |
| :------------- | :-------------- | :--------: | :-------: | :-----------: |
| Windows        | 10              |   **X**    |   **X**   |     **X**     |
| Windows        | 11              |   **X**    |   **X**   |     **X**     |
| Windows server | 2012 R2         |   **X**    |   **X**   |     **X**     |
| Windows server | 2016            |   **X**    |   **X**   |     **X**     |
| Windows server | 2019            |   **X**    |   **X**   |     **X**     |
| Windows server | 2022            |   **X**    |   **X**   |     **X**     |
| RHEL           | 9[^rhel9]       |            |   **X**   |     **X**     |
| Debian         | LTS[^debianLTS] |            |   **X**   |     **X**     |

Please see footnotes (links available in the above mentioned table) for more information on LTS versions.

## Java development kits

Java Development Kits (JDKs) are third‑party components and are not included with Identity Analytics.
Refer to the table below for details on the supported versions of Java.

|                         | **Windows[^2]** | **Linux RHEL 9** | **Linux Debian** |
| :---------------------: | :-------------: | :--------------: | :--------------: |
|   **Java SE JDK 17**    |      **X**      |                  |                  |
|     **Open JDK 17**     |                 |      **X**       |      **X**       |
|     **Corretto 17**     |      **X**      |                  |                  |
| **AdoptOpenJDK 17 LTS** |      **X**      |                  |                  |

> When downloading AdoptOpenJDK, ensure that you select the HotSpot JVM, as it is the only supported option.

## Database Servers

Database servers are third‑party components and are not included with Identity Analytics. Identity Analytics supports the following database servers:

* Microsoft SQL Server
* PostgreSQL
* Oracle

The database support matrix, organized by operating system, is provided below.

### Windows environment

| **Database**         | **Version** | **Window 10** | **Window 11** | **Window Server 2012** | **Windows server 2016** | **Windows server 2019** | **Windows server 2022** |
| :------------------- | :---------- | :-----------: | :-----------: | :--------------------: | :---------------------: | :---------------------: | :---------------------: |
| Microsoft SQL server | 2014        |     **X**     |               |         **X**          |                         |                         |                         |
| Microsoft SQL server | 2016        |               |               |         **X**          |          **X**          |                         |                         |
| Microsoft SQL server | 2017        |               |               |                        |          **X**          |                         |                         |
| Microsoft SQL server | 2019        |               |     **X**     |                        |                         |          **X**          |                         |
| Microsoft SQL server | 2022        |               |     **X**     |                        |                         |          **X**          |          **X**          |
| Microsoft SQL server | 2025        |               |     **X**     |                        |                         |          **X**          |          **X**          |
| Oracle[^1]           | 19c         |     **X**     |     **X**     |         **X**          |                         |                         |          **X**          |

### Linux environment

| **Database** | **Version** | **RHEL 9** | **Debian** |
| :----------- | :---------- | :--------: | :--------: |
| PostgreSQL   | 13          |   **X**    |   **X**    |
| PostgreSQL   | 14          |   **X**    |   **X**    |
| PostgreSQL   | 15          |   **X**    |   **X**    |
| PostgreSQL   | 16          |   **X**    |   **X**    |
| PostgreSQL   | 17          |   **X**    |   **X**    |
| Oracle[^1]   | 19c         |   **X**    |            |

> [!warning] Oracle is only supported for existing and deployed projects. For new projects, only Microsoft SQL Server and PostgreSQL are supported.

For more information, refer to the official RMDS support lifecycles:

* [postgres](https://www.postgresql.org/support/versioning/)
* [Microsoft SQL server](https://learn.microsoft.com/en-us/lifecycle/products/?terms=sql%20server)
* [Oracle](https://endoflife.date/oracle-database)

> [!warning] Retrieve the PostgreSQL version corresponding to your target distribution. Do not recompile the PostgreSQL kernel.

## Database drivers

The database drivers used must be compatible with the version of Java in use. The following drivers have been certified with Identity Analytics:

* `mssql-jdbc-11.2.1.jre17.jar` and `mssql-jdbc-12.2.0.jre11.jar`
* `ojdbc11.jar`
* `postgresql-42.5.1`

> If you are using an Oracle JDBC driver, download the version that matches your database engine. More details are available [here](https://www.oracle.com/fr/database/technologies/appdev/jdbc-downloads.html).

> [!warning] Oracle's OCI driver is not supported by Identity Analytics.

Oracle and Microsoft SQL Server JDBC drivers are third‑party components and are not included with Identity Analytics.
For installation instructions, refer to the following guides:

* [How-To install and use Microsoft SQL server official driver](../../how-to/database/sqlserver/install-sqlserver-driver.md)
* [How-to install and use the official Oracle database driver](../../how-to/database/oracle/install-orcl-driver.md)

## Java Application Server

The Java Application Server is a third‑party component and is not provided with Identity Analytics. Apache Tomcat 9.0 (all Operating Systems) is the only application server supported by Identity Analytics.

## Web browsers

Identity Analytics supports only long-term support (LTS) or business editions of major web browsers. The supported browsers are listed below.

### Firefox

The current version of Firefox ESR is supported (Extended Support Release). For more information, refer to the following link:  
<https://www.mozilla.org/en-US/firefox/enterprise/>

### Chrome

The supported version of chrome is the current version of the Chrome Browser for Businesses. Read the following document for more information: <https://chromeenterprise.google/>

### Microsoft Edge

Refer to Microsoft’s official Edge support lifecycle documentation:
<https://learn.microsoft.com/en-US/deployedge/microsoft-edge-support-lifecycle>

> Internet explorer is no longer recommended. Refer to: <https://techcommunity.microsoft.com/t5/Windows-IT-Pro-Blog/The-perils-of-using-Internet-Explorer-as-your-default-browser/ba-p/331732>

[^1]: As of version Braille, Oracle is only supported in the case of existing and deployed projects. In the case of a new project only Microsoft SQL server and PostgreSQL are supported.

[^2]: All versions of java 17

[^debianLTS]: Refer to Debian's official documentation for more information: <https://wiki.debian.org/LTS>

[^rhel9]: Refer to Red Hat's official documentation for more information: <https://access.redhat.com/support/policy/updates/errata#Maintenance_Support_2_Phase>
