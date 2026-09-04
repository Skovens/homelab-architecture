---
title: Mesh VPN
---

# Mesh VPN

## What

All hosts are connected via DNClient — a managed Nebula mesh VPN from Defined Networking. Every server, the DR host, and the reverse proxy join the same overlay network.

## How

DNClient is deployed on every host. The binary lives at `/usr/local/bin/dnclient` and is auto-installed from the Defined API. Each host gets a mesh address in the `10.0.0.x` range.

Services bind to their mesh address, not the public internet. The reverse proxy routes traffic from public DNS to mesh addresses. NFS is restricted to the mesh subnet:

```yaml
nfs_exports:
  - path: /bulkdata/Media
    clients: "10.0.0.0/24"
    options: "all_squash,anonuid=0,anongid=451"
```

## Why

Defined Networking is managed Nebula. The open-source Nebula project provides the overlay network — certificate-based identity, encrypted tunnels, and firewall policy. Defined adds a management plane on top: automatic cert rotation, NAT traversal (UDP hole punching + relays), and a central dashboard.

The key architectural insight: the management plane (Defined API) is separate from the control plane. The overlay keeps running even if the Defined API is unreachable. DNS resolution, firewall rules, and tunnel establishment all happen locally after initial enrollment.

!!! note "Why not WireGuard?"
    WireGuard is excellent for point-to-point tunnels. But a homelab with 4+ hosts needs mesh connectivity — every host must reach every other host. With WireGuard, that means N×(N-1)/2 tunnels, each manually managed. Nebula's mesh overlay handles this automatically: add a host, and it discovers and connects to every other host.
