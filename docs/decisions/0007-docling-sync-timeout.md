---
title: "ADR-0007: Docling Timeout"
---

# ADR-0007: Docling sync timeout raised instead of PDF size cap

## Status

Accepted

## Context

Large PDFs (13 MB+) exceeded the docling-serve synchronous conversion default `DOCLING_SERVE_MAX_SYNC_WAIT=120` seconds. The sync endpoint returned 504, and Open WebUI re-submitted the document on every sync cycle, pinning docling-serve at full CPU. The alternative considered was capping PDF size in the oikb filter (ADR-0006) — rejected because it silently drops legitimate documents.

## Decision

Support large documents by **raising sync timeouts**, not by capping size:

- `DOCLING_SERVE_MAX_SYNC_WAIT: 600` — official docs recommend 600s for large documents.
- `docling_serve_eng_loc_num_workers: 4` plus thread knobs (`DOCLING_NUM_THREADS` / `OMP_NUM_THREADS` / `MKL_NUM_THREADS=4`) for conversion throughput.
- **`UVICORN_WORKERS` stays at 1** — multiple workers cause "Task Not Found" errors with async conversion.

## Consequences

- **Positive:** Large PDFs convert successfully; no documents silently dropped; no re-submit storm.
- **Negative:** A single conversion can hold a sync request for up to 600s; resource tuning (workers/threads) is a balance — workers ≠ single-doc latency due to the GIL, hence the thread knobs.

## Related

- ADR-0006 (the filter allowlist is unchanged — no size cap added)
