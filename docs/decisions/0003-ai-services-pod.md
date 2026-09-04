---
title: "ADR-0003: AI Services Pod"
---

# ADR-0003: Shared `ai-services` pod for AI containers

## Status

Accepted

## Context

The AI service family (llamacpp, open-webui, docling-serve, searxng, searxng-valkey, oikb, github-mcp, forgejo-mcp, openhands) grew into a set of standalone containers. Each had its own user and pod, producing:

- Cross-container communication over the LAN/IP instead of localhost (open-webui calling llamacpp, docling-serve, searxng, MCP servers).
- Duplicate user management and lingering sessions per service.
- A duplicated pod template across 5 roles — any pod change had to be updated in 5 places.

The services share persistent data (llamacpp models dir, open-webui data) and are deployed as one logical unit. The tradeoff of a shared user (single UID across services) was accepted here because container-level hardening — not the host UID — is the primary isolation layer (ADR-0001), and these services already trust each other with shared volumes.

## Decision

Consolidate all AI containers into a single `ai-services` user and one shared `ai-services` pod:

- One user `ai-services` (nologin, per ADR-0001), one lingering session.
- One `ai-services.pod` quadlet pod template, used by all AI container roles.
- Containers join the pod via `Pod=ai-services.pod` and reach each other over localhost.
- Pod containers declare `After=`/`Wants=` on the pod unit (and on any containers they depend on at startup) to avoid race conditions — systemd starts all pod containers simultaneously without this.

## Consequences

- **Positive:** One user + one pod = simpler user management; localhost URLs instead of network hops; single pod template to maintain.
- **Negative:** All AI containers share one host UID — a compromise of any single container exposes that UID to the rest (mitigated by `DropCapability=ALL`, `NoNewPrivileges`, read-only rootfs, per-container `MemoryMax`).
- **Operational rule:** dependency ordering inside the pod is explicit systemd `After=`/`Wants=`, never `sleep` hacks.

## Related

- ADR-0001 (per-service users — this is the sanctioned shared-user exception)
- ADR-0002
