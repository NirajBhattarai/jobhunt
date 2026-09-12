---
name: duplicate-analysis
description: Rules for deciding when two job listings are the same underlying role. Use during duplicate detection and when merging GitHub, ATS, and job-board records into one canonical job.
---

# Duplicate Analysis

Single source of truth for duplicate decisions. Output a canonical record plus members.

## Signals (strongest first)

| Signal | Weight |
|--------|--------|
| Identical application URL or ATS job id | Definite same job |
| Same company + same ATS slug + similar title | Very likely |
| Same official job URL | Definite |
| Same company + normalized title + same location | Likely |
| Same company + overlapping description sentences | Likely |
| Same company only | Not enough |

## Title similarity

Normalize before comparing: lowercase, strip `senior`/`staff`/`sr`/`ii`/`iii`, strip punctuation.

- Exact normalized title → title match
- One is a prefix of the other ("engineer" vs "software engineer") → weak title match; require location or URL overlap
- Unrelated titles at the same company → different jobs

## Decision

```text
SAME        — merge into one canonical job
LIKELY_SAME — merge, keep a CONFLICT if titles differ materially
DIFFERENT   — keep separate
UNKNOWN     — do not merge
```

When merging, the canonical record prefers the highest-reliability `source_type` (`skills/source-validation.md`) for `official_job_url` and description. Union all `source_urls`. Preserve every member id in `duplicate_of` / `members`.

## Cross-board pattern

The same role often appears as:

- GitHub community README
- Company careers page
- Greenhouse/Lever/Ashby
- LinkedIn / Wellfound / web3.career

That is one job, not four, once URLs or (company + title + location) line up.
