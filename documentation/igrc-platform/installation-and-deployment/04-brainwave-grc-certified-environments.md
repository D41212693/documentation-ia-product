---
title: "Certified Operating Environment for Identity Analytics"
description: "Certified Operating Environment for Identity Analytics"
---

# Certified Operating Environment for Identity Analytics

Following you will find the list of certified operating environments for Identity Analytics.  
The following operating systems, databases and web browsers are valid for all versions of the Descartes major release.

## Operating Systems

Operating Systems are Third Party Software not provided with Identity Analytics.
Identity Analytics is composed of three main modules:

- The studio (or back office)
- The batch
- The web portal

The OS support matrix is the following:

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

Java development kits (JDK) are Third Party Software not provided with Identity Analytics.  
Please see the following table for more information on the support version of JAVA :

|                         | **Windows[^2]** | **Linux RHEL 9** | **Linux Debian** |
| :---------------------: | :-------------: | :--------------: | :--------------: |
|   **Java SE JDK 17**    |      **X**      |                  |                  |
|     **Open JDK 17**     |                 |      **X**       |      **X**       |
|     **Corretto 17**     |      **X**      |                  |                  |
| **AdoptOpenJDK 17 LTS** |      **X**      |                  |                  |

> **Note:** When downloading AdoptOpenJDK only the HotSpot JVM is supported

## Database Servers

Database servers are Third Party Software not provided with Identity Analytics.

The following database servers are supported by Identity Analytics:

- Microsofts SQL server
- PostgreSQL
- Oracle[^1]

The database certified support matrix, per operating systems, is as follows:

### Windows environment

| **Database**         | **Version** | **Window 10** | **Window 11** | **Window Server 2012** | **Windows server 2016** | **Windows server 2019** | **Windows server 2022** |
| :------------------- | :---------- | :-----------: | :-----------: | :--------------------: | :---------------------: | :---------------------: | :---------------------: |
| Microsoft SQL server | 2014        |     **X**     |               |         **X**          |                         |                         |                         |
| Microsoft SQL server | 2016        |               |               |         **X**          |          **X**          |                         |                         |
| Microsoft SQL server | 2017        |               |               |                        |          **X**          |                         |                         |
| Microsoft SQL server | 2019        |               |     **X**     |                        |                         |          **X**          |                         |
| Microsoft SQL server | 2022        |               |     **X**     |                        |                         |          **X**          |          **X**          |
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

> [!warning] Please note that oracle is only supported in the case of existing and deployed projects. In the case of a new project only Microsoft SQL server and PostgreSQL are supported.

Please navigate each RMDS support lifecycle for more information:

- [postgres](https://www.postgresql.org/support/versioning/)
- [Microsoft SQL server](https://learn.microsoft.com/en-us/lifecycle/products/?terms=sql%20server)
- [Oracle](https://endoflife.date/oracle-database)

> [!warning] Please retrieve the PostgreSQL version for the target distribution. Do NOT recompile the PostgreSQL kernel.

## Database drivers

The used database drivers must be compatible with the corresponding version of JAVA used:  
The following database drivers have been certified with Identity Analytics:

- `mssql-jdbc-11.2.1.jre17.jar` and `mssql-jdbc-12.2.0.jre11.jar`
- `ojdbc11.jar`
- `postgresql-42.5.1`

> If using an oracle JDBC driver please download the correct driver in accordance to the version of the database engine used. Please see [here](https://www.oracle.com/fr/database/technologies/appdev/jdbc-downloads.html) for more information.

> [!warning] Oracle's OCI driver is not supported by Identity Analytics.

Oracle and Microsoft SQL JDBC drivers are Third Party Software not provided with Identity Analytics.
Please refer to the following pages for information on how to download and install the drivers :

- [How-To install and use Microsoft SQL server official driver](../../how-to/database/sqlserver/install-sqlserver-driver.md)
- [How-to install and use the official Oracle database driver](../../how-to/database/oracle/install-orcl-driver.md)

## Java Application Server

The Java Application Server is Third Party Software not provided with Identity Analytics.  
Apache Tomcat 9.0 (all Operating Systems) is the only application server supported by Identity Analytics.

## Web browsers

Web Browsers are the only web browsers supported by Identity Analytics are the "long term support" or the "For work" versions. As such, following you will find the list of the supported web browsers.

### Firefox

The current version of Firefox ESR is supported (Extended Support Release). For more information please refer to the following link:  
https://www.mozilla.org/en-US/firefox/enterprise/

### Chrome

The supported version of chrome is the current version of the Chrome Browser for Businesses. See the following link for more information:  
https://chromeenterprise.google/

### Microsoft Edge

Please refer to the Microsoft's official documentation for more information :

https://learn.microsoft.com/en-US/deployedge/microsoft-edge-support-lifecycle

> **Note:** Internet explorer is no longer recommended see : https://techcommunity.microsoft.com/t5/Windows-IT-Pro-Blog/The-perils-of-using-Internet-Explorer-as-your-default-browser/ba-p/331732

[^1]: Please note that as of version Braille, Oracle is only supported in the case of existing and deployed projects. In the case of a new project only Microsoft SQL server and PostgreSQL are supported.

[^2]: all versions of java 17

[^debianLTS]: Please refer to Debian's official documentation for more information: https://wiki.debian.org/LTS

[^rhel9]: Please refer to Red Hat's official documentation for more information: https://access.redhat.com/support/policy/updates/errata#Maintenance_Support_2_Phase
