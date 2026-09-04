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

I tried Prometheus + Grafana. I had 400 dashboards I never looked at and alert rules I never tuned. The signal-to-noise ratio was terrible.

healthchecks.io does one thing: tells me if a service is down. A service sends a ping every N minutes. If the ping doesn't arrive, healthchecks sends a notification. That's it.

The tradeoff is visibility — I can't see CPU usage, memory trends, or request latency. But for a homelab, the question that matters is "is it working?" not "how efficiently is it working?"
