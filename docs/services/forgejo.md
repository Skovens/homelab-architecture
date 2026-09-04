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

Forgejo over Gitea because:

- Forgejo is community-governed (Gitea Ltd has a history of license changes)
- Rootless container support is first-class
- Feature set is identical for my use case (single-user, SSH, web UI)

SQLite over Postgres because:

- Single-user forge doesn't need Postgres concurrency
- One fewer container to manage
- Backup is a single file copy

!!! note "The Obsidian vault"
    The Obsidian vault (`ObsidianVault-Personal`) is the primary repository hosted on Forgejo. oikb syncs it into Open WebUI's knowledge base for RAG.
