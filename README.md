# Jobhunt — multi-agent job research

Markdown agents that discover, verify, cross-check, and match jobs. This is a **research organization**, not a scraper.

An executing AI loads the orchestrator, spawns specialist agents, and returns evidence-backed case files. Works in **Grok**, **Claude Code**, and **Cursor**; the orchestrator's *Runtime adapters* table maps generic tool names to each.

## Run a research pass

1. Edit the candidate YAML in [`agents/matching/candidate-profile.md`](agents/matching/candidate-profile.md)
2. Open the repo in your AI runtime and run **`/job-research`**, or ask: `Find currently open jobs that match my profile and verify they are real.`
3. Optional focus: `Is Acme actually hiring a Java engineer?` runs the company pipeline only

The parent session follows [`agents/orchestrator.md`](agents/orchestrator.md). It delegates; it does not invent listings from memory.

Budgets: `quick` (6 jobs deep-verified), `standard` (12, default), `deep` (25). Say `budget: deep` in your request to change it.

## What a run produces

```text
research/runs/2026-09-12/
├── summary.md            Verified / Needs verification / Stale-closed / Conflicted
├── state.json            orchestrator run state (leads, canonical jobs, agent results)
├── jobs/{job_key}.md     one case file per deep-verified job, with research trace
└── evidence/{job_key}.md claim-level FACT / INFERENCE / UNCERTAINTY / CONFLICT
```

`research/runs/` is gitignored. If the runtime cannot write files, the same documents are printed in the conversation.

## Repo map

| Path | Purpose |
|------|---------|
| [`AGENTS.md`](AGENTS.md) | Operating instructions every AI session loads |
| [`agents/`](agents/) | Agent contracts — orchestrator plus 19 specialists |
| [`skills/`](skills/) | Shared schemas and rules (single source of truth) |
| [`docs/nepal-portal-scraper-spec.md`](docs/nepal-portal-scraper-spec.md) | Separate, **unimplemented** Python scraper spec; only its portal list is used, by `job-board-discovery`, when the candidate is in Nepal |
| `.grok/` `.claude/` `.cursor/` | `/job-research` skill entry points per runtime |

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
| `web-search.md` | Careers / web fetch tactics, ATS public JSON endpoints |
| `freshness-analysis.md` | ACTIVE / RECENT / STALE / CLOSED / UNKNOWN |
| `duplicate-analysis.md` | Same-job merge rules |
| `contradiction-analysis.md` | Conflict records |

## Quality bar

- No single agent is the source of truth
- GitHub and job boards are **leads**
- `VERIFIED` requires an official or ATS page fetched in the run
- Unknown is better than a guess
- Conflicts are first-class output

## Extending

- **New agent**: copy the section order from any existing agent (see `AGENTS.md` → *Editing this repo*), add it to the table in `agents/orchestrator.md` and to `agents/README.md`. Its Output Format is a contract; name the consumer.
- **New source type or freshness rule**: edit the skill, not the agent. Agents cite skills; they do not restate them.
- **New candidate**: only `agents/matching/candidate-profile.md` changes.
