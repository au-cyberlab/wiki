---
title: Expo IT Configs
---

# Overview

This patch adds monitoring support for any system for Expo IT.

To include it, the following values are required for the system you want to monitor.

```yml
labs:
  web: 
    systems:
      - system: email
        patch: # include the expo it config patch
          expo_it_config: latest
        config:
          environment_vars:
            expo_config_profile: # the configurations what "what" we want to monitor
              - email
              - ssh
            expo_config_source_id: "cyberlab-email-system" # the source config
            expo_config_team_id: team1 
            expo_config_team_name: "Team 1"
```

First, you need to include the patch as detailed above.

Then, in `environment_vars`, provide a list of the profiles you would like to apply. You can find these profiles in the `expo_it_config` module under the `playbooks` directory, specified in the format `<profile_name>.yml`.

If it doesn't exist, then you will need to make one yourself.

## Creating Expo Configs

Each profile follows the same configuration steps:

1. Build your system block
2. Construct your services
3. Append both to the manifest

To construct your system block, you need to specify the following:

This example is taken from the battery system configuration.

```yml
- name: Construct the system block
  ansible.builtin.set_fact:
    expo_systems:
      - id: battery-system
        display_name: Battery System
        hostname: "{{ ansible_host }}"
        team: "{{ expo_config_team_id | default('bt01') }}"
        team_name: "{{ expo_config_team_name | default('Blue Team 01') }}"
```

This part is fairly self explanatory - we bind assign a system to a team and a hostname, and store the variable for later use.

Next, constructing services.

Services are stored in a list under `expo_services`, which we will later merge into a full manifest and push to expo all at once.

To do this, we can specify a list in the following format below:

```yml
- name: Construct the services block
  ansible.builtin.set_fact:
    expo_services:
      - id: battery-dashboard
        display_name: Battery System dashboard
        description: Battery System dashboard endpoint
        system_id: battery-system
        url: "http://{{ ansible_host }}:{{ battery_port | default(5000) }}/"
        scoring:
          components:
            - id: http-health
              criticality: 5
              check:
                type: http
                url: "http://{{ ansible_host }}:{{ battery_port | default(5000) }}/"
                content_regex: '<p class="title">Battery Monitor</p>'
            - id: tcp-port
              criticality: 5
              check: "{{ {'type': 'tcp', 'host': ansible_host, 'port': battery_port | default(5000) | int} }}"
```

We give the service a name, and bind it to its parent system id.
We also need to provide a url - but this field is not mandatory, just a useful fact for the system to display to each team.
The most important part are each of the components.

Each component has an id, a criticality, and a type of check.

IDs need to be unique, and criticality determines how many points are lost when the system is down, checked every 15s.

The `check` field determines the type of check to run.

Valid checks include:
 - `http` - checks for HTTP/HTTPS connectivity. Has an optional `content_regex` field that allows for a specific regex to be checked for on the page. Doesn't support dynamically loaded pages such as those built with React or Vue
 - `tcp` - checks that the port can be connected to.

 Both examples are included above, and all of their fields as well.

 Once the system block is created, we need to append both of these to the manifest.

 We can do so by using the following ansible snippet:

 ```yaml
 - name: Assemble the Battery System resource manifest from deployed addresses
   ansible.builtin.set_fact:
     expo_config_manifest:
        apiVersion: expo-it/v1
        source_id: "{{ expo_config_source_id }}"
        systems: "{{ expo_config_manifest.systems + expo_systems }}"
        services: "{{ expo_config_manifest.services + expo_services }}"
 ```
