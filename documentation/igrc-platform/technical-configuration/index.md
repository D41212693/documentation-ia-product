---
title: "The technical configuration"
description: "The technical configuration description"
---

# The technical configuration

## Overview

### Configuration selection

This version makes a clear separation between the project configuration (shared by all technical environments) and many technical configurations (for each platform).

The current technical configuration is now selected from the main menu. A project may have several configurations like DEV, TEST, UAT, PROD,... The name of the current configuration is stored in the local workspace (in metadata). When working in a team using GIT or SVN, the selection of a different configuration by one member does not alter any file in the project.

![Studio configuration menu](./images/studio_config_menu.png "Studio configuration menu")

### Configuration editor

The configuration is a file with the extension `.configuration` in the new folder `configurations`. Now, when executing batch or running the portal, a configuration must be specified.
![Studio configuration tabs](./images/studio_config_tabs.png "Studio configuration tabs")

Each configuration file contains the following elements:

- Values for project variables. The list of declared variables is still located in the project but the values are defined in each configuration
- Database settings declared as usual (with datasource dialog box) or manually
- Execution plan parameters (useful to test a new reconciliation policy on DEV without changing the execution plan settings for PROD)
- List of silos to include in the execution plan
- Mail server settings
- Web portal configuration which contains what was available in the `config.properties` in the previous version
- War export options
- Workflow settings including calendar and database configuration
- Mail from the batch

### Properties export

The legacy properties files (`datasource.properties`,`mail.properties`,...) may still be used to override some values. This icon lets you save these files with only the values usually overridden like database URL, login, password,...
![Studio Export the configuration](./images/studio_config_export.png "Studio Export the configuration")

### JNDI datasource

Now you can use a JNDI datasource. This means that the datasource to access the Identity Analytics database is declared in the Web container (like Tomcat) and not in `datasource.properties`. The icon is used to export a context.xml file to put in `conf/Catalina/localhost` folder inside Tomcat installation directory. Then this file should be renamed with the name of the webapp

![Studio export the JNDI context file](./images/studio_config_jndi.png "Studio export the JNDI context file")

This configuration has the following benefits :

- Possibility to use a connection pool with auto-reconnect feature
- Database configuration is not in the war anymore

### WAR settings

In the previous version, the only way to point to the studio project from the webapp was to edit the web.xml file. Now this is part of the configuration and you can use different options depending on the configuration (DEV, PROD,...)

![Studio WAR file configuration](./images/studio_config_war.png "Studio WAR file configuration")

### Sandbox name template

The name of the sandox can be changed in the configuration. This name will be used by the studio and the batch. This is a kind of template where the current date can be included. The product provides 6 date formats.

![Studio sandbox name configuration](./images/studio_config_sandbox.png "Studio sandbox name configuration")

## Defining custom configuration variables

To allow better customization of the project, configuration variables can be defined in the main project file, under the 'Project' tab, but this is not always convenient, as you might want to keep the variables defined for a specific purpose (the collection of an application, a workflow process, a facet, etc) separate from each other.

To achieve this a dedicated file with extension `.configvariables` that will hold the definitions of your variables. This type of file has to be put in the `/configurations` folder of the project.

To create such a file, simply choose the **Configuration variables** entry in the **New...** option of the main menu:

![New configuration variable file](./images/new-config-var-menu.png "New configuration variable file")

You will then access the configuration variables editor, which looks like this:

Each variable has a name, a type, and a display name. Those display names will be used to explain what the variable stands for and what kind of value is expected.
The typical value will be used as a default value if the variables are used to build a facet, otherwise it serves as an example that will be displayed when a value for the variable is prompted.

![Config variables](./images/configvariables.png "Config variables")

**WARNING:** Please bear in mind that the name of the variable has to be unique in the project. It is therefore **strongly** advised to avoid names like the `filename` given in the above example, as it is:

1. Non descriptive
2. Guaranteed to clash with another variable of the same name

A much better name would be for instance `sharepoint_extraction_filename`, and even better : `myvariables_sharepoint_extraction_filename`.
The only exception to this rule is if you use your variables to build a facet, in which case they will automatically be changed at facet installation to avoid conflicting names.

As for the variables defined in the main project file, the variable values will have to be entered in the `Variables` tab of your configuration file(s).

## Secure / encrypted configuration variables

You might want to store sensitive information in configuration variables, like **tokens** and/or **passwords**.

Any configuration variable which **name** contains `password` will be encrypted:

- By the studio when you edit the variable value, directly in the `.configuration` file
- By the portal or batch in the `.properties` files if the value is in clear text when read

An encrypted variable value will look like this: `nomacro:{crypt2}xxxx`.

### Encryption algorithm

Encryption is performed as follows:

- A 3-character **salt** is randomly generated.
- This **salt** is combined with an hardcoded 128-bit encryption key.
- The combined key is used to encrypt the password using **AES**.
- The information stored in the variable value is the concatenation of the salt and the encrypted password.

> The product **never generates the same encrypted** version of the password because the salt, which is randomly generated, **changes if the password is re-encrypted**.
