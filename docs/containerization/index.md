---
title: Containerization
---

# Containerization

Every service in this homelab runs in a container. Not because containers are trendy — because they provide isolation, reproducibility, and declarative deployment.

The containerization stack is built on four pillars:

- [Podman over Docker](podman-over-docker.md) — rootless, daemonless, secure by default
- [Quadlets](quadlets.md) — systemd is the orchestrator, not Docker Compose
- [Per-Service Users](per-service-users.md) — nologin shells, UID isolation, separate secret stores
- [The AI Services Pod](the-ai-services-pod.md) — the one sanctioned exception to per-service isolation
- [Egress Gate](egress-gate.md) — fixing Podman's pasta boot-race networking problem
