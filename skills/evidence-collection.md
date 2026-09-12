---
name: evidence-collection
description: Shared evidence standard for job research. Every factual claim needs a source, excerpt, claim type, and confidence. Use when recording, validating, or combining research claims.
---

# Evidence Collection

Single source of truth for how claims are recorded. All agents follow this file. Do not invent a parallel schema.

## Claim types

| Type | Meaning | Allowed without a live source? |
|------|---------|--------------------------------|
| `FACT` | Directly observed on a retrieved page or API payload | No |
| `INFERENCE` | Derived from facts. Must state the derivation | No — rest on cited facts |
| `UNCERTAINTY` | Investigated, still unknown | Yes — record what was tried |
| `CONFLICT` | Two or more sources disagree | No — cite both sides |

Never present `INFERENCE` as `FACT`. Never convert `UNCERTAINTY` into a positive verification.

## Evidence object

```json
{
  "claim": "Acme lists a Senior Solidity Engineer on its careers page",
  "claim_type": "FACT",
  "source_url": "https://acme.example/careers/solidity",
  "source_type": "official_company",
  "excerpt": "Senior Solidity Engineer — Remote — Apply",
  "observed_at": "2026-09-12T12:04:00Z",
  "confidence": 0.91,
  "agent": "job-verification"
}
```

Field rules:

- `claim` — one testable sentence.
- `source_url` — the URL actually fetched. Not a guessed canonical URL.
- `source_type` — from `skills/source-validation.md`.
- `excerpt` — short quote or structured snippet copied from the page. Empty excerpts are invalid for `FACT`.
- `observed_at` — timestamp of the fetch, ISO-8601. Not the job's posting date.
- `confidence` — 0.0–1.0 from source reliability and how directly the excerpt supports the claim. Not a vibe score.

## What counts as evidence

Valid:

- Page text you fetched in this run
- GitHub README, issue, discussion, or commit metadata you fetched
- HTTP status of an application URL you requested
- Structured fields on an ATS page you opened (Greenhouse, Lever, Ashby, Workday)
- Dates that appear in the retrieved document

Invalid:

- Training-data recollection of a company or job
- "Probably still hiring"
- A URL you did not fetch
- A date you computed without a document signal
- Another agent's prose without its evidence objects

## Confidence calibration

Start from the source reliability score in `skills/source-validation.md`, then adjust:

- Direct quote of the claim on the page: keep or raise slightly
- Partial / ambiguous mention: cap at 0.6
- Third-party restatement of an official page you did not open: cap at 0.45
- Broken link, interstitial, or login wall: do not emit `FACT`; emit `UNCERTAINTY`

## Bundling

When an agent returns findings, every non-uncertain finding includes `evidence: Evidence[]`. An empty evidence array is allowed only on `UNCERTAINTY` records that list `attempts`.
