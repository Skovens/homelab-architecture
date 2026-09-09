---
title: Per-Service Users
---

# Per-Service Users

## What

Every containerized service runs under its own dedicated system user with a `nologin` shell. There is no shared service account.

## How

Each service user is created as a normal user (`system: no`) with `shell: /usr/sbin/nologin`:

```yaml
- name: Create service user
  user:
    name: myservice
    system: no
    shell: /usr/sbin/nologin
    create_home: yes
    home: /home/myservice
```

Linger is enabled for each user so their systemd session persists without a login:

```bash
loginctl enable-linger myservice
```

Subuid/subgid ranges are auto-allocated by the system — never manually set with `usermod`.

The per-service model:

| User | Services |
|------|----------|
| `ai-services` | llamacpp, open-webui, docling-serve, searxng, oikb, MCP servers (shared pod) |
| `healthchecks` | healthchecks.io |
| `forgejo` | Forgejo git forge |
| `jellyfin` | Jellyfin media server |
| `homeassistant` | Home Assistant |

## Why

I use per-service users to limit blast radius: if one container is compromised, the attacker only has that user's UID. The security model (UID isolation, separate secret stores, separate audit trails) is covered in the [Security Model](../../principles/security-model.md). This approach was formalized in [ADR-0001](../../decisions/0001-rootless-per-service-users.md).

!!! note "Why `system: no`?"
    Normal users get subuid/subgid auto-allocated. System users do not. Using `system: yes` would require manual `usermod` calls to set up subordinate UID/GID ranges, which is fragile and non-idempotent.
