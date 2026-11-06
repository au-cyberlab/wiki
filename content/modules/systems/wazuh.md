# Overview

The Security Operations Center (SOC) environment has two primary components.
First, the monitoring agents which run alongside other environments and report
incidents. Second, the SOC itself, with a dashboard to analyse reported
incidents.

This implies a SIEM platform, and indeed the SOC environment is a combination of
the Wazuh SIEM and agent systems.

## 1. Setup

To deploy the email system, add `wazuh` to a lab configuration that is being deployed or your `deployment_config.yml` file.

## 2. Configuration

Inside `values.yml`, there are several environment variables.
```yaml
environment_vars:
  spec:
    worker: 
      replicas: 4
      cpu: 1
      memory: 4Gi
    master:
      replicas: 1
      cpu: 3
      memory: 12Gi
```
`worker.replicas` specifies the number of workers to use inside kubernetes. This is separate from the number of worker instances present in AWS. The more you have, the more effectively the wazuh siem will work with larger systems. You may need to increase the specs of the machines used.

`worker.cpu` and `worker.memory` specify the amount of resources to allocate to each of these workers.

`master.replicas` specifies the number of masters to use inside kubernetes. Typically you should just leave this value set to `1` but in some cases you may need to increase it.

`master.cpu` and `master.memory` specify the amount of resources to allocate to the master node. You may need to scale this with how large the environment is but it is much more feasible to scale the number of workers.

You can overwrite these in a lab configuration. Do **NOT** overwrite them directly in the modules, unless it directly contributes to the functionality of the system across all deployments.

## 3. Usage

The wazuh dashboard is exposed on port `30000`.

```bash

# for AWS
ssh -L 8000:<WAZUH_MACHINE_IP>:30000 -i cyberlab-jumpbox-ssh-key.pem <CYBERLAB_USER>@<JUMPBOX_IP>
```

You can also access this through the CyberLab VPN.

You should now be able to access the wazuh dashboard at `https://localhost:8000`.

If you can't, first, ensure that the protocol used is HTTPS and NOT HTTP.
Your browser may warn you about certificates - you can safely ignore this and proceed.

The credentials are:
username: `admin`
password: `SecretPassword`

You should now be logged in:

<!--![Wazuh dashboard](images/wazuh.png)-->

## 4. Contributors

Saurab Kharel ([@Saurabkharel1][3]) and Anton Ivanov ([@kv-o][4]) added the SOC
system to the CyberLab.


[1]: https://ronin.uoa.cloud
[2]: https://github.com/UAdelaide/Cyberlab/wiki/Kubernetes
[3]: https://github.com/Saurabkharel1
[4]: https://github.com/kv-o
