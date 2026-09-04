---
title: Quadlets
---

# Quadlets

## What

I use Podman quadlets instead of Docker Compose files. Quadlets are declarative container definitions that integrate directly with systemd.

## How

A quadlet file is a systemd unit file with container-specific sections. It lives in `~/.config/containers/systemd/` and systemd discovers it automatically — no `systemctl enable` needed.

```ini title="myapp.container"
[Container]
Image=docker.io/library/nginx:latest
PublishPort=8080:80
Volume=myapp-data.volume:/usr/share/nginx/html:ro
Network=myapp.network
DropCapability=ALL
NoNewPrivileges=true
ReadOnly=true
UserNS=keep-id

[Service]
Restart=always
MemoryMax=512M

[Install]
WantedBy=default.target
```

```ini title="myapp.volume"
[Volume]
VolumePath=/home/myapp/volumes/data

[Install]
WantedBy=default.target
```

```ini title="myapp.network"
[Network]
NetworkName=myapp
Driver=bridge

[Install]
WantedBy=default.target
```

Quadlet files generate corresponding systemd units:

| Quadlet File | Generated Unit |
|-------------|---------------|
| `myapp.container` | `myapp.service` |
| `myapp.pod` | `myapp.service` |
| `myapp.volume` | `myapp.service` |

!!! warning "Cross-unit references require file extensions"
    When referencing another quadlet unit, include the extension: `Pod=ai-services.pod` (correct), not `Pod=ai-services` (causes generator failure).

## Why

Docker Compose is a separate orchestration system. It runs its own process, has its own state, and doesn't integrate with systemd. When a Compose-managed container crashes, Docker's restart policy handles it — but systemd doesn't know about it.

Quadlets make systemd the single orchestrator. There is no separate `docker-compose up` process. systemd starts, stops, restarts, and monitors every container. `journalctl` captures container logs. `systemctl status` shows container state. Everything is in one place.

The tradeoff is that quadlets are less widely documented than Docker Compose. But the integration with systemd is worth it — especially when managing 15+ services across multiple users.

!!! tip "No `systemctl enable` needed"
    The Podman systemd generator discovers quadlet files automatically. Drop a `.container` file in `~/.config/containers/systemd/` and systemd picks it up on the next daemon-reload.
