---
title: "Project migration"
description: "Adapt an on-premises Identity Analytics project to run on Self-Managed."
---

# Project migration

A project that runs on-premises must be adapted before it runs on Self-Managed: the platform expects configuration elements and file locations that an on-premises project does not have. The simplest way is to merge the reference project of the target Self-Managed version into your project.

## Steps

1. **Check the `.gitignore` files** of your project. Restrictive rules (for example ignoring the whole configuration folder) prevent a proper merge: relax them first.
2. **Create a branch** in the Git repository of your project, holding the reference project of the target Self-Managed version provided by Radiant Logic.
3. **Merge** that branch into the latest version of your project, in the Studio. Conflicts are usually few and simple to solve.
4. **Set the new configuration variables** brought by the merge. In Self-Managed, source files are read from fixed folders inside the platform:

    | Variable | Value | Purpose |
    | :------- | :---- | :------ |
    | `importfiles` | `/data/importfiles` | Folder of the final source files, read by the collectors |
    | `bwdsm_sourcepath` | `/data/sourcefiles` | Folder the data source manager reads its source files from |
    | `bwdsm_workpath` | `{config.importfiles}/bw_dsm/work` | Work folder of the data source manager |
    | `bwdsm_genericapp_sourceFolder` | `{config.importfiles}/bwdsm_generic_app` | Applications mapped in the data source manager |
    | `bw_generic_hr_sourceFolder` | `{config.importfiles}/bwdsm_generic_hr` | HR data mapped in the data source manager |
    | `ad_sourceFolder` | `{config.importfiles}/bw_activedirectory/` | Active Directory data |
    | `aad_sourceFolder` | `{config.importfiles}/bw_aad_connector/` | Microsoft Entra ID data |

    Compare with the reference project of your version: new variables can appear between releases.
5. **Update the collectors and discoveries** so that they read their files from `{config.importfiles}` instead of on-premises paths.
6. **Replace the on-premises extractions** (`igrc_extract`, scheduled scripts on the server) by the Self-Managed extractors or by uploads to the controller. See the Configuration and Extractors pages.
7. **Commit and push** the branch, and set it in the `git.default` values of the instance (or in the configuration interface, **Git Project**).

Then migrate the database if you keep the history (see [Import database](./import-database.md)), and configure the authentication of the instance.
