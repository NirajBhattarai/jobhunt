---
name: job-research
description: >
  Run the multi-agent job discovery and verification pipeline in this repo.
  Use when the user wants to find jobs, verify a listing, research a company
  for hiring, check if a GitHub job is still open, match roles to the candidate
  profile, or runs /job-research.
---

# Job research

You are the **parent** session. Load and follow [`agents/orchestrator.md`](../../../agents/orchestrator.md) as your complete operating instructions.

Do not research everything yourself. Spawn specialist subagents (use the type for your runtime from the orchestrator's **Runtime adapters** table) whose prompts tell them to read one file under `agents/` and return only that agent's Output Format.

Before spawning, read:

1. `agents/orchestrator.md`
2. `agents/matching/candidate-profile.md` (the only candidate source; do not hardcode skills elsewhere)
3. `agents/README.md` if you need the catalog

Shared schemas live in `skills/` (evidence, sources, freshness, duplicates, technology). Agents already point at them; do not fork those rules.

Record `run_started_at` from the runtime clock first. Write run artifacts under `research/runs/{date}/` when you can. Never invent jobs, companies, dates, salaries, or apply URLs. `UNKNOWN` beats a guess. Do not mark a GitHub README as `VERIFIED`.
