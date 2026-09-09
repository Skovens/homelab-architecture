---
title: Podman for Rootless Containers
---

# Podman for Rootless Containers

## What

For this homelab, I use Podman to manage container workloads. This approach prioritizes a rootless, daemonless architecture, which aligns with my goals for security and system integration.

## How

Podman runs containers as unprivileged user processes. Unlike the traditional Docker daemon model, there is no long-running `dockerd` process running with root privileges. Instead, container lifecycles are managed directly by `systemd` via Quadlets.

### Configuration

To ensure optimal performance and security, the following configurations are used:

```toml title="containers.conf"
[containers]
default_rootless_network_cmd = "pasta"

[engine]
events_logger = "journald"
cgroup_manager = "systemd"
```

```toml title="storage.conf"
[storage]
driver = "overlay"

[storage.options]
network_backend = "netavark"
```

## Why

I chose Podman for rootless, daemonless container management. The security rationale (reduced attack surface, unprivileged user namespaces) is covered in the [Security Model](../../principles/security-model.md); the systemd integration rationale is covered in the [Orchestration Model](../../principles/orchestration-model.md). The decision itself is recorded in [ADR-0012](../../decisions/0012-podman-for-rootless-containers.md).
