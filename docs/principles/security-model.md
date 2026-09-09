---
title: Security Model
---

# Security Model

The security model is built around three pillars: rootless execution, per-service isolation, and defense in depth. The goal is to minimize the blast radius of any single compromise.

## Rootless Execution

Containers run rootless. There is no root daemon supervising them. An attacker who escapes a container lands in an unprivileged user namespace rather than holding root on the host.

For my threat model, this trade-off is worth it: the added complexity of managing rootless containers is something I'm happy to absorb in exchange for a significantly smaller attack surface.

## Per-Service Isolation

Services run as dedicated system users with `nologin` shells. There is no shared service account. A container escape gives the attacker a single UID with no shell, no home directory content, and no access to other services' secrets.

This makes the blast radius per-service rather than per-host. See [ADR-0001](../decisions/0001-rootless-per-service-users.md) for the full reasoning behind this model.

## Defense in Depth

Beyond user-level isolation, every container is hardened with multiple layers of protection to ensure that even if a process is compromised, the attacker's movement is restricted.

### Standard Hardening Pattern

To maintain a consistent security posture, all new services should implement the following hardening pattern in their Quadlet definitions:

| Goal | Quadlet / Podman Flag | Description |
| :--- | :--- | :--- |
| **Prevent Privilege Escalation** | `NoNewPrivileges=true` | Prevents processes from gaining new privileges via `setuid` or `setgid` bits. |
| **Minimize Kernel Surface** | `DropCapability=ALL` | Removes all default Linux capabilities; add back only what is strictly necessary. |
| **Protect Host Filesystem** | `ReadOnly=true` | Mounts the container root filesystem as read-only to prevent persistent malware. |
| **Mitigate DoS Attacks** | `MemoryMax=...` | Sets a hard memory limit to prevent a single service from exhausting host resources. |

!!! tip "Complete reference"
    For a complete, exhaustive list of all available security options, refer to the [official Podman documentation](https://docs.podman.io/en/latest/markdown/podman-run.html#security-options).

## Deliberate Exceptions

While the security model is strict, certain patterns require exceptions to balance security with operational necessity.

The [AI Services Pod](../workloads/ai-stack/the-ai-services-pod.md) is a deliberate exception where a family of services that already trust each other share a single user and pod. However, the principles of Defense in Depth remain in place via the container-level hardening described above.
