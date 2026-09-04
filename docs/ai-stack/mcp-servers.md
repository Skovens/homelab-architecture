---
title: MCP Servers
---

# MCP Servers

## What

Three MCP (Model Context Protocol) servers give Open WebUI's AI agents access to external tools: GitHub, Forgejo (self-hosted git), and an Obsidian vault.

## How

### GitHub MCP (port 8082)

Runs via mcpo — a stdio-to-HTTP translation proxy. The GitHub MCP server is a stdio process; mcpo wraps it in an OpenAPI-compatible HTTP endpoint.

GitHub PAT stored as a Podman secret.

### Forgejo MCP (port 8085)

Runs the native Forgejo MCP server with Streamable HTTP transport — no mcpo wrapper needed.

Forgejo PAT stored as a Podman secret.

### Vault MCP (port 8095)

A custom build wrapping `@modelcontextprotocol/server-filesystem` + `supergateway` (stdio-to-HTTP). Mounts the Obsidian vault at `/vault` and exposes 8 MCP tools (read, write, edit, search).

Published to `10.0.0.100:8095` for VPN access. No authentication — the VPN mesh is the trust boundary.

## Why

MCP servers give AI agents the ability to act on external systems. Instead of just chatting about code, the agent can:

- Read and write to GitHub repositories
- Search and modify Forgejo repositories
- Read and search the Obsidian vault for context

The three transport patterns reflect the state of MCP server implementations:

- **mcpo (GitHub)**: the GitHub MCP server only speaks stdio, so a translation layer is needed
- **Native Streamable HTTP (Forgejo)**: the Forgejo MCP server speaks HTTP natively — simpler, fewer failure modes
- **supergateway (Vault)**: the filesystem MCP server speaks stdio, so supergateway translates to HTTP

!!! warning "Vault MCP has no auth"
    The Vault MCP server has no authentication. Anyone on the VPN mesh can read and write to the Obsidian vault. This is acceptable because the VPN is the trust boundary — mesh enrollment requires a token, and all traffic is encrypted.
