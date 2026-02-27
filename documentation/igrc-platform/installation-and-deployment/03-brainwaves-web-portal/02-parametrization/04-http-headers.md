---
title: "HTTP Headers Access"
description: "HTTP Headers Access"
---

# HTTP Headers Access

This kind of access mechanism is possible when the AuthN/AuthZ process is delegated to an upstream component (a Shibbolett agent deployed in an Apache Web Server for instance).

In this setup, the AuthN/AuthZ module inserts user credential information into the HTTP request headers.

The diagram below illustrates the flow of this type of access:

![HTTP Headers Access architecture](./images/http_header_access_architecture.png)

The following configuration has been validated:
- Tomcat 8.5 (Java 8) and Tomcat 9 (Java 8 and 17)
- Operating systems: Windows and Linux


## Pre-requisites

First step is to download [this archive](./attachments/http_headers_configuration.zip) in the Web Server (i.e. the server that hosts the iGRC webapp).

The archive contains:

- The HTTP Headers Tomcat Valve (`httpheadersauthenticator-1.1.jar`)
- A configuration `Authenticator.properties` file in a dedicated folder (to deploy as-is in the right destination, see later)

Ensure that:

- Tomcat instance is installed and available
- The operator has RW privileges in needed files and folders to proceed to the installation

## Installation 

> If you are using Linux, be mindful of files and folders permissions.

The objective of this setup is to install an Authenticator and configure Tomcat to use it for handling the HTTP headers received from the web server. Based on this information, Tomcat will build and assign a Principal that is then passed to the Identity Analytics web application. We will use these variables for the installation:

|               Variable                |                                  Description                                   |              Example value               |
| :-----------------------------------: | :----------------------------------------------------------------------------: | :--------------------------------------: |
|         `TOMCAT_CONF_FOLDER`          |                Folder that contains Tomcat configuration files                 |               /etc/tomcat8               |
|       `TOMCAT_LIB_FOLDER_HOME`        |             Folder that contains all libraries used by the Tomcat              |          /usr/share/tomcat8/lib          |
|       `HTTP_HEADER_LOGIN_FIELD`       |               HTTP Header field name that contain the user login               |              samAccountName              |
|      `HTTP_HEADER_MEMBERS_FIELD`      |     HTTP Header field name that contain all AD groups granted to the user      |                isMemberOf                |
| `HTTP_HEADER_MEMBERS_SEPARATOR_FIELD` | Separator character used for user groups list in `<HTTP_HEADER_MEMBERS_FIELD>` |                    ;                     |
|    `HTTP_HEADER_ROLE_MAPPING_FILE`    |         File that contains mapping between AD groups and Portal roles          |          rolemapping.properties          |
|             `WEBAPP_NAME`             |                         Name of the iGRC Tomcat webapp                         |                 sandbox                  |
|        `WEB_INF_WEBAPP_FOLDER`        |                  **WEB-INF** folder of the iGRC Tomcat webapp                  | /var/lib/tomcat8/webapps/sandbox/WEB-INF |

Follow these steps:

- Unzip the archive in the `<TOMCAT_LIB_FOLDER_HOME>` folder

![HTTP Headers archive unzip](./images/http_headers_archive_unzip.png)

Ensure you have proper folder and files permissions if needed.

Inside `<TOMCAT_CONF_FOLDER>` folder:

- Create a file `HTTPHeadersAuthenticator.properties` with the following content and enter appropriate values:

```properties
defaultRoleList=user
loginHttpHeader=<HTTP_HEADER_LOGIN_FIELD>
rolesHttpHeader=<HTTP_HEADER_MEMBERS_FIELD>
rolesSeparator=<HTTP_HEADER_MEMBERS_SEPARATOR_FIELD>
roleMapping=<HTTP_HEADER_ROLE_MAPPING_FILE>
```

Inside `<TOMCAT_CONF_FOLDER>` folder:

- Create a `<HTTP_HEADER_ROLE_MAPPING_FILE>` file
- Fill it according to your context and following the below format

```properties
<SHIBBOLETT_ROLE1>=<PORTAL_ROLE1>
<SHIBBOLETT_ROLE2>=<PORTAL_ROLE2>
...
<SHIBBOLETT_ROLEn>=<PORTAL_ROLEn>
```

Copy the `web.xml` file from the `<WEB_INF_WEBAPP_FOLDER>` folder to the `<TOMCAT_CONF_FOLDER>` folder

- Rename the copy as `<WEBAPP_NAME>-web.xml`
- Set the rights accordingly
- At the end of the file, in the `web-app` element, update the Authentication method (see below)

```xml
<login-config>
                <auth-method>HTTPHEADERS</auth-method>
</login-config>
```

> If you regenerate the WAR file, ensure that the web.xml file has not been altered. If it has, you will need to repeat this configuration.

- Create a file `<WEBAPP_NAME>.xml` under `<TOMCAT_CONF_FOLDER>/Catalina/localhost` folder with the below content
  - Set the rights accordingly

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context override="true" altDDName="conf/<WEBAPP_NAME>-web.xml">
        <Manager pathname="" />
</Context>
```

> Ensure the relative path for the deployment descriptor is correct. The base used is the `$CATALINA_HOME` global variable.

- Open the `server.xml` file in the `<TOMCAT_CONF_FOLDER>` folder.
  - If present, remove the **org.apache.catalina.realm.UserDatabaseRealm** realm.
  - After this removal, if the the **org.apache.catalina.realm.LockOutRealm** realm is empty, you can remove that as well.

```xml
<!-- Use the LockOutRealm to prevent attempts to guess user passwords
    via a brute-force attack -->
    <Realm className="org.apache.catalina.realm.LockOutRealm">
    <!-- This Realm uses the UserDatabase configured in the global JNDI
    resources under the key "UserDatabase".  Any edits
    that are performed against this UserDatabase are immediately
    available for use by the Realm.  -->
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
        resourceName="UserDatabase"/>
<!-- </Realm> -->
```

## Debug

To debug the authentication settings, you can add the following information at the bottom of your `<TOMCAT_CONF_FOLDER>/logging.properties`:

```properties
# To see debug messages for HttpHeader Valve, uncomment the following line
com.brainwave.utils.level = ALL
```

Once all these steps have been performed, you can restart the Tomcat service. Logs should be visible in `catalina` log files.

## Security

Ensure the Tomcat is listening only on localhost and reachable only from Apache for HTTP protocol. You can verify this in the `<TOMCAT_CONF_FOLDER>/server.xml`.

![HTTP Headers archive unzip](./images/http_headers_server_xml.png)

## Downloads

[http_headers_configuration.zip](./attachments/http_headers_configuration.zip)

