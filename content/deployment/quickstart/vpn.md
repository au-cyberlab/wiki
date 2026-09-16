---
title: Access via VPN
weight: 1
---

# Access via VPN

[Quickstart](../quickstart/) described CyberLab access via SSH. However, when interacting with *websites* in the CyberLab, using SSH is inconvenient.

An alternative way to access the CyberLab is via a [Wireguard](https://en.wikipedia.org/wiki/WireGuard) [VPN](https://en.wikipedia.org/wiki/Virtual_private_network).

## 1. Command-line setup

*(These instructions assume a Linux environment)*

### Connecting to VPN

1. Install Wireguard ([see instructions](https://www.wireguard.com/install/))
2. Download and unzip the Cluster Artifacts (as in [Quickstart](../quickstart/))
3. In the unzipped artifacts, you will find `vpn/cyberlab-<user>.conf` where `<user>` is your GitHub username.
4. Copy that config file to `/etc/wireguard/<interface>.conf`
     - `<interface>` is the name of the virtual Wireguard interface you will activate.
     - Conventionally named `wg0`.
     - If you already have a `wg0` interface configured, you can use `wg1`, and so on.
5. Run `wg-quick up <interface>` to activate your Wireguard interface.

### Disconnecting from VPN

1. Run `wg-quick down <interface>` to deactivate your Wireguard interface.

## 2. Accessing hosts

When connected via VPN, you can access CyberLab machines directly by their IP address or DNS domain name.

### SSH via VPN

When connected via VPN, you can still SSH into your subnet with the same SSH command (as in [Quickstart](../quickstart/)).

You can also SSH using internal CyberLab IP addresses (or domain names), but you need to use the branch-specific SSH key, and specify the default OS user. Example:

```
ssh -i cyberlab-<branch>-key.pem ubuntu@10.0.6.244
```
