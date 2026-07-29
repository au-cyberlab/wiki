---
title: Playbook dependencies
weight: 2
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

// TODO!!
- Referred to by Deployment > Existing systems > Deploy options > General options
- Take info from Deployment > Describing your deployment (OLD WIKI) > 2. Playbooks
