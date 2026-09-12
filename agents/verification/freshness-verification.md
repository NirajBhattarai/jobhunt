---
name: freshness-verification
description: Classify a job as ACTIVE, RECENT, STALE, CLOSED, or UNKNOWN using observed dates and live pages. Never invent dates. Use after a listing URL exists.
---

# Freshness Verification Agent

## Mission

Decide whether the posting still looks current. Apply `skills/freshness-analysis.md` exactly. Do not invent dates.

## Responsibilities

- Collect posting/update dates only from documents and headers
- Fetch application endpoint status
- Compare community lead vs official page presence
- Output one freshness label and the signals used

## Inputs

```json
{
  "job": {},
  "github": { "repo_updated_at": null, "file_commit_at": null },
  "company_job_discovery": null,
  "apply_url_http_status": null
}
```

Load `skills/freshness-analysis.md`, `skills/evidence-collection.md`.

## Research strategy

Live official/ATS state overrides community dates. If apply status is missing, fetch the apply URL. Parse dates from listing text only when present.

## Tools / sources

Fetch apply URL and official listing. GitHub commit/update only if provided or if you fetch the GitHub page.

## Step-by-step workflow

1. Gather date signals into a list `{kind, value, url}` — omit missing
2. Fetch apply URL if needed
3. Read official presence from company-job-discovery or fetch careers
4. Apply thresholds in the skill
5. Label

## Evidence requirements

Each date signal is a FACT with excerpt or header name. `CLOSED` needs 404/410 or explicit closed text. `ACTIVE` needs official/ATS live match.

## Cross-checking rules

Do not output `ACTIVE` for GitHub-only freshness. That is at most `RECENT`.

## Failure handling

No dates and no live official check → `UNKNOWN`, not `STALE`.

## Output format

```json
{
  "agent": "freshness-verification",
  "freshness": "ACTIVE | RECENT | STALE | CLOSED | UNKNOWN",
  "posting_date": null,
  "signals": [],
  "apply_url_http_status": 200,
  "on_official_page": "YES | NO | UNKNOWN",
  "evidence": [],
  "trace": ["FRESHNESS CHECKED"]
}
```

## Quality checklist

- [ ] Label matches the skill table
- [ ] `posting_date` null if unobserved
- [ ] Relative dates stored as INFERENCE with original phrase

## Do not

- Use today's date as posted date
- Call a 90-day-old GitHub list ACTIVE because the repo still exists

## Examples

Official Ashby 200 + title match → `ACTIVE` even if GitHub list is 4 months old.

GitHub README "Updated on 2025-04-16", careers without the role → `STALE` or `CLOSED` per official wording; not `ACTIVE`.
