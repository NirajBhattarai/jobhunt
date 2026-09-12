# Agent catalog

Markdown agents for job discovery, verification, and matching. The parent session loads `orchestrator.md` and spawns specialists. Specialists do not spawn children.

Keep `../agents.md` (Nepali portal scraper spec). It is a different, unimplemented pipeline. Job-board discovery may use those portal names as sources.

## Graph

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
  │     candidate-profile          ← editable data + snapshot agent
  │     role-classification
  │     technical-matching
  │     experience-matching
  └── output
        evidence-report
        job-report
        research-summary
```

## Shared skills (`../skills/`)

| Skill | Owns |
|-------|------|
| `evidence-collection.md` | Claim schema and claim types |
| `source-validation.md` | `source_type` and official vs third-party |
| `job-normalization.md` | Canonical job record |
| `technology-detection.md` | Stack graph and extraction |
| `github-search.md` | GitHub search tactics |
| `web-search.md` | Careers/web fetch tactics |
| `freshness-analysis.md` | ACTIVE / RECENT / STALE / CLOSED / UNKNOWN |
| `duplicate-analysis.md` | Same-job merge rules |
| `contradiction-analysis.md` | Conflict records |

## How to run

In this repo, ask to research jobs or run `/job-research`. The orchestrator reads this catalog, loads the candidate profile, and delegates.
