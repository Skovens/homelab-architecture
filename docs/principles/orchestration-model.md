---
title: Orchestration Model
---

# Orchestration Model

Orchestration is a matter of choosing the smallest tool that does the job well,
integrates with the system natively, and doesn't require a daemon running as
root.

## systemd as the Orchestrator

I use systemd with Quadlets to manage container lifecycles. systemd is already
present on every host, handles start/stop/restart, captures logs via
`journalctl`, and reports state via `systemctl`. There is no separate
orchestration daemon to run or keep healthy.

At the scale of this homelab, this covers everything I need without the
operational overhead of a dedicated orchestration platform. For a larger
deployment I might reach for Kubernetes — but for now, systemd does the job.

## Declarative State

Everything is declared rather than imperative. Container configuration lives in
Quadlet files. System configuration and service deployment are driven by
Ansible. If it's not in the repo, it doesn't exist on the server. This means the
entire environment can be rebuilt from scratch, and every change is reviewed,
versioned, and auditable.

## Intentional Simplicity

I gravitate toward tools that do one thing well and integrate with the system
natively. Podman over Docker. Quadlets over Compose. Sanoid over a hand-rolled
backup script.

The same thinking applies to monitoring. I use healthchecks.io rather than a
full metrics stack. The key difference is active verification: my phone checks
in on each service and confirms it actually works, rather than assuming an
alerting pipeline is functional. I don't want a set-and-forget system — I want
continuous proof that things are running.
