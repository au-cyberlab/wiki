---
title: Ansible playbooks
weight: 3
---

# Ansible playbooks

> [!CAUTION]
> This page does not yet reflect some recent major CyberLab changes. We are working to fix this.

Ansible playbooks deploy software onto CyberLab machines. They reside in `ansible/playbooks/`.

## Context

Ansible is a popular tool for pushing software and configuration to many machines at once. [Any Ansible setup](https://docs.ansible.com/projects/ansible/latest/getting_started/basic_concepts.html) consists of an:
  - *inventory*, which describes target machine IP addresses, groupings, etc.; and
  - *playbooks*, which describe tasks to run on target machines.

In the CyberLab, the Ansible inventory is auto-generated from the [base configs](../base-configs) specified in [`aws_infrastructure/deployment_config.yml`](../../deployment/existing).

// TODO!!
