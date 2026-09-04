---
title: "ADR-0006: Docling Filter"
---

# ADR-0006: oikb sources limited to docling-supported types

## Status

Accepted

## Context

oikb syncs external sources (GitHub repos, local directories) into Open WebUI Knowledge Bases, converting documents with the deployed docling-serve instance. The docling-serve image `quay.io/docling-project/docling-serve-cpu` ships **docling 2.117.0 with no `soffice` (LibreOffice) and no `ffmpeg`** — so legacy Office formats (`.doc`/`.xls`/`.ppt`) and audio/video are not convertible. Unfiltered sync sources produced conversion failures on every sync cycle.

## Decision

Restrict every oikb sync source to the `docling_filter` allowlist defined once in `playbooks/group_vars/home_server.yml`:

- The allowlist is the **intersection** of what the deployed docling-serve image can convert and Open WebUI's accepted KB upload types.
- Excluded types: `.doc`/`.xls`/`.ppt` (needs LibreOffice), audio/video (needs ffmpeg + ASR extra).
- The filter is shared across all sources via a YAML anchor, so a new source inherits the restriction automatically.

## Consequences

- **Positive:** No sync failures from unsupported formats; one source of truth for the allowlist; adding a format is a single edit.
- **Negative:** Office/AV content in source repos is silently skipped — operators must know the intersection rule; changing the docling-serve image (e.g., the full `docling-serve` variant with soffice) would require re-reviewing the allowlist.

## Related

- ADR-0007 (timeout policy for large documents)
