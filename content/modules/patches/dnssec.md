---
title: DNSSEC
---

# Overview

This is a patch that is automatically included with the DNS server.

It enables key generation and creates certificates for DNS record validation.

## Usage

To enable DNSSEC, specify the following in the `deployment_config.yml`

```yml
dns:
  deploy: true
  dnssec: true
```
