---
title: Knowledge Base
---

# Knowledge Base

oikb syncs external sources into Open WebUI Knowledge Bases for RAG (Retrieval-Augmented Generation). Currently it syncs the Obsidian vault.

## How

oikb runs on port 8084 and pulls from a local directory. The Obsidian vault is a git repository cloned to the host, with an hourly git pull keeping it current.

File type filtering restricts sync to docling-supported formats:

```yaml
oikb_docling_filter: &oikb_docling_filter
  include_extensions:
    - ".pdf"
    - ".md"
    - ".txt"
    - ".png"
    - ".jpg"
    - ".jpeg"
    - ".csv"
    - ".json"
```

The YAML anchor (`&oikb_docling_filter`) is shared across all sources — adding a new source automatically inherits the filter.

## Why

The file type filter restricts sync to docling-supported formats. Without it, unsupported files would trigger conversion failures on every sync cycle.

!!! note "Forgejo connector pending"
    A Forgejo connector for oikb is not yet available (oikb PR #12). When it lands, the vault will sync directly from Forgejo instead of a local git clone.
