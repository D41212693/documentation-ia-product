---
title: Collect line wizard
Description: Documentation related to the generation of a collect line
---

# Collect line wizard

## Introduction

The data collection wizard provides the user with a dedicated interface to map the original import data to attribute in Identity Analytics data cube. The product will then automatically generate the corresponding data collection lines.

The generated elements can then be used out of the box to run an execution plan. Further customizations to the generated files can be applied if needed, thanks to the high customization capabilities of Identity Analytics.

The following paragraphs detail the process from the design of the application model to the launch of generated collect:

- Step 1: Configure the mapping between the import data and the data cube
- Step 2: Generation of the data collection lines
- Step 3: Customize and/or adapt the generated data collection lines
- Step 4: running an execution plan

> In the following pages the terms **collect** and **application model** are used interchangeably and have the same meaning.

## Create a New Application Model

### Prerequisites

The data collection wizard is included in an add-on called `bw_appmodeltemplates`.

This add-on is not included by default in the product and must be downloaded from the [Radiant Logic file sharing site](../../downloads/01-downloads#add-ons-facets) (Customer Downloads > Facets > bw_appmodeltemplates).

The installation of the add-on provide the access to the data collection wizard.

### Application Model

Once the `bw_appmodeltemplates` add-on installed, you can access the wizard to generate an application model in the audit menu: "New..."

![Application model new](./images/new_appmodel.png "Application model new")

This menu opens a dialog box to create an application model file with the extension `.appmodel`. This file contains all information relating to the application model.

The `.appmodel` file must be located in the 'silos' folder.

![Application model start dialog](./images/application_model_wizard_1.png "Application model start dialog")

The next dialog allows the configuration of the application model, including:

- The application model identifier
- The selection of the Application model template

![Application model start dialog 2](./images/application_model_wizard_2.png "Application model start dialog 2")

The application model identifier will be used as a silo identifier and, as such, it must be unique in the project. Please note that the identifier can not be changed once the wizard is finished.

The identifier is also used to create all necessary folders in the project (silos, collectors...).

### Templates

The application model include a series of templates that determine the structure of the generated data collection lines.

The list of application models will increase as the `bw_appmodeltemplates` Add-On evolves in time. As is stands the add-on includes :

- An HR model to create data collection lines for HR information
- A Repository model to repositories, accounts, groups and groups members.
- A Right model template to load accounts, permissions and rights.

For more information on the included data models please refer to the following pages:

[Application model templates](collect-wizard-templates)

## Build

Once the creation wizard dialog completed the application model editor is opened.

![Application model editor](./images/appmodel_editor.png "Application model editor")

This editor allows the collect designer to define the collect mappings, the configuration of the silo and so on.

For more information on the configuration of the application model editor please refer to the following page:

[Application model templates](collect-wizard-templates)

Once all configuration completed the Build tab allows the user to generate all data collection lines and the documentation.

The following screenshot shows the Build tab:

![Application model build](./images/appmodel_editor_build.png "Application model build")

## Execution Plan

Once all the previously described steps have been completed it is possible to launch an execution plan to load the data to Identity Analytics ledger.
