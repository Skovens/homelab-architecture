---
title: Model Proxy
---

# Model Proxy

## What

llama-swap sits in front of the llama.cpp instances and manages model switching. It prevents aborted in-flight streaming responses when Open WebUI switches models.

## How

llama-swap runs as a pure HTTP proxy (no GPU needed) on port 8092. It uses a FIFO scheduler:

1. Open WebUI requests a model switch
2. llama-swap queues the switch request
3. Current streaming response completes
4. llama-swap tells llama.cpp to unload the old model and load the new one
5. New model is ready, queued request is served

Configuration:

```yaml
proxy: "http://localhost:8090"
routing:
  router:
    use: group
  swap: true
  exclusive: false
```

Model IDs in llama-swap must match the preset names in llama.cpp exactly.

## Why

Before llama-swap, Open WebUI switched models by telling llama.cpp to unload one while loading another. This **aborted in-flight streaming responses** — any user mid-conversation would see their response cut off.

llama-swap's FIFO scheduler queues model-switch requests until the current stream drains. Users never see aborted responses.

The proxy runs as `cmd: sleep infinity` in its quadlet — it's a long-running process managed by systemd, not a one-shot container. The actual proxy process is started as a second ExecStart in the quadlet.
