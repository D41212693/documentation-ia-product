---
title: "Data loading"
description: "Send source files to a Self-Managed Identity Analytics instance and load them into a timeslot."
---

# Data loading

To display data in an instance, run an execution plan that loads the source files into a timeslot. This page describes the platform-side workflow. The project defines what each data source contains and how its data is collected.

```
Data sources → Source files → Import files → Execution plan → Validated timeslot → Portal
               Uploaded or    Managed by    Run by the
               collected      the controller batch
```

## Starting points

| Situation | What you have | How to create the first timeslot |
| :-------- | :------------ | :------------------------------ |
| Existing project | A project already in use, such as an on-premises installation being migrated to Self-Managed, with its data sources and collectors | Either import the database from the previous installation (see [Data migration](../10-data-migration/)) or send the usual source files as described below and run an execution plan. |
| New project | The reference project for your version (see [Project repository](../02-prerequisites.md#project-repository)) | Declare the data sources in the portal, send their source files, and run an execution plan. |

In both cases, preserve the folder and file names expected by the project's collectors when sending files.

## 1. Declare the data sources

In the portal, go to **Datasource Management** > **Configure**.

- Standard data sources, such as Generic HR - File, Active Directory, and the standard connectors, are configured directly in the portal. Select Add..., choose a model, and configure the mapping between the source file and the Identity Ledger. For model settings, file definitions, and mapping details, see the project's Datasource Management documentation at `brainwave/static/bw_dsm/generated_doc/datasource_management_en.html` in the project Git repository.
- Custom data sources, such as those that use the Generic bridge - file model, also require a collection configured in the project with iGRC Studio. Push this configuration change to the project Git repository. See [Project changes](../11-operations/05-project-changes.md).

> [!warning]
> For a file-based data source, the data source name must exactly match the input file name without its extension, including case. For example, use `HR` for `HR.csv`.
>
> For a custom data source, the folder you specify is created in the import files directory. The project's collectors read the file from `/importfiles/<FOLDER_NAME>`.

## 2. Send the source files

The instance stores source files in the import files directory (`/data/importfiles`, referenced by the project variable `importfiles`), with a separate folder for each data source. Standard connectors use folders derived from this variable, such as `{config.importfiles}/bw_aad_connector/` for Entra ID.

In the controller, the Import files page lists the available files. The Path column shows each file's folder.

Use one of the following methods to send files:

| Method | How it works | Use it for |
| :----- | :----------- | :--------- |
| Portal | Go to Datasource Management > Status, select the data source, and select Upload data > Upload single file. Use Test datasource to check the file before loading it. | Manually received files and testing |
| PowerShell extractor | The extractor creates a ZIP file and uploads it to the instance. Place the extraction folders (`hr`, `ad`, `applications`, and so on) at the root of the ZIP file. Each becomes a folder in the import files directory. See [PowerShell extraction](./03-powershell-extraction.md). | Active Directory, HR, and other sources extracted on a Windows server |
| Connectors | Connectors configured in the portal run on the platform according to a schedule and write their output to the import files directory. See the relevant extractor documentation, such as [Entra ID](../08-extractors/entra-id.md). | Entra ID, SailPoint, and other standard connectors |

Each transfer creates an entry on the controller's Uploads page. Sending a file with the same name to the same folder replaces the existing file.

## 3. Run an execution plan

Start an execution plan using one of the following methods:

- In the controller, go to Execution plans > Submit a new execution plan.
- In the portal, go to Datasource Management > Status > Launch an execution plan, then confirm.
- In the configuration interface, use the Scheduling page to schedule execution plans.

To start a plan from a script, use the operations manager's execution plan API. This API is not published by default. See [ADD-006](../11-operations/04-upgrade/migration-3.8.0.md#add-006-standalone-timeslot-purge-and-post-plan-opt-out).

The plan downloads the latest synchronized version of the project, uses the current import files, runs the collection, and loads a new timeslot.

Monitor progress in the controller. The plan's status changes from Submitted to Running, then to Completed or Error. The plan's details page displays its message and provides a Download execution logs option. See [Controller](../09-components/controller.md).

To receive an email notification when each plan finishes, add recipients under Configuration > Notifications in the controller.

## 4. Check the result

By default, a successful execution plan automatically validates the new timeslot. The project can change this behavior in its technical configuration (`docker.configuration`).

After validation, the portal displays the timeslot's data. A user can access the portal only if the timeslot contains an account with the same login.

| Problem | What to check |
| :------ | :------------ |
| A data source is not loaded | Check that the file name and folder exactly match the data source configuration, including case, and that the file appears on the Import files page. |
| The plan ends with an Error status and the message `can not find /data/importfiles/<folder>/<file>` | A project collector expects a file that was not passed to the plan. Send the file using a method described in step 2, placing it in a folder named exactly `<folder>`, including case. Check that the file appears on the Import files page. Only files listed on that page are passed to the plan; files copied directly to the controller volume are ignored. |
| The plan ends with an Error status and the message `an issue occured with your license` | The license has expired or is invalid. See [Troubleshooting](../11-operations/02-troubleshooting.md). |
| The plan fails at startup with the message `last synched GIT hash is empty` | The project has not yet been synchronized. Check the logs of the `git-server` pod. |
