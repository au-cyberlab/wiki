---
title: Lab configs
weight: 1
---

# Lab configs

> [!CAUTION]
> This page does not yet reflect some recent major CyberLab changes. We are working to fix this.

A *lab config* is a YAML file that describes a group of [base configs](../new/base-configs) (e.g. 2 hospitals + 3 power plants). Lab configs reside in `aws_infrastructure/infrastructure_configs/`.

A minimal example looks like this:

```yml
systems:
  foo:
    base_configs:
      - file: email
      - file: hospital
```

This deploys the machines in the `email.yml` and `hospital.yml` base configs. All machines will be in the `foo.lab` subnet (e.g. `email-server.foo.lab`).

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
  - 2 base config replicas, per subnet (`email-server.foo.lab`, `email1-server.foo.lab`, etc.)

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
              ami: ami-096fd57f2d61eec65
              OS: Amazon2
              instance_role: server
```

Values are overridden following these rules:
  - Primitive types (e.g. numbers, strings) take new values over old ones
  - Lists are concatenated (duplicate values can exist)
  - Existing dictionary keys take new values over old ones (where possible)
  - New dictionary keys are inserted with new values

## Dependency config overrides

> [!CAUTION]
> The information below about overriding values of base config dependencies is a logical conclusion from other documentation. [However, it does not currently work.](https://github.com/UAdelaide/CyberLab/issues/474)

[Base configs may have dependencies](../../new/system-dependencies). Overriding values of base config dependencies is also possible. Take, for example, a base config with the following `dependencies` block:

```yml
dependencies:
  system:
    local:
      - traffic
    ext:
      - minibus
```

For such a base config, the lab config below would set the root disks of the traffic system server to 30 GiB, and that of the minibus booking system server to 40 GiB. Suppose the base config name is `NAME.yml`.

```yml
systems:
  foo:
    base_configs:
      - file: NAME
      - file: traffic
        config:
          server:
            size: 30
  bar:
    base_configs:
      - file: minibus
        config:
          server:
            size: 40
```

Note the distinction between local and external base config dependencies. Each must be placed under the correct subnet block.
