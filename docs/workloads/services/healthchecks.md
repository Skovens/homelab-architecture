---
title: Healthchecks
---

# Healthchecks

Healthchecks.io provides active, heartbeat-based monitoring for all services in the homelab.

## What

Healthchecks.io is used to monitor service availability. Instead of a central server probing services, each service "pings" Healthchecks when it successfully completes a task or a health check.

## How

Every critical service (databases, web servers, backups) is configured to send an HTTP ping to a unique Healthchecks.io URL.

```bash
# Example cron job for a backup task
0 2 * * * curl -fsS --retry 3 https://hc-ping.com/your-uuid-here > /dev/null
```

The status of these pings is monitored via the Healthchecks.io dashboard. To ensure the monitoring pipeline itself is functional, I perform regular manual checks via my phone, verifying that the dashboard reflects the real-time state of the lab.

## Why

The "active verification" model is more resilient than passive probing. With passive probing (e.g., Prometheus), the monitoring server must have network access to the service. With active verification (Healthchecks), the service itself initiates the contact.

This means:
1. **Zero Inbound Requirement:** Services don't need open incoming ports for monitoring.
2. **Network Resilience:** If the monitoring server is temporarily unreachable, the service can retry.
3. **Closing the Loop:** By checking the dashboard via my phone, I am not just assuming the alerting pipeline works—I am actively verifying that the entire monitoring loop (Service $\rightarrow$ Healthchecks $\rightarrow$ Dashboard $\rightarrow$ User) is intact.

