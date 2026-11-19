# Migration procedure

The following documentation details the requirements and the procedure to allow migration from an on-prem installation to a self-managed installation. This documentation includes

- The functional requirements before performing a migration
- the technical requirements to export and import an existing database.

## Functional requirements

To be eligible to migrate from an on-prem instance to self-managed the following requirements are necessary:  

- [ ] The project needs to be running the latest version of Descartes
- [ ] Databases schema version 40 (Descartes R5):
  - [Oracle](./export-oracle-database.md) and [Microsoft SQL server](./export-sqlsever-database.md) exports done following their respective procedures.
  - A Database dump for postgreSQL, see [here](./export-postgesql-database.md) for the required procedure.
- [ ] A version of the project running the latest version of IAP

### Eligible Architectures

- [ ] Simple Tomcat single web server architecture
- [ ] Authentication via Azure Idp, Ping or Okta via KeyCloak

### Eligible Volumetries

- [ ] Full loading time of a timeslot must be less than 4 hours
- [ ] The project must concern "Coarse grain" application access rights
- [ ] The Data access rights must be "limited" to sensitive resources
- [ ] The Data history limited is limited to 12 timeslots

### Limitations

Activiti's schema is not migrated. The migration is ONLY possible with NO running instances of a Workflow.

#### Project limitations

- [ ] No customizations not supported by IAP
- [ ] No static roles other than the default roles (user, technicaladmin, etc.) Dynamic custom roles remain possible
- [ ] No data “exported” to local disk via the project (local files generated at review output, etc.)

#### Data extraction and loading

- [ ] The use of `igrc_extract.[sh|cmd]` is not supported
- [ ] Extractions are performed via client-specific mechanisms or via the use of our Powershell scripts
- [ ] Orchestration must already be set up via a processing batch (external to the product) that consolidates all the data to be made available to IDA in a directory
- [ ] No need to launch batches other than the loading batch (eg: technical workflows, extraction, data export via views, etc.)

### Best Practices

Before performing the migration the best practices are to have the following elements:  

- [ ] A Diagram of pre-existing architecture required including
  - Sizings for portal, batch and database
  - The results of a DB_check (see documentation for more information)
  - Network relationships in the current architecture
  - if present the existing group mappings used for authentication
  - The current SMTP server settings

It is also a good practice to have metrics on the current on-prem performances:  

- [ ] Existing data loading times
- [ ] Existing information on page display timings
- [ ] The maximum number of simultaneously connected users
