---
title: Firewalls
---

# Firewalls

## What

Two firewalls, two operating systems, one philosophy: deny by default, allow only what's needed.

## How

### Debian (Home Server + Proxy) — UFW

```yaml
firewall_default: deny
firewall_incoming:
  - { port: 22, proto: tcp }     # SSH
  - { port: 8090, proto: tcp }   # llama.cpp GPU
  - { port: 8091, proto: tcp }   # llama.cpp CPU
  # ... additional service ports
firewall_outgoing: allow
```

## How

### Debian (Home Server + Proxy) — UFW

```yaml
firewall_default: deny
firewall_incoming:
  - { port: 22, proto: tcp }     # SSH
  - { port: 8090, proto: tcp }   # llama.cpp GPU
  - { port: 8091, proto: tcp }   # llama.cpp CPU
  # ... additional service ports
firewall_outgoing: allow
```

### FreeBSD (DR Host) — PF

PF rules are deployed via Jinja2 template and validated with `pfctl -nf` before activation:

```pf
block in all
pass out all
pass in on egress proto tcp to any port 22
```

Enabled in `rc.conf`:

```ini
pf_enable="YES"
pf_rules="/etc/pf.conf"
```

To ensure no stale rules persist, the Ansible play always enforces the desired state. It compares the current active firewall configuration against the intended template; if they do not match, it wipes the existing rules and reapplies the new configuration from scratch.

## Why

The deny-by-default posture is part of the [security model](../../principles/security-model.md) — defense in depth means every host enforces its own firewall, not just the network edge. UFW is the default on Debian; PF is the standard on FreeBSD. Each firewall follows the convention of its platform rather than forcing one tool across both OSes.

!!! tip "PF rule validation"
    The PF template is validated with `pfctl -nf` before deployment. A syntax error in PF rules can lock you out of a remote host. The validation step catches errors before they reach the firewall.
