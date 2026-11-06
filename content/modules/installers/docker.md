---
title: Docker
weight: 1
---

# Docker Installer

This installer adds Docker to your system and allows you to use `docker` commands on your system machines.

To do so, add the following snippet to your `manifest.yml` file under the `dependencies` field.

```yml
dependencies:
  installer:
    docker: latest
```

For more information on Docker, you can read the documentation [here][1].

[1][https://docs.docker.com/]
