---
title: "ADR-0001: Rootless per-service users"
---

# ADR-0001: Rootless per-service `nologin` users for Podman

## Status

Accepted

## Context

The home server runs 10+ containerized services (AI, media, automation, backup, monitoring). Containers are deployed as rootless Podman quadlets managed by systemd. Two viable alternatives to per-service users were evaluated:

- **Rootful Podman** with system-level quadlets in `/etc/containers/systemd/` — the Podman daemon runs as root, so a container escape means full host root. Volume ownership and network setup need root privileges.
- **Shared service user** (single `podman-services` UID for all containers) — a compromise of any container gives the attacker the same host UID for all other containers, no filesystem boundary, secrets visible across services.

The per-service model was already in production; the question was whether the marginal security benefit justified the operational complexity.

## Decision

Keep rootless per-service users, each with a `nologin` shell (`system: no` normal users), one per service or per service family:

| User | Services |
|------|----------|
| `ai-services` | llamacpp, open-webui, docling-serve, searxng, searxng-valkey, oikb, github-mcp, forgejo-mcp, openhands (shared pod) |
| `healthchecks` | healthchecks |
| `jellyfin` | jellyfin |
| `homeassistant` | homeassistant |
| `forgejo` | forgejo |
| `rclone-user-a` / `rclone-user-b` | rclone backups |

The primary isolation comes from **container-level controls** (user namespaces `UserNS=keep-id`, `DropCapability=ALL`, `NoNewPrivileges=true`, `ReadOnly=true`, seccomp), which work regardless of host user model. The per-service user adds a separate host UID for file ownership, a separate Podman secret store, and a separate audit trail.

Rejected options recorded for the record: rootful Podman (daemon-level trust boundary, accepted tradeoff documented in the analysis), shared service user (weakest isolation, not offset by operational gain).

## Consequences

- **Positive:** Container escape blast radius limited to a single UID; secrets isolated per user; containers stay unprivileged.
- **Negative:** User-scoped systemd units cannot be managed with the native `ansible.builtin.systemd` module (root cannot connect to another user's D-Bus socket) — see ADR-0002. User creation, `loginctl enable-linger`, and per-user systemd directories add operational overhead.

## Related

- ADR-0002 (`systemctl --machine=` management)
- ADR-0003 (shared `ai-services` pod)
