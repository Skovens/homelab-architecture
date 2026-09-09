---
title: "ADR-0012: Podman for rootless containers"
---

# ADR-0012: Podman for rootless containers

## Status

Accepted

## Context

The homelab runs 15+ containerized services on a single home server. The container runtime had to support a daemonless, rootless security model — a container escape should not grant host root, and there should be no long-running daemon that can orphan workloads if it crashes.

Two primary options were evaluated:

- **Docker Engine (`dockerd`)** — the standard container runtime. Runs a root daemon that supervises all containers. If the daemon crashes, containers are orphaned.
- **Podman** — a daemonless, rootless container runtime. Containers run as unprivileged user processes, each an independent process that can be managed directly by systemd.

The homelab already favored systemd-native tooling and per-service isolation, which made the daemonless, rootless model the natural fit.

## Decision

Use Podman for all container workloads. Containers run rootless, supervised by systemd via quadlets (see ADR-0013). There is no Docker daemon and no container running as root.

## Consequences

- **Positive:** A container escape lands in an unprivileged user namespace rather than granting host root. Each container is an independent systemd-managed process, so there is no single point of failure and no orphaned workloads if a manager crashes.
- **Negative:** Podman's rootless networking and storage configuration differs from Docker's defaults (pasta, netavark — see ADR-0011 for one networking pitfall). Some images and tooling expect a Docker daemon socket; the daemonless model occasionally requires a compat shim.

## Related

- ADR-0001 (rootless per-service `nologin` users)
- ADR-0011 (egress gate for rootless containers)
- ADR-0013 (quadlets over Docker Compose)
