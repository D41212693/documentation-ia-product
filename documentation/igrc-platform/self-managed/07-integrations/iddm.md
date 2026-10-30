---
title: "Identity Data Management Integration"
parent: "Integrations"
---

# Identity Data Management Integration

RadiantOne Identity Data Management (IDDM, formerly FID or VDS) is the Radiant Logic product that aggregates identity data from several directories and exposes it as an LDAP directory. This page is optional: follow it only if your organisation already uses Identity Data Management and you want Identity Analytics to read its users and groups from there, instead of extracting them from each source with the [PowerShell extractor](../06-configuration/03-powershell-extraction.md) or the [extractors](../08-extractors/entra-id.md).

Identity Analytics can read these sources through Identity Data Management:

- Active Directory domains: users and groups;
- Entra ID tenants: users and groups;
- LDAP directories: users and groups.

The integration has three parts:

1. In Identity Data Management, expose each source under a naming context (the root entry of an LDAP tree, for example `o=IDAAD`) with the structure described below.
2. In the configuration interface of Identity Analytics, set the connection to Identity Data Management.
3. In the portal, declare one data source per naming context. The extraction runs on a schedule, like the other connectors.

## Connection and network flow

The extractions run in the `extractor` pod of the instance, which connects to the LDAP port of Identity Data Management (2636 for LDAPS and 2389 for LDAP by default). Allow this flow from the cluster, or use a RadiantOne Secure Data Connector (SDC) group.

Set the connection in the configuration interface, **Identity Data Management** page: LDAP URL, port and the account used to read the data, then run the ping and authentication tests. See [Configuration interface](../06-configuration/01-config-ui.md#identity-data-management).

## Prerequisites in Identity Data Management

Identity Analytics requires the data in Identity Data Management to be organised in a specific way. The following sections describe the configuration for Active Directory, Entra ID and LDAP backends, in the Identity Data Management control panel.

### Active Directory backend configuration

The Active Directory domain backend must be configured in the Identity Data Management control panel. To create the required hierarchy in Identity Data Management, follow these steps:  

- In the "Directory Namespace" tab, create a new naming context `o=IDAAD` based on a virtual tree.

![Create the root naming context](./images/IDM-new-naming-context.png)

- In the newly created root, create a level. The label type must be `dc`. Next, enter a name for the label.   

![Create the IDDM label](./images/IDM-new-dc-label.png)

- Configure the mapping of the backend in the label. Select an LDAP backend mapping and point to the desired data source as the backend.  

![Configure the backend LDAP](./images/IDM-backend-mapping.png)

- Remove the DN remapping option for all attributes.  

![Remove the DN remapping option](./images/IDM-remove-dn-remapping.png)

To validate the configuration, check that the directory browser has the following information:

- The label is correctly named `dc=XXXX`. 
- The attribute `objectClass=domain` is present for the label. 
- The users and groups are present.  

### Entra backend configuration

The connection to the Entra domain backend must be configured as an "Mgraph" type datasource in the "Data sources" tab of the "Data catalogue". Once the configuration is finalized, it is necessary to update the default schema used for the user configuration.
Change the primary key from UserPrincipalName to ID.

![Update the default schema for Entra](./images/IDDM-entra-schema-configuration.png)

Once the backend and the schema are updated, navigate to the "Namespace Design" of the "Directory Namespace" to configure the namespace.

- Create a new level, such as o=IDAEntra, and assign the organization objectClass to that level.
- Select the `o=IDAEntra` entry and create a new label under it based on the name of the Entra tenant pulled, `o=<tenant>` in this example. Add the organization and domain objectClasses to the label.
- Select the `o=<tenant>` level and create a new content level.  
![Create content level](./images/IDDM-Entra-content-level.png)
  - Select the configured datasource in the drop down list.
  - Select the datasource schema from the drop down list and click next.
  - Select the `user` object to mount. 
  ![User](./images/IDDM-entra-user-content-selection.png)
  - Navigate to the object build tab and add all necessary attributes (select all if in doubt)
- Select the `o=<tenant>` level and create a new content level.  
![Create content level](./images/IDDM-Entra-content-level.png)
  - Select the configured datasource from the drop down list.
  - Select the datasource schema from the drop down list and click next.
  - Select the `group` object to mount.  
  ![User](./images/IDDM-Entra-group-content-selection.png)
  - Navigate to the object build tab and add all necessary attributes (select all if in doubt).

Navigate to the directory browser page and select the `o=<tenant>` level under `o=IDAEntra` and validate the configuration.  

### LDAP backend configuration

The LDAP backend should have been configured in the Identity Data Management control panel. The configuration is similar to the configuration for the Active Directory backend. To create the required hierarchy in Identity Data Management, follow these steps:  

- In the "Directory Namespace" tab, create a new naming context `o=IDALDAP` based on a virtual tree.

- In the newly created root, create a label. The label type must be `dc`. Next, name the label.   

- Configure the mapping of the backend in the label. Select an LDAP backend mapping and point to the desired data source as the backend.  

- Remove the DN remapping option for all attributes.  

![Remove the DN remapping option](./images/IDM-remove-dn-remapping.png)

To validate the configuration, check that the directory browser has the following information:

- The label is correctly named `dc=XXXX`.  
- The attribute `objectClass=dcObject` is present for the label.  
- Users and groups are present.  

## Declaring the data source in Identity Analytics

In the Identity Analytics portal, expand the left menu and select the *Configure* option under the *Datasource Management* section.

![Datasource Management](images/radiantone_datasource_configure.png "Datasource Management")

This section displays a list of declared data sources. If none have been configured, the list will be empty. Click on the *Add* button to declare a new data source.

![Add Datasource](images/radiantone_datasource_add.png "Add Datasource")

A wizard will launch to help you declare a new datasource, including default configurations for connecting to RadiantOne Identity Data Management: 

- RadiantOne - Active Directory  
- RadiantOne - Entra  
- RadiantOne - Generic LDAP  
- RadiantOne - Generic Bridge  

Each datasource requires a sightly different configuration. This is covered in the following sections.   

### RadiantOne - Active Directory

Use this data source for an Active Directory backend exposed by Identity Data Management (see [Active Directory backend configuration](#active-directory-backend-configuration)).  

![RadiantOne - Active Directory datasource configuration](./images/R1-AD-datasource-configuration.png)

Fill out the required fields. Example:

- Datasource name: o=IDAAD
- Datasource description: Extraction of all AD domains included in the root naming context o=IDAAD

Options:

- **Search base**: the naming context created in Identity Data Management, `o=IDAAD` in this page. Check this value: depending on the version of the project, the default value can be `o=BWAD`.
- **AD domains**: keep `(objectClass=domain)`, the object class of the `dc` labels.
- **Search filter**: keep `(objectClass=*)`, or restrict it to limit the extraction.

> The `dc` attribute of each label is required.

Follow the steps shown in the wizard and complete them.  

### RadiantOne - Entra

This connector is used when connecting to an Entra backend through Identity Data Management. See [here](#entra-backend-configuration) for the required configuration of the namespace in Identity Data Management.  

![RadiantOne - Entra datasource configuration](./images/R1-Entra-datasource-configuration.png)

Fill out the required fields. Example:

- Datasource Name: o=IDAEntra
- Datasource description: Extraction of the Entra ID tenants included in the root naming context o=IDAEntra

Options:

- **Search base**: the naming context created in Identity Data Management, `o=IDAEntra` in this page. Check this value: depending on the version of the project, the default value can be `o=BWAzure`.
- **Azure AD tenants**: keep `(objectClass=organization)`, the object class of the tenant labels.
- **Search filter**: keep `(objectClass=*)`, or restrict it to limit the extraction.

Follow the steps shown in the wizard and complete them.  

### RadiantOne - Generic LDAP

Use this data source for a generic LDAP backend exposed by Identity Data Management (see [LDAP backend configuration](#ldap-backend-configuration)).  

![RadiantOne - Generic LDAP datasource configuration](./images/R1-LDAP-datasource-configuration.png)

Fill out the required fields. Example:

- Datasource Name: o=IDALDAP
- Datasource description: Extraction of the LDAP directories included in the root naming context o=IDALDAP

Options:

- **Search base**: the naming context created in Identity Data Management, `o=IDALDAP` in this page. Check this value: depending on the version of the project, the default value can be `o=BWLDAP`.
- **LDAP domains**: `(objectClass=dcObject)`, the object class of the `dc` labels (see [LDAP backend configuration](#ldap-backend-configuration)). Check this value: depending on the version of the project, the default value can be `(objectClass=domain)`.
- **Search filter**: keep `(objectClass=*)`, or restrict it to limit the extraction.

> The `dc` attribute of each label is required.

Follow the wizard for the final steps.  

### RadiantOne - Generic Bridge

Use this data source to extract any other data exposed by Identity Data Management, with your own LDAP URL, search base and filter. It does not use the connection of the configuration interface.

> The files it produces are not loaded by the standard collectors: the project needs a dedicated collector line (configured in iGRC Studio) to load them.  

![RadiantOne - Generic Bridge datasource configuration](./images/R1-bridge-datasource-configuration.png "RadiantOne - Generic Bridge datasource configuration")

After clicking Next, provide the required information in order to connect to the Identity Data Management LDAP URL: 
- **LDAP Server URL**
- **Login**
- **Password**
- **Search base** used for the data extraction
- **Search filter** used to select the LDAP objects that will be extracted
- **Attributes** to extract, leave this empty to extract all available attributes  

![Generic LDAP Params](images/radiantone_datasource_genericldap_params.png "Generic LDAP Params")

Click Next to schedule the extraction. You can specify the data extraction schedule(for example, you can extract data once a week or every day).

![Generic LDAP Schedule](images/radiantone_datasource_genericldap_schedule.png "Generic LDAP Schedule")

Click Next to proceed to the final step of the wizard, then click Finish to confirm the datasource declaration.

![Generic LDAP Finish](images/radiantone_datasource_genericldap_finish.png "Generic LDAP Finish")

## Declaring multiple Data sources

If you have multiple Identity Data Management URLs or different *Search Bases* to extract, you can repeat the process to declare one datasource by URL or by *Search Base*. Keep in mind that each datasource must have a unique datasource name.

## Testing the Data Extraction

Each declared datasource will have a corresponding connector created in the Controller.

You can reach the Controller using the URL `/controller`. Then click on the *Connectors* item in the menu that you will find on the left.

![Controller Connectors](images/radiantone_controller_connectors.png "Controller Connectors")

Click on the connector to see more details.

![Controller Connectors Details](images/radiantone_controller_details.png "Controller Connectors Details")

From here, you can use the *Submit a new test execution* button to trigger the data extraction process. This process takes a few minutes.

When the data extraction test process is complete, the details page will automatically refresh to display the test results.

![Controller Connectors Tests](images/radiantone_controller_testresults.png "Controller Connectors Tests")

You can download the test results to verify the extracted files.  

For more information on the controller, refer to the [Controller](../09-components/controller.md) documentation.


