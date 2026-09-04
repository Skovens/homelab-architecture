---
title: Egress Gate
---

# Egress Gate

## What

Rootless Podman's default network mode (pasta) snapshots the host's network configuration at container start and never refreshes it. On a fresh boot, this means containers can start before the network is ready — and they stay broken until restarted.

## How

A `wait-for-egress` script is deployed to every container user's `~/.local/bin/`. A systemd drop-in adds it as `ExecStartPre` to every container unit:

```ini title="myapp.service.d/egress-gate.conf"
[Service]
ExecStartPre=/home/myapp/.local/bin/wait-for-egress
TimeoutStartSec=150
RestartSec=15
```

The script runs a bounded loop (default 120 seconds, 3-second interval) that checks:

1. A default route exists
2. A TCP probe succeeds (`1.1.1.1:443`)
3. DNS resolves (`getent hosts github.com`)

Only when all three pass does the script exit 0 and allow the container to start.

The gate is implemented centrally in the `podman-container-base` role — every container role imports it, so new roles get the gate automatically. Opt-out per role via `pcb_egress_gate: false`.

## Why

On 2026-08-24, the server booted at 15:02:33. Containers started at 15:02:50 — 17 seconds after boot. At that moment, the LAN and DNS had not settled. Every pasta instance snapshotted an unsettled network.

The result: all container egress failed instantly. OpenHands couldn't reach its MCP endpoint. HomeAssistant lost weather data. The fix was simple — restart the affected services — but the root cause recurred on every reboot where networking took longer than ~15 seconds to settle.

Restarting the affected user services forced fresh snapshots and fixed everything immediately. The repo history confirmed no networking misconfiguration — this is a pure boot-order race.

!!! warning "Pasta doesn't refresh"
    Pasta's `--config-net` snapshots host networking at start and never updates it. If the network changes after the container starts (e.g., DNS becomes available, default route appears), the container never sees it. This is by design in pasta — it's a feature for network isolation, but a problem for boot sequences.
