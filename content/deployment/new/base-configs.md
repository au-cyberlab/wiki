---
title: Base configs
weight: 1
---

# Base configs

A *base config* is a YAML file describing a set of machines for a particular system (e.g. a Mastodon service). Base configs reside in `aws_infrastructure/infrastructure_configs/base_configs/`.

Confusingly, base configs are used for more than just describing machines. Hence their unintuitive name.

## 1. Machines

Available options when describing a set of machines, as well as their functions, are described below.

```yml
instances:
  NAME:                         # machine name
    size: 10                    # root disk size, in GiB
    type:                       # backend-specific machine type
      aws: t3.medium            # AWS machine type, class, or category
    specs:
      cpu: 2
      ram: 4                    # in GiB
      clock_speed: 0            # in GHz
      gpu_type: ""              # GPU model
      gpu_count: 0
      vram: 0                   # in GiB
    ami: ami-096fd57f2d61eec65  # AWS OS image ID
    OS: Amazon2                 # OS distribution; used to run OS-specific setup commands
    instance_role: Undefined    # used to group machines for Ansible targeting
    vars:                       # non-standard way of defining host variables
      something: foo
```

> [!NOTE]
> If specifying either `ami` or `OS`, you MUST specify both. [This issue is being rectified.](https://github.com/UAdelaide/CyberLab/pull/453)

The term "instances" to describe machines comes from AWS "EC2 instances". It is a regrettable mistake.

A list of supported GPU models is available [here](https://docs.aws.amazon.com/ec2/latest/instancetypes/ac.html#ac_hardware) for AWS.

Host variables (including the standard way of defining them) are described [later](#2-host-variables).

### 1.1 Machine names

Assuming NAME is the machine name (e.g. `server`), SYSTEM is the name of the system (e.g. `email`), and SUBNET is the name of the subnet (e.g. `foo`), then:

  - The machine will be SUBNET_SYSTEM-NAME in the Ansible inventory (e.g. `foo_email-server`)
  - The machine will be SYSTEM-NAME.SUBNET.lab in DNS (e.g. `email-server.foo.lab`)

SYSTEM is extracted from the base config filename (assuming SYSTEM.yml).

SUBNET is the same as SYSTEM unless a different name is given via a [lab config](../../existing/lab-configs).

### 1.2 Machine type and specs

`specs` defines required machine specifications in a backend-agnostic fashion. Backend-specific machine types (e.g. `t3.medium` on AWS) can be specified using `type`. Either `specs` or `type` may be specified, or both.

Machine selection tries to honour `type` first. If `type` is a specific machine type, `specs` is ignored. If `type` is a machine class or category, machine selection searches for a machine matching `specs` from that class or category. However, if no machine matches `specs` from that class or category, all machines will be evaluated and the closest match will be selected.

Machine types, classes, and categories are explained in more detail under [AWS](../../../backends/aws).

### 1.3 OS images

A list of supported OS images is available [here](../../../backends/aws).

### 1.4 Machine roles

`instance_role` is in a transitional state. It used to dictate which "instance scripts" would run. This is being phased out in favour of Ansible setup tasks.

`instance_role` may be one of the following:
  - `Ansible` - runs `AnsibleScripts` instance script
  - `BBRouter` - *(backbone router)* runs `routerScripts` instance script
  - `EdgeRouter` - runs `routerScripts` instance script
  - `KafkaHandler` - runs Kafka handler setup tasks
  - `Jumpbox` - runs `EnvJumpboxScripts` instance script
  - `MainJumpbox` - runs `MainJumpboxScripts` instance script
  - `Master` - runs `kubernetesMasterScripts` instance script and Kubernetes master node setup tasks
  - `Worker` - runs `kubernetesWorkerScripts` instance script and Kubernetes worker node setup tasks

Machines with other `instance_role`s will run the `BasicConfigurationScripts` instance script, and may be recognised and targeted by playbooks. For example, `server` is a value commonly recognised by system-specific playbooks. Unrecognised values result in no role-specific setup tasks being run.

## 2. Host variables

// TODO!!
