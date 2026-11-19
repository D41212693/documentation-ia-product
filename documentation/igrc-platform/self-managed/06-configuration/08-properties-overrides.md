---
title: "Portal and batch properties"
description: "Set Analytics properties per environment from the Helm values."
---

# Portal and batch properties

From `ida-helm` 3.8.0, you can set properties of the portal and the batch in the Identity Analytics values, for example to give each environment its own portal colors and title.

```yaml
portal:
  technicalConfiguration:
    configOverride:          # config.properties
      interface.header.title: "ACCEPTANCE"
      interface.header.color.bg.top: "D50000"
    projectOverride:         # project.properties
      ias_maxentriestoreview: 5000
      my_project_property: my_value

batch:
  technicalConfiguration:
    configOverride: {}
    projectOverride:
      ias_permissionroletype: Role
      my_project_property: my_value
```

Then run `helm upgrade`.

Properties managed by Identity Analytics, such as URLs and paths, cannot be changed: their override is ignored, with a warning in the log.
