---
weight: 1
---

# Quickstart

## 1. Deploying a cyber range

Choose a [branch][1] to deploy. Each branch represents a description of a cyber range. If running for the first time, start with [`example`][2].

Open the [CyberLab repository][3] and navigate to **Actions > Deploy dev
AWS environment**. On the right, there should be a **Run workflow** drop-down. Click
on it, and should you see a branch selection drop-down. Select the branch you
wish to deploy (it must NOT be `main`), then press the green **Run workflow**
button:

![CyberLab deploy workflow](/images/deploy-branch.png)

Within 8-12 minutes, the deploy workflow should complete. **Do NOT cancel a running deployment.**

**Note:** The workflow completes after the *machines and network* are set up. This does not mean all the software has been deployed to the machines. Software is deployed with Ansible; you can check if deployment is complete by [checking the Ansible logs](#23-checking-deployment-status) (section 2.3).

## 2. Accessing your range

### 2.1 Logging in

Click on the workflow run you started.

![CyberLab deploy workflow](/images/workflow-done.png)

At the bottom of the page are the **Cluster Artifacts**. Download them and unzip them. You should find the following files, where BRANCH is the name of the branch you deployed:

```
artifacts/
  ├── cyberlab-BRANCH-ansible-key.pem
  ├── cyberlab-BRANCH-ansible-key-public.pem
  ├── cyberlab-BRANCH-key.pem
  ├── cyberlab-BRANCH-key-public.pem
  ├── cyberlab-jumpbox-ssh-key.pem
  ├── cyberlab-jumpbox-ssh-key-public.pem
  ├── .gitkeep
  ├── NetworkMap.png
  └── sops-age-secret.yaml
```

In the center of the workflow page, there is an **SSH command**:
  1. Copy the SSH command
  2. `cd` into your unzipped artifacts folder
  3. Paste and run the SSH command

You should now be logged in!

**Note:** Both the SSH command and the artifacts folder is different for each deployment.

### 2.2 Accessing hosts

The initial SSH command logs you into the local jumpbox. To access other hosts, run `ssh HOST` where HOST is either the IP address or domain name of the instance (e.g. `ssh ansible.lab`).

For a list of hosts, run `cat hosts.txt` on the local jumpbox. (**Note:** hosts.txt is added after DNS is deployed)

Alternatively, your unzipped artifacts folder contains your cyber range's network map (`NetworkMap.png`).

### 2.3 Checking deployment status

Software may take some time to deploy *after* the workflow finishes. Monitor the progress on the Ansible machine:

```
ssh ansible.lab
cd ansible/logs/
```

Each system deployment has its own log file. When deployment finishes, there should be a PLAY RECAP section at the end. Success occurs when `failed=0` for all machines.

## 3. Destroying your range

> [!CAUTION]
> **Keeping machines on costs money. Please ensure to destroy your cyber range as soon as you're done with it.**

To destroy the cyber range, run **Actions > Destroy dev AWS environment** on your branch.


[1]: https://github.com/UAdelaide/CyberLab/branches
[2]: https://github.com/UAdelaide/CyberLab/tree/example
[3]: https://github.com/UAdelaide/CyberLab
