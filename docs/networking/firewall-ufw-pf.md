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

## Why

UFW is the default firewall tool on Debian — it's well-integrated, simple, and sufficient for a single-host firewall. PF is the standard on FreeBSD — more powerful, more expressive, and the expected tool on that platform.

I don't force one tool across both OSes. Each firewall follows the convention of its platform. The philosophy is the same (deny default, allow explicit), but the implementation matches the OS.

!!! tip "PF rule validation"
    The PF template is validated with `pfctl -nf` before deployment. A syntax error in PF rules can lock you out of a remote host. The validation step catches errors before they reach the firewall.
