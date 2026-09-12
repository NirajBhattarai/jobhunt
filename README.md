# Jobhunt — multi-agent job research

Markdown agents that discover, verify, cross-check, and match jobs. This is a **research organization**, not a scraper.

An executing AI (Grok, Claude Code, or similar) loads the orchestrator, spawns specialist agents, and returns evidence-backed case files.

## What this is not

The repo also contains [`agents.md`](agents.md), a **Nepali job-portal scraper spec** (unimplemented). Do not overwrite it. Job-board discovery may use those portal names when the candidate location includes Nepal.

## Run a research pass

1. Edit the candidate YAML in [`agents/matching/candidate-profile.md`](agents/matching/candidate-profile.md)
2. Ask: `Find currently open jobs that match my profile and verify they are real.`
3. Or invoke **`/job-research`** (project skill)

The parent session follows [`agents/orchestrator.md`](agents/orchestrator.md). It delegates; it does not invent listings from memory.

## Agent graph

```text
orchestrator
    ├── discovery
    │     github-job-discovery
    │     hiring-repository-discovery
    │     company-job-discovery
    │     job-board-discovery
    ├── research
    │     company-research
    │     github-company-research
    │     technology-research
    │     engineering-research
    ├── verification
    │     job-verification
    │     company-verification
    │     freshness-verification
    │     duplicate-detection
    │     contradiction-analysis
    ├── matching
    │     candidate-profile
    │     role-classification
    │     technical-matching
    │     experience-matching
    └── output
          evidence-report
          job-report
          research-summary
```

Catalog: [`agents/README.md`](agents/README.md)

Shared rules (single source of truth): [`skills/`](skills/)

| Skill | Owns |
|-------|------|
| `evidence-collection.md` | Claim schema (FACT / INFERENCE / UNCERTAINTY / CONFLICT) |
| `source-validation.md` | Official vs third-party, ATS hosts |
| `job-normalization.md` | Canonical job record |
| `technology-detection.md` | Stack graph (Solidity, Java, Node.js, combined roles) |
| `github-search.md` | GitHub discovery tactics |
| `web-search.md` | Careers / web fetch tactics |
| `freshness-analysis.md` | ACTIVE / RECENT / STALE / CLOSED / UNKNOWN |
| `duplicate-analysis.md` | Same-job merge rules |
| `contradiction-analysis.md` | Conflict records |

## Quality bar

- No single agent is the source of truth
- GitHub and job boards are **leads**
- `VERIFIED` requires an official or ATS page fetched in the run
- Unknown is better than a guess
- Conflicts are first-class output

## Outputs

Written under `research/runs/{date}/` when the parent can write files:

- `summary.md` — buckets: verified, needs verification, stale/closed, conflicted
- `jobs/{job_key}.md` — case file
- `evidence/{job_key}.md` — claim-level warrants
