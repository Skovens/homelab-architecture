---
title: Security Model
---

# Security Model

The security model is built around three ideas: rootless execution, per-service
isolation, and defense in depth. The goal is to keep the blast radius of any one
compromise as small as possible.

## Rootless Execution

Containers run rootless. There is no root daemon supervising them. An attacker
who escapes a container lands in an unprivileged user namespace rather than
holding root on the host.

For my threat model this trade-off is worth it: the added complexity of managing
rootless containers is something I'm happy to absorb in exchange for a much
smaller attack surface.

## Per-Service Isolation

Services run as dedicated system users with `nologin` shells. There is no shared
service account. A container escape gives the attacker a single UID with no
shell, no home directory content, and no access to other services' secrets.

This makes the blast radius per-service rather than per-host. See
[ADR-0001](../decisions/0001-rootless-per-service-users.md) for the full
reasoning behind this model.

## Deliberate Exceptions

The security model is not absolute. The
[AI Services Pod](../workloads/ai-stack/the-ai-services-pod.md) is a deliberate
exception where a family of services that already trust each other share a
single user and pod. The container-level hardening
(`DropCapability=ALL`, `NoNewPrivileges=true`, read-only rootfs, per-container
`MemoryMax`) remains in place.
