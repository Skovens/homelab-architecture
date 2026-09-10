---
title: MCP Servers
---

# MCP Servers

Three MCP (Model Context Protocol) servers give Open WebUI's AI agents access to external tools: GitHub, Forgejo (self-hosted git), and an Obsidian vault.

## How

### GitHub MCP (port 8082)

Runs via mcpo — a stdio-to-HTTP translation proxy. The GitHub MCP server is a stdio process; mcpo wraps it in an OpenAPI-compatible HTTP endpoint.

GitHub PAT stored as a Podman secret.

### Forgejo MCP (port 8085)

Runs the native Forgejo MCP server with Streamable HTTP transport — no mcpo wrapper needed.

Forgejo PAT stored as a Podman secret.

### Vault MCP (port 8015)

A custom build wrapping `@modelcontextprotocol/server-filesystem` + `supergateway` (stdio-to-HTTP). Mounts the Obsidian vault at `/vault` and exposes 8 MCP tools (read, write, edit, search).

Published to `10.0.0.100:8095` for VPN access. No authentication — the VPN mesh is the trust boundary.

## Why

The three transport patterns reflect the current state of MCP server implementations. mcpo translates stdio to HTTP for GitHub; Forgejo speaks HTTP natively; supergateway translates the filesystem MCP server from stdio to HTTP.

!!! warning "Vault MCP has no auth"
    The Vault MCP server has no authentication. Anyone on the VPN mesh can read and write to the Obsidian vault. This is acceptable because the VPN is the trust boundary — mesh enrollment requires a token, and all traffic is encrypted.
