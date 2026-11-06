---
title: IV. Modules
weight: 4
bookFlatSection: true
---

# What are modules?

Modules are how we package systems for the CyberLab to use. Each module can be a installer, a system, or a patch. 
These modules are deployed with Ansible. You can learn more about Ansible by reading the documentation [here][4].
For more on Ansible in CyberLab, click [here][5].

Current Installer Modules:

 - Docker
 - Github
 - Kubernetes

Current System Modules:

 - Battery
 - DNS
 - Email
 - Expo IT
 - Forum
 - Gas
 - Hospital
 - Kafka
 - Mastodon
 - Minibus
 - Power System
 - Wazuh

Current Patch Modules:

 - DNSSEC
 - Expo IT Config
 - Mastodon Users
 - Wazuh Agent

## Module Specification

A module has the following directory structure:

```md
module
├── playbooks
│   └── deploy.yml
├── manifest.yml
└── values.yml
```

At the very minimum, a module must specify these values. They may change in accordance to the type of module you are creating.

The core of a module lies in the `manifest.yml` file.

For instance, we can take a look at the `manifest.yml` for the gas system.

```yml
version: 1.0
kind: system # installer, system, or patch
dependencies:
  installer:
    docker: latest
    github: latest

  package:
    git: latesthttps://docs.ansible.com/projects/ansible/latest/index.html
    curl: latest
    python3: latest

  system:
    external:
      power_system: latest
```

We supply a version for the module (currently CyberLab does not support versioning for individual modules. This will be added later.), and a specific kind.

Kinds are of the type listed above, and change the files required for a module to function properly.

Finally, `dependencies` allow us to specify modules to deploy before and/or after the system is deployed, and is also how we supply packages to install through package managers like `apt` or `yum`.

The `installer` field specifies modules to be run before the system. Typically, these modules are the common setup steps for systems, such as setting up docker or copying over the source files.

The `package` field specifies packages to install through the system's package manager. Currently, `apt`, `yum`, `dnf`, and any other package managers supported by `ansible.builtiin.package` are supported.

The `system` field specifies system dependencies i.e. other modules that the module may depend on. This can be either `internal`, which states that the system should be within the same subnet, or `external`, which states that the system should simply exist within the environment.

For more information on how each of these kinds of modules are specified, head to their pages to learn more.

 - [Installers][1]
 - [Systems][2]
 - [Patches][3]


 
[1][./installers/_index.md]
[2][./systems/_index.md]
[3][./patches/_index.md]
[4][https://docs.ansible.com/projects/ansible/latest/index.html]
[5][../deployment/new/_index.md]
