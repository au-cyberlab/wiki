---
title: What to deploy
weight: 2
---

# What to deploy

[Quickstart](../quickstart/) explains deployment of a pre-configured cyber range. If you want different systems, or different scenarios, you must create your own branch (from `main`).

> [!NOTE]
> If you are contributing to the CyberLab, **please follow [these guidelines][1] for branch creation.**

## Choosing systems

Each supported system has a wiki page, each containing a "Setup" section. It will say to add entries to two files:

1. `aws_infrastructure/deployment_config.yml`

    Each entry specifies a file describing a set of machines to deploy. Usually, this is a set of machines required to support a particular system:

    ```yml
    environments:
      folder: infrastructure_configs
      files:
        - email.yml
        - hospital.yml
        - mastodon.yml
    ```

    These files may be found under `aws_infrastructure/infrastructure_configs/base_configs/`.

    **Note:** To deploy *groups* of base configs, specify [lab configs](../how-to-deploy/lab-configs/).

2. `ansible/playbooks/plays_to_run.yml`

    Each entry specifies an Ansible playbook that deploys the software for a particular system.

    ```yml
    batches:
      batch:
        - container_projects/deploy_email.yml
        - container_projects/deploy_hospital.yml
        - container_projects/deploy_mastodon.yml
    ```

    These files may be found under `ansible/playbooks/`.

    **Note:** To control the order of execution, use multiple [batches](../how-to-deploy/batches/). Playbooks run concurrently by default.


[1]: https://github.com/UAdelaide/CyberLab/blob/main/CONTRIBUTING.md
