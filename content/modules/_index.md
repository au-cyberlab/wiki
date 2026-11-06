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

Finally, `dependencies` allow us to specify modules to deploy in addition to the system, and is also how we supply packages to install through package managers like `apt` or `yum`. 

Playbooks to deploy each of these are run in the following order:
 1. `package`
 2. `installer`
 3. `system`
 4. `patch`

The `installer` field specifies modules to be run before the system. Typically, these modules are the common setup steps for systems, such as setting up docker or copying over the source files.

The `package` field specifies packages to install through the system's package manager. Currently, `apt`, `yum`, `dnf`, and any other package managers supported by `ansible.builtiin.package` are supported.

The `system` field specifies system dependencies i.e. other modules that the module may depend on. This can be either `internal`, which states that the system should be within the same subnet, or `external`, which states that the system should simply exist within the environment.

For more information on how each of these kinds of modules are specified, head to their pages to learn more.

 - [Installers][1]
 - [Systems][2]
 - [Patches][3]

Finally, under `playbooks`, we have a `deploy.yml`. Playbooks in modules are strictly kept as lists of tasks - that is, there is no host portion. When the module is deployed, the host portion is populated automatically for each system and as such we are able to abstract it away from `modules`.
For cleanliness, and code tidiness, we prefer to keep Ansible templates in a `templates` folder under the `playbooks` directory. This makes it easy to navigate.
Any source code should go in the `src` folder in the root of your module folder.
 
[1][./installers/_index.md]
[2][./systems/_index.md]
[3][./patches/_index.md]
[4][https://docs.ansible.com/projects/ansible/latest/index.html]
[5][../deployment/new/_index.md]
