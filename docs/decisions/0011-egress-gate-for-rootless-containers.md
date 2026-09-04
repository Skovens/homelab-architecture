---
title: "ADR-0011: Egress Gate"
---

# ADR-0011: Egress gate for rootless containers

## Status

Accepted

## Context

Rootless Podman's default network mode on this server is **pasta**. At unit start pasta snapshots the host's addressing, routes and DNS configuration (`--config-net`) and never refreshes it while running.

Incident 2026-08-24: boot at 15:02:33, containers started 15:02:50–52 (~17 s after boot). At that moment LAN/DNS had not settled, so every pasta instance snapshotted an unsettled network. Consequences for the rest of the day:

- Host egress worked; **all container egress failed instantly** (raw-IP connect refused, DNS dead).
- OpenHands Agent Canvas could not reach its hosted MCP endpoint — "MCP tool listing timed out after 30 seconds" and "Could not reach the server".
- HomeAssistant lost weather data — same root cause.

Restarting the affected user services forced fresh snapshots and fixed everything immediately. The repo history confirmed no networking misconfiguration — this is a pure boot-order race, and it recurs on any reboot where networking takes longer than ~15–20 s to settle.

## Decision

Gate all rootless container units behind an egress wait, implemented centrally in `podman-container-base` (every container role already imports it):

- New script `~/.local/bin/wait-for-egress` (per service user): bounded loop (default 120 s, 3 s interval) that exits 0 only when the host has a default route, a successful TCP probe (`1.1.1.1:443`), and working DNS (`getent hosts github.com`).
- systemd drop-in `<unit>.service.d/egress-gate.conf` per managed unit (and per managed pod):
    - `ExecStartPre=/home/<user>/.local/bin/wait-for-egress`
    - `TimeoutStartSec=<timeout + 30>s`
    - `RestartSec=15s` (prevents a 100 ms-default restart hammer-loop while the network is down)
- Default **on** for every role importing the base, including offline-capable services. Opt-out per role via `pcb_egress_gate: false`.
- Drop-ins live outside the quadlet templates, so quadlet files stay untouched and future roles get the gate automatically.

## Consequences

- **Positive:** Boots self-heal — containers simply start later, healthy; no stale-snapshot state possible; single implementation point covers all current and future roles.
- **Negative:** Offline-capable services (llamacpp serving local clients) also wait for internet at boot — accepted for simplicity over per-role bookkeeping. Playbook runs block up to ~2 min per gated start if the network is down mid-run. Crash-loop recovery slows to ≥ 15 s per attempt (`RestartSec`).
- **Operational rule:** Never remove or bypass the gate without solving the underlying snapshot race; new container roles get the gate for free via `podman-container-base`.

## Related

- ADR-0001 (rootless users)
- ADR-0002 (`--machine=` user-scoped systemd)
- ADR-0003 (shared pod)
