---
title: Authentication and authorization configuration
description: Authentication and authorization configuration
---

# Authentication and authorization configuration

All resources provided by Self-Managed Identity Analytics are protected by the Authentication and Authorization server.
It is implemented with [RedHat Keycloak](https://www.keycloak.org). This chapter describes how to implement supported use cases. For detailed explanations about options, please refer to the [official documentation](https://www.keycloak.org/docs/latest/server_admin/index.html).

## Supported Identity Providers

Self-Managed Identity Analytics supports:

- **Local Directory**: this the default identity provider. Users, its credentials and groups/roles are stored in the Keycloak's internal repository.
- **User Federation with Active Directory**: the user federation allows Keycloak to delegate authentication and authorization based on groups to Active Directory through LDAP(s) v3 protocol.
- **Identity Provider with Azure Active Directory**: Azure AD will provide users authentication and authorization using OpenID Connect or SAML v2 protocol .

## Static built-in roles

This package comes with default configuration (clients, roles, etc.). The roles provided by Identity analytics are:

- **user**: standard user. It allows access to the Governance Portal user interface
- **designer**: allows the design of Mashup Dashboards in the Governance Portal user interface
- **technicaladmin**: allows the Technical Administration in the Governance Portal user interface and miscellaneous technical interfaces (configuration and controller UIs)
- **functionaladmin**: allows the Functional Administration in the Governance Portal user interface

All these roles are defined in the client `identity-analytics` of the instance realm. Each instance has its own realm, named after the namespace where it is installed (`<IDA_NAMESPACE>`).

> [!warning] If you change the configuration of those roles or delete them, the Identity analytics solution will not work anymore.

## Keycloak Administration Console

The Keycloak administration console of the Identity Analytics realm is available at `https://{{HOSTNAME}}{{PATH_PREFIX}}/auth/`, where `{{HOSTNAME}}` is the value of `global.externalDns` and `{{PATH_PREFIX}}` the value of `global.pathPrefix`.

You log in with the technical administrator account, `setup` by default (`global.technicalAdmin.username`). How you get its initial password depends on what was set in the values file at installation time.

### Password policy

The password policy of the realm (length, character classes, history, expiration) is defined in the values of the Helm chart, by the `keycloakSettings.password*` values (for example `keycloakSettings.passwordMinLength`). Its default values can change from one version to another, so they are not listed here: check them in the `values.yaml` file of the chart version you deploy, or with:

```bash
helm show values <chart> --version <version> | grep -i "password"
```

Every password you set for the `setup` account, in the values or in Keycloak, must comply with this policy.

### Case 1: password set in the values

When `global.technicalAdmin.password` is set, it is used as the initial password of the `setup` account.

- It must comply with the password policy, otherwise the environment setup job fails.
- It is also stored in the Kubernetes Secret `keycloak-app-realm-admin-credentials` (see Case 3).
- This initial password is temporary: you are asked to set a new password at first login.

### Case 2: no password but an email set in the values

When `global.technicalAdmin.password` is empty and `global.technicalAdmin.email` is set, a random password compliant with the password policy is generated during the installation, and the `setup` account gets this email address.

You can retrieve the generated password from the Secret `keycloak-app-realm-admin-credentials`, as described in Case 3.

If SMTP is configured in the values (`smtp.enabled: true` and its `smtp.*` settings), you can also set the password yourself without accessing the cluster:

1. Go to `https://{{HOSTNAME}}{{PATH_PREFIX}}/auth/` and click **Forgot password?** on the login screen.
2. Enter the username `setup` or its email address.
3. Keycloak sends an email with a link to reset the password. By default the link expires after 1 hour (`keycloakSettings.actionTokenGeneratedByUserLifespan`).
4. Follow the link and set a new password compliant with the password policy.

### Case 3: no password and no email set in the values

When `global.technicalAdmin.password` and `global.technicalAdmin.email` are empty, a random password compliant with the password policy is generated during the installation. It is stored in the Kubernetes Secret `keycloak-app-realm-admin-credentials`, in the namespace where Identity Analytics is installed.

1. Retrieve the initial password:

   ```bash
   kubectl get secret -n <namespace> keycloak-app-realm-admin-credentials \
     -o jsonpath="{.data.technical_admin_initial_password}" | base64 -d
   ```

2. Log in to the administration console with the `setup` account and this password.
3. This initial password is temporary: you are asked to set a new password at first login.

> [!warning] As no email is assigned to the `setup` account, the *Forgot password* link cannot be used for this account. To allow it later, add an email address to the account in the administration console and configure SMTP (see Case 2).

## Self-service user administration

Keycloak can notify end-users related to accounts and credentials lifecycle.  
The email configuration in Keycloak is managed by the Identity Analytics configuration UI (`https://{{HOSTNAME}}{{PATH_PREFIX}}/config`)

> Any changes in Keycloak, will be erased at next platform restart

## Local Directory configuration

There is nothing to do to use of this directory. Simply, declare users inside the realm of the instance (`<IDA_NAMESPACE>`) using the [administration console](https://www.keycloak.org/docs/latest/server_admin/index.html#using-the-admin-console).

All users declared in this directory must authenticate with the email or username and local credentials.

> Note that you cannot declare a user (with same username or email) in this directory that already exists in User Federation or Identity Provider.

To create a new user granted to this directory, follow the steps below.

Once connected to the Keycloak Administration Console:

1. Select the realm of the instance (`<IDA_NAMESPACE>`) from the dropdown box.
2. Click on the `Users` menu.
3. Then click on the button `Add user`.
4. Enter values for the `username` (not updatable after creation) and `email` fields. They can have the same values or different values. It depends how your Identity Analytics iGRC project is configured. This is the login used by the end-user when accessing the user interfaces such as Governance Portal.

    > [!note] Keycloak stores the username in lower case: `JDOE` becomes `jdoe`.

5. Enter other fields like `First name` and `Last name`.
6. Choose `Update Password` option in the select box `Required user actions` so that end-user must change the initial password during first connection.
7. Click on the `Create` button.
8. Once user successfully created, go the `Credentials` tab to setup initial password.
9. Click on `Set password` button.
10. Enter `Password` and `Password confirmation`.
11. Set `Temporary` field to `On` so that end-user must enter its own password.
12. Then click save.
13. Click on `Role mapping` tab.
14. Click on `Assign role` button.
15. Choose `Client roles` in the menu of the `Assign role` button.
16. In the search field, enter `identity-analytics` to select Identity Analytics static built-in roles to assign.
17. Check the roles to assign (ex. `user`)
18. Then click on `Assign` button.

## Active Directory user federation

The user federation with Active Directory allows Keycloak to delegate authentication and authorization based on groups to Active Directory through LDAP(s) v3 protocol.

The user federation is also known as external user storage. To support all the features of Keycloak, user information is stored locally except credentials. The User federation regularly synchronizes its local storage with Active Directory. See details on [how it works](https://www.keycloak.org/docs/latest/server_admin/index.html#_user-storage-federation).

This section explains how to configure Keycloak to look for users (identity with Active Directory credentials) and groups (for authorization based).

### Prerequisites

To enable Keycloak to query your Active Directory, you must declare a read-only service account granted to traverse the directory tree where your users and groups are located including referrals.

> Don't forget to allow the service account to follow referrals if you have multiple domains and forests.

Open the route from the cluster to the domain controllers on port 389 (LDAP, or LDAP with StartTLS) or 636 (LDAPS). For LDAPS or StartTLS, the certificate of the domain controller must contain, in its subject alternative names, the host name used in the connection URL, and Keycloak must trust its certificate authority: see [LDAP over SSL and StartTLS](#ldap-over-ssl-and-starttls).

### User Federation required information

Get from your Active Directory administrator:

- [ ] `LDAP Host`: host name of the domain controller. With LDAPS or StartTLS, use the DNS name present in its certificate, not an IP address.
- [ ] `Service Account Bind DN`: Service Account distinguished name.
- [ ] `Service Account Bind Credentials`: Service Account credentials.
- [ ] `Users DN`: distinguished name from which users are searched for. If users that you will grant to Identity analytics solution are located in distinct nodes, choose the root DN common to those nodes. It is not possible to enter more than one DN to search for users.
- [ ] `Username LDAP attribute`: name of the LDAP attribute which is mapped to Keycloak username, usually `sAMAccountName`.
- [ ] `User search scope`: possible values are `One Level` or `Subtree`. First option means that users are search at the `Users DN` level. Second options means that users are searched whatever the level from `Users DN`.
- [ ] `Groups DN`: distinguished name from which groups are searched for. If groups assigned to LDAP users are located in distinct nodes, choose the root DN common to those nodes. It is not possible to enter more than one DN to search for groups.
- [ ] `Group name LDAP attribute`: name of the LDAP attribute which is mapped to the Keycloak group name. For instance `cn`.

### User Federation authentication

Once connected to the Keycloak Administration Console:

1. Select the realm of the instance (`<IDA_NAMESPACE>`) from the dropdown box.  
2. Select `User Federation` from the menu.  
3. Click on `Add new provider`dropdown list and choose `LDAP`.  
4. Enter **General options** and **Connection settings**  
    - `UI display name`: a name to identify your Active Directory users repository.
    - `Vendor`: choose `Active Directory`
    - `Connection URL`: LDAP connection URL of Active Directory: `ldap://<LDAP_HOST>:389` (plain LDAP or StartTLS) or `ldaps://<LDAP_HOST>:636` (LDAPS). See [gathered information](#user-federation-required-information).
    - `Enable StartTLS`: `On` only with an `ldap://` URL, to encrypt the connection. Leave it `Off` with `ldaps://`: the connection is already encrypted and StartTLS fails with `TLS is already enabled on this LDAP session`.
    - `Use Truststore SPI`: keep `Always` (default).
    - `Connection pooling`: set to `Off` when StartTLS is `On`: Keycloak refuses to save the provider otherwise (`Can not enable both StartTLS and connection pooling`). It can stay `On` with `ldaps://` or plain `ldap://`.
    - `Connection timeout`: LDAP connection timeout in milliseconds. For instance, 1000.
    - `Bind type`: select `simple` to connect to Active Directory with LDAP service account and password.
    - `Bind DN`: service account distinguished name. See [gathered information](#user-federation-required-information).
    - `Bind credentials`: password of the service account. See [gathered information](#user-federation-required-information).
5. Enter **LDAP searching and updating** fields
    - `Edit mode`: choose the `READ_ONLY` value so that LDAP attributes are one-way synched to Keycloak.
    - `Users DN`: distinguished name from which users are searched for. See [gathered information](#user-federation-required-information).
    - `Username LDAP attribute`: the LDAP attribute in Active Directory that fills the Keycloak username. The console proposes `cn` for Active Directory: replace it with `sAMAccountName` (see the warning below).  
  
    > [!warning]  
    > If you are using standard *iGRC project configuration*, `Username LDAP attribute` must be equal to `sAMAccountName` (same as `username` attribute mapper defined later in this section).  
    >
    > If users do not connect with `sAMAccountName` or `email`, then you must adapt the configuration *iGRC project configuration*, the provided iGRC view (`br_portal_identity`), for username to match the corresponding login in the right repository.

    - `Search scope`: recursive or one level user search. See [gathered information](#user-federation-required-information). For example, `Subtree`.
    - `Pagination`: set to `On` in case of large number of users.
    - `Referral`: set to `follow` if you have multiple domains or forests and the service account is allowed to follow referrals.
6. Enter **Synchronization settings** fields
    - `Import users`: set to `On`. Keycloak will import users at after configuration terminated.
    - `Batch size`: import `1000` users per transaction.
    - `Periodic full sync`: set to `On`. Execute a full synchronization from Active Directory to Keycloak.
    - `Full sync period`: full synchronization every `86400` seconds (24 hours).
    - `Periodic changed users sync`: set to `On`.  
    - `Changed users sync period`: synchronize changes in Active Directory to Keycloak every `14400` seconds (4 hours).
7. Enter **Cache settings** fields
    - `Cache policy`: MAX_LIFESPAN
    - `Max lifespan`: `3600000` milliseconds (1 hour)
8. Enter **Advanced settings** fields
    - `Trust email`: set to `On`. Active Directory is considered as authoritative source for email and verification is not required.
9. Click `Save` button

### LDAP over SSL and StartTLS

LDAPS (`ldaps://`, port 636) and StartTLS (`ldap://`, port 389, `Enable StartTLS` on) are supported. Keycloak checks the certificate of the domain controller against its truststore, and checks that the host name of the connection URL is in the certificate.

If the certificate of your domain controllers is issued by an internal certificate authority, add this authority to Keycloak:

1. Create a ConfigMap with the root certificate of the authority, in PEM format, in the Shared Services namespace. Add the intermediate certificates to the same file if the domain controllers do not send them.

    ```bash
    kubectl create configmap ldap-ca -n <SHARED_NAMESPACE> --from-file=ldap-ca.pem=<CA_FILE>.pem
    ```

2. Add the following block to `shared.values.yaml`, then run the `helm upgrade` of Shared Services. Keycloak restarts (about one minute) and logs `Found the following truststore files in the truststore paths`.

    ```yaml
    keycloak:
      extraVolumes:
        - name: ldap-ca
          configMap:
            name: ldap-ca
      extraVolumeMounts:
        - name: ldap-ca
          mountPath: /opt/keycloak/certs/ldap-ca
          readOnly: true
      extraEnvVars:
        - name: KC_TRUSTSTORE_PATHS
          value: /opt/keycloak/certs/ldap-ca
    ```

3. In the user federation, use `ldaps://<DC_FQDN>:636`, or `ldap://<DC_FQDN>:389` with `Enable StartTLS` on and `Connection pooling` off. Then click `Test connection` and `Test authentication`.

Keep this block in your values: an upgrade without it removes the authority from Keycloak. If you replace the certificate later, update the ConfigMap and restart Keycloak.

If `Test connection` returns `SSLHandshakeFailed` (or `AuthenticationFailure` with StartTLS), check the Keycloak logs:

| Message in the Keycloak logs | Cause |
| :--------------------------- | :---- |
| `PKIX path building failed … unable to find valid certification path to requested target` | Keycloak does not trust the authority of the certificate |
| `No subject alternative names matching IP address …` or `No subject alternative DNS name matching …` | The host of the connection URL is not in the certificate |
| `hostname of the server '…' does not match the hostname in the server's certificate` | Same cause, with StartTLS |

> [!note] This procedure was validated without FIPS mode. In FIPS mode, the truststore format accepted by Keycloak may differ.

### User Federation group mapping

Once connected to the Keycloak Administration Console, do the following steps to map authorizations to Active Directory groups.

When you created the User Federation in the previous section, Keycloak created default mappers. There is no mapper for the groups yet: you create it below.

| Mapper Name           | Mapper Type                      |
| --------------------- | -------------------------------- |
| creation date         | user-attribute-ldap-mapper       |
| email                 | user-attribute-ldap-mapper       |
| full name             | full-name-ldap-mapper            |
| Kerberos principal attribute mapper | kerberos-principal-attribute-mapper |
| last name             | user-attribute-ldap-mapper       |
| modify date           | user-attribute-ldap-mapper       |
| MSAD account controls | msad-user-account-control-mapper |
| username              | user-attribute-ldap-mapper       |

You will now check the configuration to create a new mapper for the Active Directory groups.

1. Select the realm of the instance (`<IDA_NAMESPACE>`) from the dropdown box.
2. Select `User Federation` from the menu.
3. Click on the card with the `UI display name` you have chosen in the previous section.
4. Click on `Add mapper` button.
5. Enter the general mapper fields
  - `Name`: the mapper will be named `groups`.
  - `Mapper type`: the type to map LDAP group is `group-ldap-mapper`.
6. Enter the `Group LDAP mapper` fields
  - `LDAP Groups DN`: distinguished name from which groups are searched for. See [gathered information](#user-federation-required-information).
  - `Preserve Group Inheritance`: set to `Off`. It is mandatory to disable this option if you have multiple parent groups for a group. This is often the case with nested groups.
  - `User Groups Retrieve Strategy`: choose `GET_GROUPS_FROM_USER_MEMBEROF_ATTRIBUTE` option
  - `Drop non-existing groups during sync`: set to `On`

### Synchronize Users and Groups

Once connected to the Keycloak Administration Console, do the following steps to synchronize Active Directory users and groups to Keycloak.

1. Select the realm of the instance (`<IDA_NAMESPACE>`) from the dropdown box.
2. Select `User Federation` from the menu.
3. Click on the card with the `UI display name` you have chosen in the previous section.
4. In the upper right dropdown list, select `Sync changed users` Action.

> When you finished the configuration above, the full synchronization is executed and may take long time depending on number of users and groups to synchronize.  
> `Remove imported` action with READ_ONLY edit mode (and with read-only service account) has no effect. Neither in Active Directory, neither in Keycloak. No users or groups are deleted from both sides even menu entry is enabled.

### Roles group mapping

When the synchronization is finalized, you will be able to assign Identity analytics static built-ins roles (Keycloak `Client Role`) to Active Directory groups.

#### Example

When an Active Directory user is member of an Active Directory group (for instance `BW_USERS`), the user will be granted to access (`user`) the Governance Portal user interface.

Once connected to the Keycloak Administration Console, do the following steps to assign a set of roles when an Active Directory user is member of an Active Directory group.

1. Select the realm of the instance (`<IDA_NAMESPACE>`) from the dropdown box.
2. Select `Groups` from the menu.
3. Select the Active Directory group (ex. `BW_USERS`)
4. Click on `Role mapping` tab.
5. Click on `Assign role` button.
6. Choose `Client roles` in the menu of the `Assign role` button.
7. In the search field, enter `identity-analytics` to select Identity Analytics static built-in roles to assign.
8. Check the roles to assign (ex. `user`)
9. Then click on `Assign` button.

## Azure Active Directory identity provider

This feature is based on [Identity Broker](https://www.keycloak.org/docs/latest/server_admin/index.html#_identity_broker) available in Keycloak.

In Keycloak, you can integrate with two main types of Identity Provider, depending on the security protocol supported by Identity Provider

- SAML v2.
- OpenID Connect v1.0.

### OpenID Connect protocol

#### OIDC Prerequisites

Ask your Azure Active Directory Administrator to create a new `App Registration` with the following properties

- `Redirect URI`: this is the address of Keycloak instance. This URI looks like `PROTOCOL`://`DNS_SERVICE_NAME`/auth/realms/`REALM`/broker/`ALIAS`/endpoint where
  - `REALM` is the realm of the instance, named after its namespace (`<IDA_NAMESPACE>`)
  - `PROTOCOL` is http or https depending on your deployment configuration
  - `DNS_SERVICE_NAME` is the DNS of the Identity analytics solution cluster
  - `ALIAS` is the unique identifier set in Keycloak to this identity provider (remember it for later configuration). We suggest to provide a string that represents the Identity Provider without space or special characters. For instance, `bw-aad-oidc` for Identity Analytics Azure Active Directory using OpenID Connect.
- `Token configuration`: ask for optional groups claim with properties
  - Add `Security groups`.
  - Add `Group ID` in Access Token.

Authorizations in Azure Active Directory are supposed to be assigned using security groups. So ask your Azure Active Directory Administrator to create new groups that will be mapped later against static built-in roles

- `Technical Administrator` group
- `Functional Administrator` group
- `Standard User` group
- `Designer` group

Finally the group `claims` should be configured in the `token configuration`. If necessary add a group claims and customize the token properties to `Group ID`.  

#### Configure Identity Provider with OpenID Connect

When the administrator finalized the `App Registration` configuration and the groups creation in Azure Active Directory, gather the following information  

- `Application (Client) ID` <a id="oidc-aad-client-id">
- `Client Secret` <a id="oidc-aad-client-secret">
- Group ID for each of Active Directory group
- `OpenID Connect Metadata Document` <a id="oidc-aad-discovery-endpoint">

Once connected to the Keycloak Administration Console, do the following steps to configure the Identity Provider, either using OpenID connect or SAML protocol.

1. Select the realm of the instance (`<IDA_NAMESPACE>`) from the dropdown box.
2. Select `Identity providers` from the menu.
3. Click on `Add provider` button and select `OpenID Connect v1.0` option.
4. Fill in `General options` section
    - `Alias`: this is the value you provided to your Azure Active Directory (see `Redirect URI`).
    - `Display name`: Enter the display name that will be displayed on login user interface.
5. Fill in `OpenID Connect settings`
    - `Discovery endpoint`: this value was provided by your Azure Active Directory (see [OpenID Connect Metadata Document](#oidc-aad-discovery-endpoint)). If the value is valid, you'll see a green checkbox at the end of the URL and the section `Show metadata` is filled with information downloaded from the Identity Provider.
    - `Client authentication`: choose `Client secret sent as post` option.
    - `Client ID`: this value was provided by your Azure Active Directory (see [Application (Client) ID](#oidc-aad-client-id)).
    - `Client Secret`: this value was provided by your Azure Active Directory (see [Client Secret](#oidc-aad-client-secret)).
6. Click on `Add` button
7. Open `Advanced` section to change some parameters
    - `Pass current locale`: set to `On`
    - `Backchannel logout`: set to `On`
8. Change following `Advanced settings`
    - `Trust Email`: set to `On`.
    - `Sync mode`: select `Force` option.
9. Click on `Save` button
10. Click on `Mappers` tab
11. Click on `Add mapper` button. Create as many mappers as Azure Active Directory groups that match static built-in roles

    -  Map `Technical Administrator` group to the role `technicaladmin`. Fill in the mapper form fields
        - `Name`: `Technical Administrator Group Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `Claim to Role` option
        - `Claim`: the claim that contains the Azure Active Directory groups is named `groups`
        - `Claim Value`: this is the group ID of `Technical Administrator` group provided by your Azure Active Directory administrator
        - `Role`
            - Client role: `identity-analytics`
            - Name: `technicaladmin`

    -  Map `Functional Administrator` group to the role `functionaladmin`. Fill in the mapper form fields
        - `Name`: `Functional Administrator Group Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `Claim to Role` option
        - `Claim`: the claim that contains the Azure Active Directory groups is named `groups`
        - `Claim Value`: this is the group ID of `Functional Administrator` group provided by your Azure Active Directory administrator
        - `Role`
            - Client role: `identity-analytics`
            - Name: `functionaladmin`

    -  Map `Standard User` group to the role `user`. Fill in the mapper form fields
        - `Name`: `Standard User Group Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `Claim to Role` option
        - `Claim`: the claim that contains the Azure Active Directory groups is named `groups`
        - `Claim Value`: this is the group ID of `Standard User` group provided by your Azure Active Directory administrator
        - `Role`
            - Client role: `identity-analytics`
            - Name: `user`

    -  Map `Designer` group to the role `designer`. Fill in the mapper form fields
        - `Name`: `Designer Group Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `Claim to Role` option
        - `Claim`: the claim that contains the Azure Active Directory groups is named `groups`
        - `Claim Value`: this is the group ID of `Designer` group provided by your Azure Active Directory administrator
        - `Role`
            - Client role: `identity-analytics`
            - Name: `designer`

### SAML v2.0 protocol

#### Prerequisites

Ask your Azure Active Directory Administrator to create a new `Enterprise Application` for SAML v2 authentication with the following properties:  

- **Entity ID**: `<PROTOCOL>://<DNS_SERVICE_NAME>/auth/realms/<REALM>`
- **Consumer Service URL (ACS)**: `<PROTOCOL>://<DNS_SERVICE_NAME>/auth/realms/<REALM>/broker/<ALIAS>/endpoint` where `<REALM>` is the realm of the instance (the name of its namespace),
    - `<PROTOCOL>` is http or https depending on your deployment configuration
    - `<DNS_SERVICE_NAME>` is the DNS of the Identity Analytics solution cluster
    - `<ALIAS>` is the unique identifier set in Keycloak to this identity provider (remember it for later configuration). We suggest to provide a string that represents the Identity Provider without space nor special characters. For instance, `bw-aad-saml` for Identity Analytics Azure Active Directory using SAML.
- **Claims**

| Claim name                                                         | Value                       |
| ------------------------------------------------------------------ | --------------------------- |
| Unique User Identifier (Name ID)                                   | user.userprincipalname      |
| name                                                               | user.userprincipalname      |
| http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname    | user.givenname              |
| http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname      | user.surname                |
| http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name         | user.userprincipalname      |
| http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress | user.mail                   |
| http://schemas.microsoft.com/ws/2008/06/identity/claims/groups     | user.groups [SecurityGroup] |

Authorizations in Azure Active Directory are supposed to be assigned using security groups. So ask your Azure Active Directory Administrator to create new groups that will be mapped later against static built-in roles.

- `Technical Administrator` group.
- `Functional Administrator` group.
- `Standard User` group.
- `Designer` group.

Finally the group `claims` should be configured in the `token configuration`. If necessary add a group claims and customize the token properties to `Group ID`.  

#### SAML v2 configuration

When the administrator finalized the `Enterprise Application` configuration and the groups creation in Azure Active Directory, gather the following information

- List of group IDs for each of Active Directory group.
- `App Federation Metadata Url` <a id="saml-aad-metadata-endpoint">.

Once connected to the Keycloak Administration Console, do the following steps to configure the Identity Provider, either using OpenID connect or SAML protocol.

1. Select the realm of the instance (`<IDA_NAMESPACE>`) from the dropdown box.
2. Select `Identity providers` from the menu.
3. Click on `Add provider` button and select `SAML v2.0` option.
4. Fill in `General options` section
    - `Alias`: this is the value you provided to your Azure Active Directory (see `Consumer Service URL (ACS)`).
    - `Display name`: enter the display name that will be displayed on login user interface.
5. Fill in `SAML Settings` section
    - `Service provider entity ID`: this is the value you provided to your Azure Active Directory (see `Entity ID`).
    - `Use entity description`: set to `On`.
    - `SAML entity descriptor`: enter the value provided by Azure Active Directory Administrator. See [App Federation Metadata Url](#saml-aad-metadata-endpoint). If the value is valid, you'll see a green checkbox at the end of the URL and the section `Show metadata` is filled with information downloaded from the Identity Provider.
6. Click on `Add` button.
7. Change following `SAML settings`
    - `Backchannel logout`: set to `On`.
    - `NameID policy format`: select `Unspecified` option.
    - `Allow create`: set to `On`.
    - `HTTP-POST binding response`: set to `On`.
    - `HTTP-POST binding for AuthnRequest`: set to `On`.
    - `Signature algorithm`: select `RSA_SHA256` option.
    - `SAML signature key name`: select `CERT_SUBJECT` option.
    - `HTTP-POST binding logout`: set to `On`.
    - `Want AuthnRequests signed`: set to `On`.
    - `Want Assertions signed`: set to `On`.
8. Change following `Advanced settings`
    - `Trust Email`: set to `On`.
    - `Sync mode`: select `Force` option.
9. Click on `Save` button.
10. Click on `Add mapper` button. Create as many mappers as Azure Active Directory groups that match static built-in roles

    -  Map `Technical Administrator` group to the role `technicaladmin`. Fill in the mapper form fields
        - `Name`: `Technical Administrator Group Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `SAML Attribute to Role` option
        - `Attribute Name`: the claim that contains the Azure Active Directory groups is named `http://schemas.microsoft.com/ws/2008/06/identity/claims/groups`
        - `Attribute Value`: this is the group ID of `Technical Administrator` group provided by your Azure Active Directory administrator
        - `Role`
            - Client role: `identity-analytics`
            - Name: `technicaladmin`

    -  Map `Functional Administrator` group to the role `functionaladmin`. Fill in the mapper form fields
        - `Name`: `Functional Administrator Group Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `SAML Attribute to Role` option
        - `Attribute Name`: the claim that contains the Azure Active Directory groups is named `http://schemas.microsoft.com/ws/2008/06/identity/claims/groups`
        - `Attribute Value`: this is the group ID of `Functional Administrator` group provided by your Azure Active Directory administrator
        - `Role`
            - Client role: `identity-analytics`
            - Name: `functionaladmin`

    -  Map `Standard User` group to the role `user`. Fill in the mapper form fields
        - `Name`: `Standard User Group Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `SAML Attribute to Role` option
        - `Attribute Name`: the claim that contains the Azure Active Directory groups is named `http://schemas.microsoft.com/ws/2008/06/identity/claims/groups`
        - `Attribute Value`: this is the group ID of `Standard User` group provided by your Azure Active Directory administrator
        - `Role`
            - Client role: `identity-analytics`
            - Name: `user`

    -  Map `Designer` group to the role `designer`. Fill in the mapper form fields
        - `Name`: `Designer Group Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `SAML Attribute to Role` option
        - `Attribute Name`: the claim that contains the Azure Active Directory groups is named `http://schemas.microsoft.com/ws/2008/06/identity/claims/groups`
        - `Attribute Value`: this is the group ID of `Designer` group provided by your Azure Active Directory administrator
        - `Role`
            - Client role: `identity-analytics`
            - Name: `designer`

11. Click on `Add mapper` button. Create mappers for last name, first name and email from Azure Active Directory to Keycloak user attributes
    - Map `givenname` SAML Attribute to the Keycloak user attribute. Fill in the mapper form fields
        - `Name`: `First name Attribute Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `Attribute Importer` option
        - `Attribute Name`: enter the claim name you supplied to your Azure Active Directory administrator (like `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`)
        - `Name Format`: select `ATTRIBUTE_FORMAT_URI` option
        - `User Attribute Name`: firstName

    - Map `surname` SAML Attribute to the Keycloak user attribute. Fill in the mapper form fields
        - `Name`: `Last name Attribute Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `Attribute Importer` option
        - `Attribute Name`: enter the claim name you supplied to your Azure Active Directory administrator (like `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`)
        - `Name Format`: select `ATTRIBUTE_FORMAT_URI` option
        - `User Attribute Name`: lastName

    - Map `emailaddress` SAML Attribute to the Keycloak user attribute. Fill in the mapper form fields
        - `Name`: `Email Attribute Mapper`
        - `Sync mode override`: choose `Force` option to always update Keycloak during login
        - `Mapper type`: choose `Attribute Importer` option
        - `Attribute Name`: enter the claim name you supplied to your Azure Active Directory administrator (like `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`)
        - `Name Format`: select `ATTRIBUTE_FORMAT_URI` option
        - `User Attribute Name`: email

### Troubleshooting

If the user has "access denied" message after successful IdP login, check the Keycloak user whether it has been granted the appropriate role. If not, verify:  

- Keycloak Identity Provider `Mappers` configuration
    - Do you define one mapper per role?
    - Is the Azure Group ID valid?
    - Does the AAD user member of the expected group?
- Azure Active Directory claims definition with your Administrator
    - Does the group claim correctly defined?
    - Does the `Token Configuration` in AAD `App registration` contain the optional group claim named `groups`?
