---
title: Playbook batches
weight: 2
---

# Playbook batches

`ansible/playbooks/plays_to_run.yml` is structured around *batches* of playbooks:

```yaml
batches:
  batch1:
    - container_projects/deploy_email.yml
    - container_projects/deploy_kafka.yml
  batch2:
    - container_projects/deploy_hospital.yml
    - container_projects/deploy_mastodon.yml
    - container_projects/deploy_minibus.yml
```

Playbooks in the same batch will run concurrently, and playbooks in subsequent batches will not start until previous batches are done:

```mermaid
---
config:
  theme: 'base'
  themeVariables:
    primaryColor: '#707070'
    primaryTextColor: '#ffffff'
    primaryBorderColor: '#707070'
    lineColor: '#707070'
---
stateDiagram
    state batch1 <<fork>>
        [*] --> batch1
        batch1 --> email
        batch1 --> kafka

    state batch2 <<fork>>
        email --> batch2
        kafka --> batch2
        batch2 --> hospital
        batch2 --> mastodon
        batch2 --> minibus

        hospital --> [*]
        mastodon --> [*]
        minibus --> [*]
```

## Sequential execution

To execute playbooks in sequence, set `asynchronous: false`:

```yaml
asynchronous: false
batches:
  batch:
    - container_projects/deploy_email.yml
    - container_projects/deploy_hospital.yml
    - container_projects/deploy_mastodon.yml
```

Batches have no effect under sequential execution.
