---
title: "Technical configuration"
description: "Technical configuration"
---

# Technical configuration

## Overview

This page covers how to manage project and technical configurations within the Studio environment, as well as how to securely encrypt sensitive configuration variables such as passwords and tokens. It explains how to select and edit configurations, define variables, and handle environment-specific settings like database connections, mail servers, execution plans, and WAR exports. You’ll also learn how to use JNDI datasources, customize sandbox names, and define your own configuration variables to better organize and adapt your project across multiple environments such as DEV, TEST, UAT, and PROD.

### Configuration Selection

This version clearly distinguishes between a **project configuration** (shared across all technical environments) and multiple **technical configurations** (defined for each platform).

The current technical configuration is selected from the **main menu**.  
A single project can include multiple configurations—for example, **DEV**, **TEST**, **UAT**, and **PROD**.  

The name of the currently active configuration is stored in the **local workspace metadata**. When working as a team using **Git** or **SVN**, each member’s configuration selection is local and does **not** modify any project files.


### Configuration Editor

Each configuration is stored as a `.configuration` file within the `configurations` folder.  
When running a batch process or launching the portal, an explicit configuration must now be selected.



Each configuration file includes:

- Values for **project variables**. (Variables remain declared in the project but get their specific values per configuration.)  
- **Database settings**, defined as usual through the datasource dialog or manually.  
- **Execution plan parameters** to test new policies (e.g., on DEV) without affecting other configurations like PROD.  
- **Silo list** for inclusion in the execution plan.  
- **Mail server** settings.  
- **Web portal** configuration (replacing what used to be in `config.properties`).  
- **WAR export** options.  
- **Workflow** settings, including calendar and database setup.  
- **Batch mail** settings.

### Properties Export

Legacy property files (such as `datasource.properties`, `mail.properties`, and others) can still be used to override specific values.  
The export icon allows you to generate these files with only the commonly overridden entries, such as database URL, login, and password.


### JNDI Datasource

You can now configure a **JNDI datasource**, meaning the connection to the Identity Analytics database is defined in the **web container** (e.g., Tomcat) instead of in `datasource.properties`.

Use the export icon to generate a `context.xml` file for placement in the `conf/Catalina/localhost` folder of your Tomcat installation. Rename this file to match your web application’s name.


**Benefits:**

- Enables connection pooling and automatic reconnect.  
- Removes database configuration details from the deployed WAR.


### WAR Settings

Previously, linking the web application to the Studio project required editing the `web.xml` file.  
Now, this mapping is defined within the configuration itself, allowing you to use different parameters for each environment (e.g., DEV, TEST, PROD).


### Sandbox Name Template

The **sandbox name** can be customized per configuration.  
This name is applied both in the Studio and in batch processes and can include a date stamp.  
The product supports **six date formats** for use in the template.


## Defining Custom Configuration Variables

To improve flexibility, you can define additional **configuration variables** outside the main project file.  
Although variables can still be created under the **Project** tab, dedicated configuration variable files help keep context-specific variables (for applications, workflows, facets, etc.) organized.

Create a new file with the `.configvariables` extension in the `configurations` folder.  
To do this, select **New… → Configuration variables** from the main menu:


This opens the **configuration variables editor**:

Each variable includes a **name**, **type**, and **display name**.  
Display names clarify the purpose and expected input for each variable.  
Typical values serve as defaults (when used in facets) or as examples displayed during input prompts.

> **Warning:** Each variable name must be **unique** within the project. Avoid generic names like `filename`, which can easily conflict with others.

Good examples include:
- `sharepoint_extraction_filename`  
- `myvariables_sharepoint_extraction_filename`

If variables are used to build facets, name conflicts are automatically resolved at facet installation.

As with variables defined in the main project file, you can specify their values in the **Variables** tab within your configuration file(s).

## Secure Encrypted Configuration Variables

Sensitive credentials, such as **tokens** or **passwords**, can be safely stored as configuration variables.

Any configuration variable whose **name** includes the word `password` is automatically encrypted in these scenarios:

- In the **studio**, when you edit the variable’s value directly in the `.configuration` file.  
- In the **portal** or **batch**, when the `.properties` files are read and any plaintext values are detected.

Encrypted values follow this format: `nomacro:{crypt2}xxxx`

### Encryption Algorithm

Encryption works as follows:

- A random **3-character salt** is generated.  
- This salt is combined with a hardcoded **128-bit encryption key**.  
- The combined key is used to encrypt the password using **AES**.  
- The stored variable value is a concatenation of the salt and the encrypted password.

> Because the salt is randomly generated each time, the **same password will never produce the same encrypted value** when re-encrypted.
