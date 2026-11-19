---
title: "Project migration"
description: "Adapt an on-premises Identity Analytics project to run on Self-Managed."
---

# Project migration

Before running an on-premises Identity Analytics project on Self-Managed, update it to use the configuration settings and file locations required by the platform. The simplest approach is to merge the reference project for your target Self-Managed version into your existing project.

## Steps

1. Review your project's `.gitignore` files. Adjust any rules that could prevent required files from being merged, such as rules that exclude the entire configuration folder.

2. Create a branch in your project's Git repository containing the reference project provided by Radiant Logic for your target Self-Managed version.

3. In iGRC Studio, merge the reference project branch into the latest version of your project. Resolve any merge conflicts, which are usually limited and straightforward.

4. Configure the variables introduced by the merge. Self-Managed uses the following locations for source and working files:

    | Variable | Value | Purpose |
    | :------- | :---- | :------ |
    | `importfiles` | `/data/importfiles` | Source files read by collectors |
    | `bwdsm_sourcepath` | `/data/sourcefiles` | Source files read by the data source manager |
    | `bwdsm_workpath` | `{config.importfiles}/bw_dsm/work` | Working folder for the data source manager |
    | `bwdsm_genericapp_sourceFolder` | `{config.importfiles}/bwdsm_generic_app` | Application data mapped in the data source manager |
    | `bw_generic_hr_sourceFolder` | `{config.importfiles}/bwdsm_generic_hr` | HR data mapped in the data source manager |
    | `ad_sourceFolder` | `{config.importfiles}/bw_activedirectory/` | Active Directory data |
    | `aad_sourceFolder` | `{config.importfiles}/bw_aad_connector/` | Microsoft Entra ID data |

    Check the reference project for your target version, as new releases may introduce additional variables.

5. Update collectors and discoveries to read files from `{config.importfiles}` instead of on-premises paths.

6. Replace on-premises extraction methods, such as `igrc_extract` and scheduled server scripts, with Self-Managed extractors or file uploads to the controller. See the Configuration and Extractors pages.

7. Commit and push the migrated project branch. Configure the instance to use this branch through the `git.default` Helm values or the Git Project page in the configuration interface.

After migrating the project, import the database if you want to retain historical data (see [Import database](./import-database.md)), then configure authentication for the instance.
