---
title: Github
weight: 2
---

# Github Installer

This installer copies the `src/` folder to `/opt/<module_name>/src`.

There is a variable that exists in the Ansible inventory that autofills `/opt/<module_name>`, which you can access under the variable `repo_folder`.

To include it, add the following snippet to your `manifest.yml` file under the `dependencies` field.

```yml
dependencies:
  installer:
    github: latest
```
