# AGENTS.md — operating instructions for AI sessions in this repo

This repository is a **multi-agent job research organization written in markdown**. There is no application code to build or test. The "program" is the set of agent contracts under `agents/` and shared rules under `skills/`, executed by an AI session that can spawn subagents and fetch web pages.

## What to do when asked to find or verify jobs

1. Load `agents/orchestrator.md` and follow it as your complete operating instructions.
2. Read the candidate from `agents/matching/candidate-profile.md` only. Never hardcode the candidate anywhere else.
3. Spawn specialists as described in the orchestrator's **Runtime adapters** section. Specialists read one file under `agents/` and return only that file's Output Format.
4. Write run artifacts under `research/runs/{date}/` (gitignored) when you have write tools.

The project skill `/job-research` (registered for Grok, Claude Code, and Cursor) does exactly this.

## Non-negotiable rules

- Never invent companies, jobs, dates, salaries, stacks, or apply URLs. `UNKNOWN` beats a guess.
- A GitHub README or job-board listing is a **lead**, never verification. `VERIFIED` requires an official or ATS page fetched in this run.
- Verification labels come from `agents/verification/job-verification.md`; freshness labels from `agents/verification/freshness-verification.md`. The orchestrator propagates them and does not override them.
- Conflicts are output, not noise. Do not resolve a disagreement by dropping a source.
- Matching happens after verification and never gates it. Do not create a "recommended" bucket that hides unverified or low-match jobs.

## Editing this repo

- Every agent file follows the same section order: Mission, Responsibilities, Inputs, Research strategy, Tools / sources, Step-by-step workflow, Evidence requirements, Cross-checking rules, Failure handling, Output format, Quality checklist, Do not, Examples. Keep it when adding or editing agents.
- Shared schemas (evidence, source types, canonical job, freshness, duplicates, contradictions, technology graph) live in `skills/`. Agents point at them; they do not fork them.
- Field names in an agent's Output Format are a contract. If you rename one, update every consumer (usually the orchestrator, `job-report`, `evidence-report`, `research-summary`).
- `docs/nepal-portal-scraper-spec.md` is an **unimplemented** spec for a separate Python scraper. Do not implement it as part of a research run and do not delete it; `job-board-discovery` uses its portal list when the candidate is in Nepal.

## Layout

```text
AGENTS.md                      this file
README.md                      human overview
agents/                        agent contracts (orchestrator + specialists)
skills/                        shared rules and schemas (single source of truth)
docs/                          reference documents that are not agents
.grok/ .claude/ .cursor/       /job-research skill entry points per runtime
research/runs/                 run output (gitignored)
```
