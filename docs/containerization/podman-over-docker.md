---
title: Podman over Docker
---

# Podman over Docker

## What

I use Podman instead of Docker for all container workloads. Rootless. No daemon. No root.

## How

Podman runs every container as an unprivileged user process. There is no `dockerd` daemon running as root. Containers are managed by systemd through quadlet files, not through a long-running daemon process.

Key configuration choices:

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

The `unprivileged_userns_clone` sysctl is set to `0` — user namespaces are not cloned, maintaining strict UID isolation between host and container.

## Why

Docker's daemon runs as root. A container escape in Docker means full root access to the host. With Podman rootless, a container escape gives the attacker an unprivileged user namespace with no host access.

Beyond security, Podman integrates natively with systemd. There is no separate orchestration layer needed — systemd manages the lifecycle, and quadlet files declare the container configuration. Docker requires Docker Compose or a separate init system to achieve the same thing.

Docker also requires a persistent daemon. If `dockerd` crashes, all containers are orphaned. Podman containers are individual processes managed by systemd — if one fails, systemd restarts it independently.

!!! note "Why not Docker Desktop?"
    Docker Desktop is irrelevant for server workloads. The comparison is between `dockerd` (Docker Engine) and Podman. On a headless server, Docker Desktop doesn't exist — the question is whether to run the Docker daemon or use Podman's daemonless model.
