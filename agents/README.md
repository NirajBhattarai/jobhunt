# Agent catalog

Markdown agents for job discovery, verification, and matching. The parent session loads `orchestrator.md` and spawns specialists. Specialists do not spawn children.

`../docs/nepal-portal-scraper-spec.md` is a different, unimplemented pipeline. Job-board discovery may use its portal names as sources; nothing else reads it.

## Graph

```text
orchestrator
  ├── discovery
  │     github-job-discovery
  │     hiring-repository-discovery
  │     company-job-discovery
  │     job-board-discovery  (× region)
  │     hn-hiring-discovery
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
  │     location-matching
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
| `location-normalization.md` | Work mode, remote scope, region expansion, time-zone windows, visa/relocation signals |

## Data flow between agents

| Producer | Field(s) | Consumer |
|----------|----------|----------|
| discovery agents (GitHub, HN, boards × region) | `leads[]` (raw, `location` verbatim, `visa_sponsorship` if structured) | orchestrator stamps `lead_id`, normalizes, location pre-screens → `duplicate-detection` |
| `location-matching` | `location_fit`, `relocation_target`, `location.*` | `job-report`, `research-summary` (sort key + breakdown, never a gate) |
| `duplicate-detection` | `canonical_jobs[]` with `members[]` | orchestrator `canonical_jobs` map, keyed by `job_key` |
| `company-job-discovery` | `target_role_found`, `matching_positions`, `absence_evidence` | `job-verification`, `freshness-verification`, `contradiction-analysis` |
| `freshness-verification` | `freshness`, `apply_url_http_status` | `job-verification`, `research-summary` |
| `job-verification` | `status`, `answers`, `evidence` | `contradiction-analysis`, `job-report`, `evidence-report`, `research-summary` |
| `technology-research` | `technologies[]` with `required`/`mentioned`/`inferred` | `role-classification`, `technical-matching`, `experience-matching` |
| matching agents | `technical`, `experience`, `domain`, `why` | `job-report`, `research-summary` (sort key only, never a gate) |

Renaming any field here requires updating every consumer in the same change.

## Adding an agent

1. Copy the section order from an existing file: Mission, Responsibilities, Inputs, Research strategy, Tools / sources, Step-by-step workflow, Evidence requirements, Cross-checking rules, Failure handling, Output format, Quality checklist, Do not, Examples
2. Use generic tool names (`web_search`, `open_page`); the orchestrator maps them per runtime
3. Point at `../skills/` for schemas instead of restating them
4. Include `"agent"` and `"trace"` in the Output Format, and `"status"` if the agent can fail
5. Add the file to the graph above, to the path table in `orchestrator.md`, and to the data-flow table if another agent consumes it

## How to run

In this repo, ask to research jobs or run `/job-research`. The orchestrator reads this catalog, loads the candidate profile, and delegates.
