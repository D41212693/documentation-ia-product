---
title: "Portal and batch properties"
description: "Add Analytics properties per environment from the Helm values."
---

# Portal and batch properties

From `ida-helm` 3.8.0, you can add properties to the `config.properties` and `project.properties` files of the portal and the batch, from the Identity Analytics values. Use it to give each environment its own settings, for example the colors and the title of the portal, without changing the project in Git.

## Set the properties

```
portal:
  technicalConfiguration:
    configOverride: # config.properties
      interface.browser-title: "IDA - ACCEPTANCE"
      interface.header.title: "ACCEPTANCE"
      interface.header.color.bg.top: "D50000"
      pages.formats.date: "yyyy-MM-dd"
    projectOverride: # project.properties
      ias_maxentriestoreview: 5000
      my_project_property: my_value

batch:
  technicalConfiguration:
    configOverride: {} # config.properties
    projectOverride: # project.properties
      ias_permissionroletype: Role
      my_project_property: my_value
```

The keys of the existing project properties are the variables of the technical configuration in the Git project:

- `default.configuration` for the portal
- The value of `batch.technicalConfiguration.name` in the Identity Analytics values for the batch (`docker` by default).

Then run `helm upgrade`. The portal applies the properties when its pod restarts. The batch applies them at each execution plan and each purge.

- Colors are hexadecimal values without `#`. Numbers and booleans are written as text.
- Quote values that contain special characters. They are escaped when written.
- `interface.theme` is not applied.

## Properties you cannot change

Identity Analytics already writes some properties from its own settings: URLs, paths, the mail sender (`smtp.from.*`), the database, and the debug mode. A property that is already written keeps the product's value. The override is skipped with a warning in the portal init log, or in the batch log of the run, and the pod keeps running.

For example: `server.url`, `pages.debug`, `upload.directory`, `logPath`, `numberOfThreads`, `bwa_senderemail`, `ias_sqlinlimitvalue`, and the import folders (`*_sourceFolder`).

## Portal theme properties

| Setting              | Property                                           |
| :------------------- | :------------------------------------------------- |
| Browser title        | `interface.browser-title`                          |
| Header title         | `interface.header.title`                           |
| Logo                 | `interface.header.logo`                            |
| Header background    | `interface.header.color.bg.top`                    |
| Header title color   | `interface.header.color.fg.title`                  |
| Left menu background | `interface.header.color2.bgtoolbar`                |
| Selected menu item   | `interface.header.color2.bgselitem`                |
| Page colors          | `interface.pages.color.bg1` … `bg4`, `fg1` … `fg4` |
| Date format          | `pages.formats.date`                               |
