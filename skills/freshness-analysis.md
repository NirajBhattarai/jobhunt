---
name: freshness-analysis
description: Rules for classifying a job as ACTIVE, RECENT, STALE, CLOSED, or UNKNOWN. Forbids invented dates. Use during freshness verification and whenever a posting age is reported.
---

# Freshness Analysis

Single source of truth for freshness labels. The freshness agent applies this file; other agents must not assign these labels on their own.

## Labels

| Label | Meaning |
|-------|---------|
| `ACTIVE` | Official or ATS page currently shows the role, and the application URL works |
| `RECENT` | Independent source updated within 30 days, official confirmation missing |
| `STALE` | Best available source is older than 60 days, or official page no longer lists the role while a third-party still does |
| `CLOSED` | Official/ATS page gone (404/410), or page states the role is filled/closed |
| `UNKNOWN` | No trustworthy date or live-page signal |

`ACTIVE` requires an official or ATS fetch in this run. A GitHub README updated yesterday is at most `RECENT`.

## Date signals (observe only)

Accept:

- ISO or calendar dates written on the page
- GitHub file commit datetime you fetched
- HTTP `Last-Modified` if present
- ATS "posted on" / "updated on" fields
- Relative phrases ("3 days ago") resolved against `observed_at` — store as `INFERENCE` and keep the original phrase in the excerpt

Reject:

- Invented posting dates
- Using today's date as the posting date
- Inferring age from GitHub stars, issue ids, or "looks new"

If no signal exists, `posting_date` is `null` and the label is `UNKNOWN` unless `CLOSED`/`ACTIVE` can be decided from live page state alone.

## Live page state beats dates

1. Official/ATS 404 or "no longer accepting applications" → `CLOSED` even if a GitHub list is recent
2. Official/ATS 200 with matching title → `ACTIVE` even if a community list is old
3. Official page loads but the role is absent → not `ACTIVE`; usually `STALE` or `CLOSED` depending on wording; if wording is unclear, `UNKNOWN` plus a `CONFLICT` with the original lead

## Decision order

Apply in this order; stop at the first rule that fires.

1. Official/ATS fetched this run and returns 404/410, or states filled/closed → `CLOSED`
2. Official/ATS fetched this run, 200, title matches, apply URL works → `ACTIVE` (age of any community date is irrelevant)
3. Official/ATS fetched this run, 200, role absent → `STALE` if a third-party still lists it; `UNKNOWN` + `CONFLICT` if the page wording is ambiguous
4. No official fetch this run → use date thresholds below

## Date thresholds (only when rule 4 applies)

Measured from `observed_at` to the best observed date signal:

- ≤ 30 days → `RECENT`
- 31–60 days → `UNKNOWN` (note the age in `signals`)
- \> 60 days → `STALE`
- No date signal → `UNKNOWN`

Do not use other thresholds. Do not output `ACTIVE` from dates alone.
