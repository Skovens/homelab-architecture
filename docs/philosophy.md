---
title: Philosophy
---

# Philosophy

Three principles drive every decision in this homelab.

## Rootless-First

Containers run rootless. Services run as dedicated system users with `nologin` shells. There is no shared service account. A container escape gives the attacker a single UID with no shell, no home directory content, and no access to other services' secrets.

This is not the default for most homelab guides. Most use Docker with root, or a single `podman` user for everything. I chose the harder path because the security model is sound: blast radius is per-service, not per-host.

## Infrastructure as Code

Every service is deployed by Ansible. No manual `podman run`. No SSH-in-and-fix. If it's not in the repo, it doesn't exist on the server.

This means I can rebuild the entire homelab from scratch. It means changes are reviewed, versioned, and auditable. It means I can show you exactly how every service is configured — and you can reproduce it.

## intentional Simplicity

I use Podman instead of Docker. Quadlets instead of Compose. Sanoid instead of a backup script. Each choice favors the tool that does one thing well, integrates with the system natively, and doesn't require a daemon running as root.

I don't run Prometheus, Grafana, or Loki. I run healthchecks.io. Monitoring should tell me if something is down, not give me 400 dashboards I never look at.

I don't run a full Kubernetes cluster. I run systemd with quadlets. For 15 services, Kubernetes is overhead, not capability.

Every technology in this homelab earned its place. Nothing is here because it's popular — it's here because it solves a specific problem better than the alternative.
