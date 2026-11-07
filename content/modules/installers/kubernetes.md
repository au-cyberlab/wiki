---
title: Kubernetes
weight: 3
---

# Kubernetes Installer

This installer sets up the master-worker relationship between machines that are designed to run a Kubernetes cluster.

To include it, add the following snippet to your `manifest.yml` file under the `dependencies` field.

```yml
dependencies:
  installer:
    kubernetes: latest
```

You will also need to apply some modifications to your `machines.yml` file, wherever that may be for your `system` module.
Exactly one machine with the role `master` is required, and any number of machines with the role `worker` must be present for this installer to work properly.

For instance, in `hospital/machines.yml`, the following needs to be specified.
```yml
master:
  name: master # name of instance type
  size: 60
  role: master # default - name of machine
  os:
    image: amazon-2
  type:
    aws: t3.xlarge
worker:
  name: worker # name of instance type, goes <name>_1, <name>_2, if more than one
  size: 60
  count: 2 # optional, default 1
  role: worker
  os:
    image: amazon-2
  type:
    aws: t3.xlarge # amazon instance type, saves the lookup (optional)
```

Doing this will link the master machine and the two worker machines in a kubernetes cluster.

To assist with the management of these systems, `k9s` is automatically installed by default, which is a tool that is used to manage kubernetes clusters easier.

You can read more about Kubernetes [here][1].

You can also read more about K9S [here][2].

[1]: https://kubernetes.io/docs/home/
[2]: https://k9scli.io/
