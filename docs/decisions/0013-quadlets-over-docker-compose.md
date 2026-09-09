---
title: "ADR-0013: Quadlets over Docker Compose"
---

# ADR-0013: Quadlets over Docker Compose

## Status

Accepted

## Context

With Podman chosen as the runtime (ADR-0012), the question became how to declare and manage container lifecycles. The homelab prefers tools that integrate natively with systemd and avoid a separate long-running process.

Two options were evaluated:

- **Docker Compose** — a separate orchestration layer with its own daemon process and state. It does not integrate with systemd; when a Compose-managed container crashes, Docker's restart policy handles it but systemd is unaware.
- **Quadlets** — declarative container definitions that generate systemd units. systemd becomes the single orchestrator: it starts, stops, restarts, and monitors every container, captures logs via `journalctl`, and reports state via `systemctl`.

The homelab already runs systemd everywhere, so making it the orchestrator avoided adding a second process and a second place to check state.

## Decision

Use Podman quadlets to declare container workloads. Each service is one or more `.container`, `.pod`, `.volume`, or `.network` files in `~/.config/containers/systemd/`, which systemd discovers and manages as native units.

## Consequences

- **Positive:** One orchestrator (systemd) for containers and host services alike. Unified logging, state, and restart policies. No separate Compose process to run or keep healthy.
- **Negative:** Quadlets are less widely documented than Compose, so there is a steeper learning curve. Cross-unit references must include file extensions (e.g. `Pod=ai-services.pod`), and the declarative model is less flexible than an imperative compose script.

## Related

- ADR-0012 (Podman for rootless containers)
- ADR-0002 (user-scoped systemd via `systemctl --machine=`)
