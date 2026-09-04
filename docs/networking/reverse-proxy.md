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

The home server sits behind a NAT. Public-facing services need a reverse proxy with a public IP. The VPS provides that endpoint.

Caddy was chosen over Nginx or Traefik because:

- Automatic HTTPS with zero configuration — Caddy handles certificate renewal natively
- Simple file-based config — no dynamic service discovery needed for 6 services
- Single binary, no daemon dependencies

The mesh VPN means the proxy connects to services over encrypted tunnels, not the public internet. Traffic flow: public internet → Caddy (VPS) → Nebula tunnel → service (home server).

!!! note "Why not Cloudflare Tunnel?"
    Cloudflare Tunnel would eliminate the need for a VPS entirely. But it routes traffic through Cloudflare's infrastructure, which adds latency and means trusting a third party with all request headers. The VPS + Caddy approach keeps traffic on infrastructure I control.
