---
title: Mastodon Users
---
# Overview

This is a patch that is automatically included with `mastodon`.

It automatically creates a number of mastodon users.

## Usage

To automatically create mastodon users, you will need to put the `mastodon` system into a lab configuration.

```yml
labs:
  mastodon:
    systems:
      - name: mastodon
        config:
          environment_vars: 
            users:
              group_1: 20
              group_2: 20
```

Currently they are split into groups and these credentials are put straight onto the main jumpbox. It is advised that you modify the current playbook to redirect the output to the ansible machine or add an additional output to the ansible machine.

This will create 20 mastodon users under group 1, and 20 mastodon users under group 2, with random passwords.
