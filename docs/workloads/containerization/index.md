---
title: Containerization
---

# Containerization

Every service in this homelab runs in a container. They provide isolation,
reproducibility, and declarative deployment — which makes them a natural fit
for this project. The design choices here are grounded in the
[Security Model](../../principles/security-model.md) and
[Orchestration Model](../../principles/orchestration-model.md) principles.

The containerization stack is built on four pillars:

- [Podman over Docker](podman-over-docker.md) — rootless, daemonless, secure by default
- [Quadlets](quadlets.md) — systemd as the orchestrator
- [Per-Service Users](per-service-users.md) — nologin shells, UID isolation, separate secret stores
- [The AI Services Pod](../ai-stack/the-ai-services-pod.md) — a deliberate exception to per-service isolation
