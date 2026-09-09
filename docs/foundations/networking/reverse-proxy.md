---
title: Reverse Proxy
---

# Reverse Proxy

## What

A dedicated Debian 13 VPS runs Caddy as a reverse proxy. It terminates TLS and routes traffic to services on the home server via the mesh VPN.

## How

Caddy handles automatic HTTPS via Let's Encrypt (ACME). Routes point to mesh addresses:

```caddyfile
jellyfin.example.com {
    reverse_proxy 10.0.0.4:8096
}

homeassistant.example.com {
    reverse_proxy 10.0.0.4:8123
}

ai.example.com {
    reverse_proxy 10.0.0.4:3000
}

git.example.com {
    reverse_proxy 10.0.0.4:3001
}
```

The proxy host has its own Ansible playbook and bootstrap process: create an `ansible` user, disable root SSH, then run everything as `ansible`.

## Why

The home server sits behind a NAT. Public-facing services need a reverse proxy with a public IP. The VPS provides that endpoint. Keeping this on self-managed infrastructure aligns with the [security model](../../principles/security-model.md) — traffic stays on infrastructure I control, routed over encrypted mesh tunnels.

Caddy was chosen because it provides automatic HTTPS with zero configuration, uses simple file-based config, and runs as a single binary with no daemon dependencies.
