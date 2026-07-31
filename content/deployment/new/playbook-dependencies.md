---
title: Playbook dependencies
weight: 4
---

# Playbook dependencies

Commonly used playbook tasks (e.g. to set up a Kubernetes cluster) are isolated into dedicated playbooks. These playbooks can be specified as dependencies for a specific system. This is done in a [base config](../base-configs):

```yml
dependencies:
  setup:
    - kubernetes
```

Playbook dependency names are mapped to their corresponding playbooks in `ansible/playbooks/dependencies.yml`, like so:

```yml
kubernetes: kubernetes/deploy_clusters.yml
```

Paths are relative to `ansible/playbooks/`.

## Dependency waiting

If you specify playbook dependencies, your system's playbook will likely need to wait for them to finish executing. Assuming your playbook dependencies are called `foo` and `bar`, this Ansible block will wait for both to finish executing before running further tasks:

```yml
- name: Wait for dependencies
  hosts: ansible_machine
  connection: local
  tasks:
    - name: Wait for dependencies
      ansible.builtin.include_tasks:
        file: ../dependency_wait.yml
      loop:
        - ansible/done/foo.done
        - ansible/done/bar.done
      vars:         # optional block
        time: 1200  # default timeout between retries, in seconds
```

**Note:** 1200 seconds is 20 minutes.

Usually this block appears near the top of a playbook, just before software setup tasks begin. `vars` is usually omitted, unless overriding values.
