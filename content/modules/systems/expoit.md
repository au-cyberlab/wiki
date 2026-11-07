---
weight: 1
---

> [!NOTE]
> EXPO IT is not currently an open source system. As such, only limited documentation for this is available.

# Overview

EXPO IT is a (to-be) open source system that is designed to mimic the functionality of the dashboards used for Locked Shields. It keeps track of scores, and allows systems to be monitored with different types of health checks.

## Usage

To deploy the expo system, add `expo_it` to a lab configuration that is being deployed or your `deployment_config.yml` file.

Once deployed, it should be available on the port `8080`.

You should be able to see many dashboards, and the admin password is configured via environment variables in the field `expo_admin_password`
