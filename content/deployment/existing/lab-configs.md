---
title: Lab configs
weight: 1
---

> [!CAUTION]
> Only the last example in this wiki page works. [The others should, but they don't.](https://github.com/UAdelaide/CyberLab/issues/455)

# Lab configs

A *lab config* is a YAML file that describes a group of [base configs](../new/base-configs) (e.g. 2 hospitals + 3 power plants).

A minimal example looks like this:

```yml
systems:
  foo:
    base_configs:
      - file: email
      - file: hospital
```

This deploys the machines in the `email.yml` and `hospital.yml` base configs. All machines will be in the `foo.lab` subnet (e.g. `email_server.foo.lab`).

## Replicas

Replicas are possible at the base config and subnet levels.

```yml
systems:
  foo:
    replica: 3  # subnet replicas
    base_configs:
      - file: email
        replica: 2  # base config replicas
```

This creates:
  - 3 replicas of the subnet (`foo.lab`, `foo1.lab`, `foo2.lab`)
  - 2 base config replicas, per subnet (`email_server.foo.lab`, `email1_server.foo.lab`, etc.)

## Config overrides

If needed, you can override values for base config fields:

```yml
systems:
  foo:
    base_configs:
      - file: email
        config:
          environment_vars:
            something: true
          instances:
            new_machine:
              size: 60
              instance_type: t3.small
              ami: ami-096fd57f2d61eec65
              OS: Amazon2
              instance_role: server
```

Values are overridden following these rules:
  - Primitive types (e.g. numbers, strings) take new values over old ones
  - Lists are concatenated (duplicate values can exist)
  - Existing dictionary keys take new values over old ones (where possible)
  - New dictionary keys are inserted with new values
