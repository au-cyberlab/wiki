---
title: Systems
weight: 2
bookCollapseSection: true
---

# Overview

System modules contain the core playbooks and code that are required for setting up systems.

## Specification

A system module has the following structure and manifest:

```md
module
├── playbooks
│   └── deploy.yml
├── src/
├── manifest.yml
├── machines.yml
└── values.yml
```

```yml
version: 1.0
kind: system
dependencies:
  # any dependencies can go here
```

Many system modules include the use of the `installer` dependency `github`, because by default, your system machines do not have access to the repository. With the `github` installer dependency, it handles the overhead for you and puts your code directly onto the machine. See [here](../installers/github) for more.

`values.yml` is for environment variables that the module may need, or optional installer values that you may want to set defaults for.

Specifying the following would give any modules that include this module the variable `foo` with value `bar`.

You can also specify `host_vars` for each machine, which you would reference by the expanded name e.g. `worker_1` instead of `worker`.

Finally, you can fill the `ports` with the ports and protocols that your system uses. 

```yaml
environment_vars:
  foo: bar

host_vars:
  worker_1:
    hello: world

ports:
  web:
    port: 8080
    service: http
    protocol: tcp
    machine: master
```

`machines.yml` is how you specify your machines, and at first, it can be difficult.
The general structure for defining a machine is as follows:

```yml
master: # machine
  name: master # name of machine type
  size: 60 # size of storage
  count: 1 # number of machines of this type
  role: master # role of machine, defaults to name of machine
  os:
    image: ubuntu-22.04 # what operating system to use
  type:
    aws: t3.xlarge # what instance type to use
```

Most of the fields are self explanatory, and you can have one or more of these blocks in your `machines.yml`.

For a non-exhaustive list of aws machine types, head to the [AWS](../../backends/aws) page.

For an exhaustive list of image types, head to the [images](../../deployment/new/images) page. 

## Existing Systems

We have a large number of systems in the CyberLab. You can find individual pages for each of them on the left.
