---
title: "Data loading"
description: "Send source files to a Self-Managed Identity Analytics instance and load them into a timeslot."
---

# Data loading

An instance shows data once an execution plan has loaded the source files of its data sources into a timeslot. This page describes this flow on the platform side. What the data sources contain and how they are collected is defined by the project.

```text
data sources ──> source files ──> import files ──> execution plan ──> validated timeslot ──> portal
                 (portal upload,    (controller)      (batch)
                  PowerShell extractor,
                  connectors)
```

## Starting points

| Situation | What you have | First timeslot |
| :-------- | :------------ | :------------- |
| Existing project | A project already in use, for example an on-premises installation migrated to Self-Managed, with its data sources and collectors | Either import the database of the previous installation (see [Data migration](../10-data-migration/)), or run a first execution plan with the usual source files, sent as described below |
| New project | The reference project of your version (see [Project repository](../02-prerequisites.md#project-repository)) | Declare the data sources in the portal, send their files, then run a first execution plan |

In both cases, the folders and file names expected by the collectors come from the project: keep them when you send the files.

## 1. Declare the data sources

The data sources are declared in the portal, in **Datasource Management**, **Configure**.

- **Standard data sources** (for example **Generic HR - File**, Active Directory or the standard connectors) are declared directly in the portal: **Add...**, choose the model, then configure the mapping of the file to the Identity Ledger. The settings of each model, the file definition and the mapping are described in the Datasource Management documentation of the project: `static/bw_dsm/generated_doc/datasource_management_en.html` in the Git repository of the project.
- **Custom data sources** (for example with the **Generic bridge - file** model) also need a collection configured in the project with iGRC Studio. Push this change to the Git repository of the project: see [Project changes](../11-operations/05-project-changes.md).

> [!warning] The name of a file-based data source must be exactly the name of its input file, including case. For a custom data source, the folder you enter is created under the import files, and the collectors read the file from `/importfiles/<FOLDER_NAME>` in the project.

## 2. Send the source files

The files sent to the instance are stored in the import files (`/data/importfiles`, project variable `importfiles`), each data source in its own folder. The standard connectors use folders derived from this variable, for example `{config.importfiles}/bw_aad_connector/` for Entra ID. The controller lists the current files in **Import files**, with their folder in the **Path** column.

Send them in one of these ways:

| Method | How | Use it for |
| :----- | :-- | :--------- |
| Portal | **Datasource Management**, **Status**, select the data source, **Upload data**, **Upload single file**. **Test datasource** checks the file before you load it. | Files you receive by hand, tests |
| PowerShell extractor | The extractor builds a ZIP file and uploads it to the instance. The extraction folders (`hr`, `ad`, `applications`…) must be at the root of the ZIP file: each one becomes a folder of the import files. See [PowerShell extraction](./03-powershell-extraction.md). | Active Directory, HR and other sources extracted on a Windows server |
| Connectors | The connectors declared in the portal run on the platform on a schedule and add their result to the import files. See the extractors pages, for example [Entra ID](../08-extractors/entra-id.md). | Entra ID, SailPoint and the other standard connectors |

Each sending creates an upload in the controller (**Uploads**). A file sent again with the same name and folder replaces the previous one in the import files.

## 3. Run an execution plan

Start an execution plan in one of these ways:

- in the controller, **Execution plans**, **Submit a new execution plan**;
- in the portal, **Datasource Management**, **Status**, **Launch an execution plan**, then confirm;
- on a schedule, in the **Scheduling** page of the configuration interface.

The plan downloads the last synchronised version of the project, takes the current import files, runs the collection and loads a new timeslot. Follow it in the controller: its status goes from **Submitted** to **Running**, then **Completed** or **Error**. The details page of the plan gives its message and **Download execution logs**. See [Controller](../09-components/controller.md).

To be notified by email at the end of each plan, add recipients in the controller (**Configuration**, **Notifications**).

## 4. Check the result

By default, the new timeslot is validated automatically at the end of a successful plan. A project can change this behaviour in its technical configuration (`docker.configuration`).

Once the timeslot is validated, the portal displays the data. A user sees the portal only if an account with the same login exists in this timeslot.

| Problem | What to check |
| :------ | :------------ |
| A data source is not loaded | The file name and folder match the data source declaration (case included), and the file is listed in **Import files** |
| The plan ends in **Error** with `can not find /data/importfiles/<folder>/<file>` | A collector of the project expects a file that the plan did not receive. Send it as described in step 2, in a folder named exactly `<folder>` (case included), and check that it is listed in **Import files**. Only the files listed there are passed to the plan: a file copied directly to the controller volume is ignored |
| The plan ends in **Error** with `an issue occured with your license` | The license is expired or not valid: see [Troubleshooting](../11-operations/02-troubleshooting.md) |
| The plan fails at the start with `last synched GIT hash is empty` | The project has not been synchronised yet: check the logs of the `git-server` pod |
