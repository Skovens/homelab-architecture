---
title: Forgejo
---

# Forgejo

## What

Forgejo is my self-hosted git forge. It hosts the Obsidian vault repository and any private code that doesn't belong on GitHub.

## How

Runs as `codeberg.org/forgejo/forgejo:15-rootless` with SQLite — no Postgres dependency for a single-user instance.

Accessible at `git.example.com` via the Caddy reverse proxy. SSH on port 2222 (non-standard to avoid conflicts with the host SSH).

## Why

Forgejo is community-governed with first-class rootless container support — see the [Security Model](../../principles/security-model.md) for the broader trust rationale.

SQLite is sufficient for a single-user forge: no concurrency concerns, one fewer container, and backup is a single file copy.

!!! note "The Obsidian vault"
    The Obsidian vault (`ObsidianVault-Personal`) is the primary repository hosted on Forgejo. oikb syncs it into Open WebUI's knowledge base for RAG.
