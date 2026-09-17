---
title: System dependencies
weight: 2
---

# System dependencies

> [!CAUTION]
> This page does not yet reflect some recent major CyberLab changes. We are working to fix this.

One system may require another system be deployed alongside it to function. Required dependencies can be declared using a `dependencies` block in a [base config](../base-configs):

```yml
dependencies:
  setup:          # playbook dependencies
    - kubernetes
  system:
    local:        # local system dependencies
      - traffic
    ext:          # external system dependencies
      - minibus
```

> [!NOTE]
> Playbook dependencies are also specified in this section of the base config. They are described in more detail [here](../playbook-dependencies).

System dependencies are divided into local and external dependencies:

  - Machines from *local* dependencies will share the same subnet as your base config's machines;
  - Machines from *external* dependencies will be given dedicated subnets.

An external dependency is deployed exactly as if you had specifed it in [`aws_infrastructure/deployment_config.yml`](../../deployment/existing) instead. Specifying it again in `deployment_config.yml` or lab config will not deploy a replica.
