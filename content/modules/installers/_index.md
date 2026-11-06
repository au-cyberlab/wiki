---
title: Installer Modules
weight: 1
bookCollapseSection: true
---

# Overview

Installer modules contain the core playbooks that are required for setting up the infrastructure for a system.

## Specification

An installer module has the following structure and manifest:

```md
module
├── playbooks
│   └── deploy.yml
├── manifest.yml
└── values.yml
```

```yml
version: 1.0
kind: installer
```

You can specify `package` dependencies if you need, and they will be aggregated into the `package` field for any `modules` that include it.

`values.yml` is for environment variables that the module may need, or optional installer values that you may want to set defaults for.

Specifying the following would give any modules that include this module the variable `foo` with value `bar`.

```yaml
environment_vars:
  foo: bar
```

## Existing Installers

We have a bunch of useful installers already in the CyberLab, though you may find it beneficial to create your own if you find multiple modules have similar dependencies.

 - `github`: A module to copy the `src/` folder to `/opt/<module_name>` in your system machines. 
 - `docker`: A module to set up `docker` and its utilities.
 - `kubernetes`: A module to set up a k8s cluster over a set of one master and many workers.

 You can find more about each of these installers by navigating to their respective pages.
