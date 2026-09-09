---
title: Healthchecks
---

# Healthchecks

## What

healthchecks.io runs as a lightweight ping-based monitoring service. Services send periodic pings; if a ping doesn't arrive, healthchecks alerts.

## How

Runs as `docker.io/healthchecks/healthchecks:latest` with a Podman secret for `secret_key`.

Accessible at `healthchecks.example.com` via the Caddy reverse proxy.

## Why

Ping-based monitoring aligns with the [Orchestration Model](../../principles/orchestration-model.md) — the only question that matters is "is it working?" healthchecks.io answers that with minimal overhead. The tradeoff is reduced visibility into metrics like CPU or latency, which is acceptable at this scale.
