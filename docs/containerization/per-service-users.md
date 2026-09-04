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

A shared service user means one UID for all containers. A compromise of any container gives the attacker that UID for every other container — no filesystem boundary, no secret isolation.

Per-service users provide:

- **UID isolation** — container escape blast radius is limited to one user
- **Separate secret stores** — Podman secrets are scoped per user
- **Separate audit trails** — `journalctl --user --unit=myapp` shows only that service
- **File ownership boundaries** — each user owns only its own data

!!! warning "The nologin shell matters"
    `nologin` prevents interactive access to the service user. Even if an attacker compromises the container and somehow gets host access, they cannot log in as that user. This is a defense-in-depth layer — the primary isolation comes from user namespaces (`UserNS=keep-id`), but the nologin shell removes another attack surface.

!!! note "Why `system: no`?"
    Normal users get subuid/subgid auto-allocated. System users do not. Using `system: yes` would require manual `usermod` calls to set up subordinate UID/GID ranges, which is fragile and non-idempotent.
