---
title: Deploy options
weight: 3
---

# Deploy options

Aside from selecting systems and scenarios to deploy, `aws_infrastructure/deployment_config.yml` provides options to customise deployment.

> [!IMPORTANT]
> The `backend` field and `environments` block from [Existing systems](../existing) are both required.

## Backend

```yml
backend: aws
```

This field specifies the target backend. Currently, the value may only be `aws`.

**Note:** `backend` is a required field.

## AWS options

```yml
aws_vpc:
  name: vpc-1234567890
```

If `aws_vpc` does not exist or no `name` is given, the default CyberLab AWS VPC will be used.

> [!CAUTION]
> The `aws_vpc` block is for **CyberLab admins ONLY**.

## DNS options

```yml
dns:
  deploy: true   # whether to deploy a DNS server
  dnssec: false  # whether to configure DNSSEC
```

Available DNS options, their default values, and their functions are described above.

> [!CAUTION]
> **Bug:** The `dns` block should be optional, but it is not. See [this ticket](https://github.com/UAdelaide/CyberLab/issues/460).

## Training options

```yml
LS-Training:
  enable: false  # whether to enable training mode
  user: USER     # sets the SSH user to cyberlab-USER
```

Enabling training mode removes access to the Ansible machine. `user` must be set, and must be unique across branches.

Users get a `cyberlab-USER-ssh-key.pem` instead of `cyberlab-jumpbox-ssh-key.pem` in [Cluster Artifacts](../../quickstart). The key is unique to each deployment.

The Ansible machine is still accessible via the main jumpbox for CyberLab admins.

## General options

```yml
general:
  auto_include: true      # whether to automatically include missing dependencies
  no_missing_deps: false  # whether to abort deployment if missing dependencies
  log_level: info         # log verbosity
```

Available "general" options, their default values, and their functions are described above.

`no_missing_deps` ignores the value of `auto_include`. Dependencies are further described in [Dependency playbooks](../../new/dependencies).

Log levels control which logs to output:
  - `debug`/`all` - All logs
  - `info` - All logs except debug
  - `warn` - Warnings, errors, and fatal logs
  - `error` - Errors and fatal logs
  - `none` - No logs
