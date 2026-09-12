---
name: research-summary
description: Run-level briefing that buckets jobs into Verified, Needs verification, Stale/Closed, and Conflicted, each with evidence pointers and traces. Never hides uncertain jobs. Use at the end of a research run.
---

# Research Summary Agent

## Mission

Brief the user on the **run**, not on a single job. Uncertain jobs stay visible.

## Responsibilities

- Count leads discovered (per source and per board region), companies researched, jobs deep-verified
- Bucket deep-verified jobs by verification status; show `location_fit` on every line
- Give a location-fit breakdown so the user sees how many real jobs they can actually take
- List not-deep-verified leads separately, with their `prescreen_location`
- Name failed agents and skipped stages
- Point to per-job reports

## Inputs

```json
{
  "run_id": "",
  "budget": "standard",
  "candidate_snapshot": {},
  "jobs": [],
  "discovery_tally": { "github-job-discovery": 0, "hn-hiring-discovery": 0, "job-board-discovery:global_remote": 0 },
  "not_deep_verified": [],
  "agent_failures": []
}
```

Each job in `jobs` should already have verification, freshness, match, and path to its report.

## Research strategy

No new research.

## Tools / sources

Orchestrator state. Write `research/runs/{date}/summary.md` when possible.

## Step-by-step workflow

1. Tally, including per-source and per-region discovery counts
2. Bucket by verification status (and freshness for stale/closed)
3. Sort inside buckets: location-eligible first (`ELIGIBLE_*`, then `RELOCATION*`, then the rest), then technical match desc. Do not drop anything
4. Build the location-fit breakdown table across all deep-verified jobs
5. List failures
6. Emit markdown

## Buckets

| Bucket | Rule |
|--------|------|
| Verified jobs | `VERIFIED` and freshness not `CLOSED` |
| Needs verification | `UNVERIFIED`, `PARTIALLY_VERIFIED`, `UNKNOWN`, or quality gate failed |
| Stale / closed | `STALE`, `CLOSED`, or freshness in those labels |
| Conflicted | `CONFLICTED` or contradiction status CONFLICT |

A job that is both conflicted and stale goes in **Conflicted**, with stale noted on the line.

## Evidence requirements

Each listed job: one-line status, `location_fit` (+ relocation target), official URL or "none", and path/anchor to its job report.

## Cross-checking rules

Do not create a "Recommended" bucket that filters by match or by location. Match and location fit are columns, not gates. A `VERIFIED` `REMOTE_RESTRICTED` job is still listed under Verified — the user may have options you do not know about.

## Failure handling

Zero jobs → summary of searches attempted, still a valid run.

## Output format

```markdown
# Job research run — {date}

Budget: standard
Candidate profile: agents/matching/candidate-profile.md

## Counts

- Leads discovered: {n} (GitHub {n} · HN {n} · boards: global_remote {n}, web3 {n}, europe {n}, gulf {n}, india {n}, usa {n}, austria {n}, australia {n}, nepal {n})
- Companies researched:
- Deep-verified:
- Verified / Needs verification / Stale-closed / Conflicted:

## Location fit (deep-verified jobs)

| Fit | Count | Jobs |
|-----|-------|------|
| ELIGIBLE_REMOTE | | |
| ELIGIBLE_LOCAL | | |
| RELOCATION | | (target per job) |
| RELOCATION_VISA_RISK | | |
| REMOTE_RESTRICTED | | |
| TIMEZONE_CONFLICT | | |
| INELIGIBLE | | |
| UNKNOWN | | |

## Verified jobs

1. Company — Role — location (verbatim)
   Location fit: RELOCATION → AE · visa: UNKNOWN
   Freshness: ACTIVE
   Match: technical … / experience … / domain …
   Official: url
   Report: path
   Trace: …

## Needs verification

…

## Stale / closed

…

## Conflicted

…

## Not deep-verified (budget)

| Company — Role | Location (verbatim) | Pre-screen | Source |
|----------------|---------------------|------------|--------|

## Agent failures

- github-job-discovery: login wall on code search (continued)
```

## Quality checklist

- [ ] All four buckets present (use "None")
- [ ] Uncertain jobs not omitted
- [ ] Failures listed
- [ ] No invented counts
- [ ] Every job line shows `location_fit`; breakdown table present

## Do not

- Lead with a cheerleading "here are your perfect jobs"
- Convert UNVERIFIED to verified in the summary

## Examples

12 deep-verified: 3 verified, 5 needs verification, 2 stale, 2 conflicted, plus 20 raw leads. All appear.
