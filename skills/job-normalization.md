---
name: job-normalization
description: Canonical job record schema and field-normalization rules. Use when converting raw discoveries into a single job object before verification, matching, or reporting.
---

# Job Normalization

Single source of truth for the canonical job record. Discovery agents emit raw jobs; the orchestrator (or duplicate agent) normalizes into this schema. Unknown fields are `null`. Never fill a field to make the record look complete.

## Canonical job

```json
{
  "job_key": "acme|senior-solidity-engineer|remote",
  "company": "Acme",
  "role": "Senior Solidity Engineer",
  "location": "Remote",
  "remote": true,
  "employment_type": "full-time",
  "department": null,
  "source_urls": ["https://github.com/org/repo"],
  "official_job_url": null,
  "company_url": null,
  "job_id": null,
  "ats": null,
  "discovered_at": "2026-09-12T12:00:00Z",
  "posting_date": null,
  "technologies_mentioned": ["Solidity", "EVM"],
  "raw_excerpt": "Acme — Senior Solidity Engineer — Remote",
  "source_type": "github_community",
  "pipeline": "github"
}
```

## `job_key`

Lowercase `normalized_company|normalized_role|normalized_location`.

- Company: strip `Inc.`, `Ltd.`, `LLC`, `GmbH`, punctuation; collapse whitespace
- Role: strip seniority tokens for the key only (`senior`, `staff`, `sr.`, `l5`) so "Senior X" and "X" can be compared later; keep the display `role` intact
- Location: `remote` if remote/anywhere/distributed; otherwise the most specific place mentioned

If company or role is missing, do not invent a key. Leave the record in a `needs_normalization` pile.

## Field rules

| Field | Rule |
|-------|------|
| `remote` | `true` / `false` / `null`. `null` if unstated. "Hybrid" is `false` plus location text |
| `employment_type` | `full-time`, `contract`, `internship`, `part-time`, or `null` |
| `official_job_url` | Set only after an official or ATS URL is fetched and returns a live job page |
| `job_id` | ATS id or official slug you observed. Never generate one |
| `ats` | `greenhouse` / `lever` / `ashby` / `workday` / `workable` / `other` / `null` |
| `posting_date` | ISO date copied or parsed from the document. Relative phrases ("2 days ago") may be resolved against `observed_at` and stored as `INFERENCE`. If no date signal, `null` |
| `technologies_mentioned` | Tokens observed in the listing. Extraction rules live in `skills/technology-detection.md` |

## Pipeline tag

Set `pipeline` to one of: `github`, `github_org`, `company`, `job_board`. Used by the orchestrator to choose the next agents, not as a quality score.

## Lead ids

Discovery agents do not assign ids. On receipt, the orchestrator stamps every raw lead with `lead_id` = `{agent-short-name}-{n}` (`gh-1`, `repo-3`, `board-2`, `co-1`) in arrival order. Duplicate-detection refers to leads by `lead_id` in `members`; reports keep `lead_id` in `source_urls` provenance. Never renumber after assignment.

## Raw jobs vs canonical jobs

Discovery output is a raw job: it may be messy. Normalization happens before verification. Do not verify a record that still lacks `company` and `role`.
