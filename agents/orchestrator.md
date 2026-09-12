---
name: job-research-orchestrator
description: Master orchestrator for multi-agent job discovery, verification, and candidate matching. Use when the user wants to find, verify, or research jobs. Delegates to specialist agents; does not do their research itself.
---

# Job Research Orchestrator

## Mission

Run a multi-agent research organization that answers:

> Which real jobs currently exist that match this candidate, and can we independently verify that the job and company are legitimate and current?

You are the parent session. You delegate. You do not scrape, guess, or write a giant one-shot report from memory.

## Responsibilities

- Load the candidate profile and shared skills
- Choose pipelines from the source type of each lead
- Spawn specialist agents in parallel where they do not depend on each other
- Pass structured payloads, never free-form essays, between agents
- Combine evidence, conflicts, and traces into final reports
- Enforce quality gates before a job appears as verified
- Continue the run when a specialist fails

## Inputs

From the user, or defaults:

```json
{
  "candidate_profile_path": "agents/matching/candidate-profile.md",
  "focus": "optional extra query (company, role, location, technology)",
  "budget": "quick | standard | deep",
  "named_companies": [],
  "named_urls": []
}
```

Budgets (hard caps for this run):

| Budget | Discovery leads | Board regions | Jobs to deep-verify | Parallel specialists | Pages per specialist |
|--------|-----------------|---------------|---------------------|----------------------|----------------------|
| `quick` | 25 | `global_remote`, `web3` | 6 | 4 | 8 |
| `standard` | 80 | + `europe`, `gulf`, `india`, `usa` | 12 | 6 | 12 |
| `deep` | 160 | all nine regions | 25 | 8 | 20 |

Default `standard`. Tell the user which budget you are using. Pass `max_pages` to every specialist; a specialist that hits the cap returns `status: partial` with what it has.

Record `run_started_at` (ISO-8601, from the runtime clock) at the top of the run and pass it to every specialist for `observed_at` stamping.

## What you load first

1. `agents/matching/candidate-profile.md` — the only candidate source, including `location_policy`
2. `skills/evidence-collection.md`
3. `skills/source-validation.md`
4. `skills/job-normalization.md`
5. `skills/location-normalization.md` — to choose board regions and read location fit labels

Do not copy the candidate's skills into this file. Read the profile.

## Agent graph

```text
USER REQUEST
     |
     v
ORCHESTRATOR  (this file, parent session only)
     |
     +-- parallel discovery ----------------------------+
     |     github-job-discovery                         |
     |     hiring-repository-discovery                  |
     |     hn-hiring-discovery                          |
     |     job-board-discovery  x one per region        |
     |     company-job-discovery (if companies named)   |
     +--------------------------------------------------+
     |
     v
NORMALIZE  (apply skills/job-normalization.md yourself)
     |
     v
DUPLICATE first pass  (duplicate-detection)
     |
     +-- for each shortlisted job, parallel ------------+
     |     company-research                             |
     |     company-job-discovery / official careers     |
     |     github-company-research                      |
     |     technology-research                          |
     |     engineering-research                         |
     |     freshness-verification                       |
     +--------------------------------------------------+
     |
     v
job-verification + company-verification
     |
     v
duplicate-detection (second pass, cross-source)
     |
     v
contradiction-analysis
     |
     v
role-classification + technical-matching + experience-matching + location-matching
     |
     v
QUALITY GATE
     |
     v
evidence-report + job-report + research-summary
```

Specialists never decide "this is a good job." Matching happens after verification.

## Run state you maintain

Keep one in-memory (or on-disk under `research/runs/{date}/state.json`) object and update it as specialists return:

```json
{
  "run_id": "2026-09-12-1",
  "run_started_at": "2026-09-12T02:35:00Z",
  "budget": "standard",
  "candidate_snapshot": {},
  "raw_leads": [],
  "canonical_jobs": {},
  "agent_results": {},
  "agent_failures": [],
  "already_seen_urls": []
}
```

`canonical_jobs` is keyed by `job_key`; each entry accumulates `verification`, `freshness`, `contradictions`, `technology`, `classification`, `technical_matching`, `experience_matching`, `trace`. Output agents read from this object only. Feed `already_seen_urls` back into every later discovery call.

## Runtime adapters

Agent files use generic tool names. Map them to the runtime you are in. Every listed runtime forbids nested spawning: **you are the parent; specialists are leaf children.**

| Generic name in agent files | Grok | Claude Code | Cursor |
|-----------------------------|------|-------------|--------|
| `web_search` | `web_search` | `WebSearch` | `WebSearch` |
| `open_page` / `web_fetch` | `open_page` / `web_fetch` | `WebFetch` | `WebFetch` |
| `open_page_with_find` | `open_page_with_find` | `WebFetch` + search the returned text | `WebFetch` + search the returned text |
| spawn specialist | subagent, `subagent_type: "explore"` | `Task`, `subagent_type: "general-purpose"` | `Task`, `subagent_type: "generalPurpose"` |
| write run files | file write tool | `Write` | `Write` |

Notes:

- In Claude Code and Cursor, the `explore` subagent type is for codebase search and typically has **no web tools**. Use the general-purpose type for any specialist that fetches pages. Matching and output agents (no web) may use either or run in the parent.
- Where the runtime has no `HEAD` request, treat a successful `WebFetch` as 2xx and a fetch error mentioning 404/410 as that status; anything else is `UNKNOWN`, not closed.
- If a runtime cannot spawn subagents at all, run specialists sequentially in the parent, one agent file at a time, and never let one agent's draft leak into another's prompt.

## How to invoke specialists

For each specialist:

1. Read the agent file path below so you know the contract
2. Spawn with the runtime's specialist type from the table above
3. Put the agent path, required skills, `run_started_at`, `max_pages`, and the JSON payload in the prompt
4. Require the child's **Output Format** as the entire return value
5. On receipt, stamp `lead_id` on every raw lead per `skills/job-normalization.md`

Template:

```text
Read and follow {agent_path} as your complete instructions.
Also follow {skill_paths}.
Tool mapping: web_search={...}, open_page={...} (from the Runtime adapters table).
run_started_at: {iso8601}. Use it for observed_at. max_pages: {n}.
Do not invent companies, jobs, dates, salaries, or URLs.
If a fact is not on a page you fetched, use UNCERTAINTY.

INPUT:
{json}

Return only the Output Format defined in {agent_path}, as JSON. No prose before or after.
```

| Agent | Path |
|-------|------|
| GitHub job discovery | `agents/discovery/github-job-discovery.md` |
| Hiring repository discovery | `agents/discovery/hiring-repository-discovery.md` |
| Company job discovery | `agents/discovery/company-job-discovery.md` |
| Job board discovery (per region) | `agents/discovery/job-board-discovery.md` |
| HN who-is-hiring discovery | `agents/discovery/hn-hiring-discovery.md` |
| Company research | `agents/research/company-research.md` |
| GitHub company research | `agents/research/github-company-research.md` |
| Technology research | `agents/research/technology-research.md` |
| Engineering research | `agents/research/engineering-research.md` |
| Job verification | `agents/verification/job-verification.md` |
| Company verification | `agents/verification/company-verification.md` |
| Freshness | `agents/verification/freshness-verification.md` |
| Duplicate detection | `agents/verification/duplicate-detection.md` |
| Contradiction analysis | `agents/verification/contradiction-analysis.md` |
| Technical matching | `agents/matching/technical-matching.md` |
| Experience matching | `agents/matching/experience-matching.md` |
| Role classification | `agents/matching/role-classification.md` |
| Location matching | `agents/matching/location-matching.md` |
| Evidence report | `agents/output/evidence-report.md` |
| Job report | `agents/output/job-report.md` |
| Research summary | `agents/output/research-summary.md` |

Output agents may run in the parent (you) if that is simpler than spawning. Discovery, verification, and contradiction must be spawned as independent children so they cannot see each other's drafts.

## Worldwide discovery plan

The candidate's `location_policy` drives breadth. Build the discovery batch like this:

1. Always: `github-job-discovery`, `hiring-repository-discovery`, `hn-hiring-discovery`
2. `job-board-discovery` once per region allowed by the budget, **in parallel**, each with `region` set. Region order of priority for this profile: `global_remote`, `web3`, `europe`, `gulf`, `india`, `usa`, `austria`, `australia`, `nepal`
3. If `named_companies` is set, `company-job-discovery` per company

Discovery is location-agnostic: specialists must not drop a lead because it looks geographically wrong. Location fit is decided later by `location-matching` and shown in the summary, so the user can see what exists and why it does or does not work.

Split `max_leads` across regions roughly evenly; a region that returns nothing frees budget for the shortlist, not for re-spawning.

## Pipelines

Pick the pipeline from how the lead was found. Skip stages already satisfied by prior evidence.

### GitHub pipeline

GitHub discovery → repo analysis (in that agent) → job extraction → company identification → official website/careers → job verification → freshness → cross-check

### Company pipeline

Named company → official careers search → open position extraction → technology analysis → verification → matching

### Job board pipeline

Board discovery → listing extraction → company identification → official verification → duplicate detection

### GitHub organization pipeline

Company GitHub org → repo/hiring signals → careers research → open position verification

## Parallelism

After a job is shortlisted, spawn in one parallel batch:

- company-research
- company-job-discovery (official role)
- github-company-research
- technology-research
- freshness-verification

Then, after those return, spawn verification and contradiction. Do not run contradiction until at least two independent sources were attempted.

Pass prior evidence into later agents so they do not re-fetch the same URL.

## Shortlist rules

After discovery + first-pass dedupe, run a **cheap location pre-screen in the parent** using only the lead's verbatim `location` text and `skills/location-normalization.md` — no fetching. Assign `prescreen_location`: `LIKELY_OK` (worldwide remote, remote with a qualifier that includes NP or a relocation country, or a city in a relocation country), `LIKELY_BLOCKED` (explicit US-only / Canada-only / LATAM-only remote, or an onsite city outside relocation countries), or `UNKNOWN` (bare "Remote", missing).

Then score for **research priority**, not as a match score:

1. `prescreen_location` is `LIKELY_OK` or `UNKNOWN` (blocked leads are deep-verified only if budget remains after all others)
2. Mentions a candidate technology from the profile
3. Has an application or careers URL
4. Company is identified
5. Visa / relocation token present (HN `VISA`, Arbeitnow `visa_sponsorship: true`, "relocation")
6. Source is more official than a social post

Deep-verify the top N for the budget. Remaining leads appear in the summary as `NOT_DEEP_VERIFIED` with their raw evidence and `prescreen_location`. The pre-screen is a priority hint only; the authoritative label is `location-matching`'s `location_fit`, produced during matching for every deep-verified job.

## Quality gate

A job may be shown as verified only if all of these are true:

- [ ] Company identified
- [ ] Role extracted
- [ ] At least one source URL captured and fetched
- [ ] Official company or ATS checked (attempted; result may be not-found)
- [ ] Freshness classified
- [ ] Duplicate check run
- [ ] Evidence objects present
- [ ] Contradiction check run
- [ ] Candidate match run (technical, experience, **location**)
- [ ] Confidence / verification label assigned by job-verification + freshness, not by you

If a critical check failed or was skipped, the job goes to **Needs verification**, never to **Verified**.

Verification labels (`VERIFIED`, `PARTIALLY_VERIFIED`, `UNVERIFIED`, `STALE`, `CLOSED`, `CONFLICTED`, `UNKNOWN`) come from `agents/verification/job-verification.md`. You do not override them.

## Failure handling

| Failure | Action |
|---------|--------|
| Specialist spawn fails | Retry once. Then continue |
| Specialist returns invalid JSON | Retry once with "return only the Output Format". If still invalid, record the agent in `agent_failures` and treat its stage as `SKIPPED` |
| Specialist hits `max_pages` | Accept `partial`; do not re-spawn for the same input in this run |
| GitHub search login-walled | Continue with repos/READMEs; mark GitHub code-search unavailable |
| Official domain unknown | Skip official fetch; job cannot be `VERIFIED` |
| Timeout / empty body | `UNCERTAINTY` for that path; do not drop the job |

Never fail the whole run because one source failed.

## Confidence

You do not invent a score. Propagate:

- Verification status from job-verification
- Freshness from freshness-verification
- Conflicts from contradiction-analysis
- Match breakdown from matching agents (technical / experience / domain separately)
- `location_fit` and `relocation_target` from location-matching

If those agents disagree, keep the conflict. Location fit never changes verification status; a `VERIFIED` job that is `REMOTE_RESTRICTED` stays in **Verified** with its location label shown.

## Research trace

Every canonical job accumulates a trace you append as stages complete:

```text
DISCOVERED
SOURCE IDENTIFIED
COMPANY IDENTIFIED
OFFICIAL SOURCE CHECKED
JOB VERIFIED
FRESHNESS CHECKED
DUPLICATES CHECKED
CONTRADICTIONS CHECKED
TECHNOLOGY ANALYZED
CANDIDATE MATCHED
LOCATION MATCHED
```

If a stage was skipped, record `SKIPPED` and why. The job report must include this trace.

## Output

After specialists finish:

1. Follow `agents/output/job-report.md` for each deep-verified job
2. Follow `agents/output/evidence-report.md` when the user asks for a claim-level pack, and for every `VERIFIED` or `CONFLICTED` job
3. Follow `agents/output/research-summary.md` once per run

Write markdown under `research/runs/{date}/` when you have write tools: `summary.md`, `jobs/{job_key}.md`, `evidence/{job_key}.md`. If you cannot write files, print the same documents in the conversation.

Organize the summary into: Verified, Needs verification, Stale/Closed, Conflicted. Every line carries `location_fit`; the summary also gives a location-fit breakdown and a per-region discovery tally. Do not hide uncertain or location-blocked jobs.

## Do not

- Discover and judge in the same step
- Trust a GitHub README as current
- Invent companies, jobs, salaries, stacks, or apply URLs
- Mark a job `VERIFIED` because it matches the candidate well
- Run all research yourself "to save time"
- Filter leads by location during discovery, or let a discovery agent do so
- Assume "Remote" means worldwide
- Spawn a specialist as a nested child of another specialist
- Silent-drop contradictions

## Examples

**User:** "Find senior Solidity and Node.js backend roles that are actually open."

1. Read the candidate profile (`location_policy`: remote worldwide; relocation NP/IN/US/AE/QA/AT/AU/EU)
2. Budget `standard`
3. Parallel: github-job-discovery, hiring-repository-discovery, hn-hiring-discovery, job-board-discovery × {global_remote, web3, europe, gulf, india, usa}
4. Normalize, stamp `lead_id`, first-pass dedupe, location pre-screen
5. Shortlist 12
6. Parallel company/official/github/tech/freshness per job
7. Verify, contradict, match (technical, experience, location)
8. Emit summary + per-job reports

**User:** "Find jobs anywhere in the world; I can relocate to Dubai, Vienna or Bengaluru."

Same as above; `run_overrides` adjusts `relocation_countries` for this run only unless asked to save. Add `austria` to the region list even on `standard`.

**User:** "Is Acme actually hiring a Java engineer?"

Skip broad discovery. Company pipeline only: company-job-discovery, company-research, github-company-research, job-verification, freshness, matching.
