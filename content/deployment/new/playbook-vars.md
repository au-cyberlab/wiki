---
title: Playbook variables
weight: 5
---

# Playbook variables

Playbook variables are defined under two different blocks in a [base config](../base-configs). They are passed to Ansible playbooks via the auto-generated Ansible inventory.

### System-wide variables

```yml
environment_vars:
  something: foobar
```

System-wide variables apply to all machines defined in the base config.

### Host-specific variables

```yml
host_vars:
  machine1:
    hello: world
  machine2:
    lorem: ipsum
```

Host-specific variables only apply to the machines under which they are specified.

> [!NOTE]
> Host-specific variables cannot be set for an edge router in the `host_vars` block.
