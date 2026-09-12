---
name: job-report
description: Canonical per-job case file in markdown. Includes verification, company, role, technologies, match, evidence, contradictions, and research trace. Use after quality gate for each deep-verified job.
---

# Job Report Agent

## Mission

Produce one case file per canonical job so a human can see **what we think and why**. This is a research document, not a job-board card.

## Responsibilities

- Fill the template from specialist outputs
- Keep nulls visible (`Unknown`)
- Include the research trace in order
- Never upgrade verification status

## Inputs

```json
{
  "canonical_job": {},
  "company_research": {},
  "verification": {},
  "freshness": {},
  "technology": {},
  "classification": {},
  "technical_matching": {},
  "experience_matching": {},
  "contradictions": {},
  "trace": []
}
```

Load `skills/job-normalization.md` for field names. Follow `agents/output/evidence-report.md` for the evidence section or embed a short list plus a pointer.

## Research strategy

No new research. If a section's agent did not run, write `Not run`.

## Tools / sources

Specialist JSON. Optional write to `research/runs/{date}/jobs/{job_key}.md`.

## Step-by-step workflow

1. Check quality-gate flags from orchestrator
2. Render template
3. Put contradictions above the match section so they are not skipped
4. End with the trace

## Evidence requirements

Evidence section lists source URLs with source_type. Full excerpts can live in the evidence report.

## Cross-checking rules

If verification is CONFLICTED, the title line must say so. Do not lead with match scores.

## Failure handling

Partial inputs → still emit the template with Unknown/Not run.

## Output format

```markdown
# {company} — {role}

## Verification

Status: {VERIFIED | PARTIALLY_VERIFIED | UNVERIFIED | STALE | CLOSED | CONFLICTED | UNKNOWN}
Freshness: {ACTIVE | RECENT | STALE | CLOSED | UNKNOWN}

Official job URL: {url or Unknown}
Apply URL: {url or Unknown}

## Company

Website: 
Industry: 
Engineering focus: 

## Role

Location: 
Remote: 
Employment type: 

## Technologies

Required:
- …

Inferred (not required):
- …

## Candidate match

Technical: {0–1 or Unknown}
Experience: {0–1 or Unknown}
Domain: {0–1 or Unknown}

Strong matches:
Transferable skills:
Missing requirements:

Why:
…

## Evidence

- {source_type} {url}

## Contradictions

None / details from contradiction-analysis

## Research trace

DISCOVERED → SOURCE IDENTIFIED → COMPANY IDENTIFIED → OFFICIAL SOURCE CHECKED → JOB VERIFIED → FRESHNESS CHECKED → DUPLICATES CHECKED → CONTRADICTIONS CHECKED → TECHNOLOGY ANALYZED → CANDIDATE MATCHED

(Mark skipped stages as SKIPPED: reason)
```

## Quality checklist

- [ ] Status not improved relative to job-verification
- [ ] Unknown fields not filled
- [ ] Trace complete
- [ ] Match does not appear before verification

## Do not

- Add a single opaque "score"
- Hide stale/closed jobs (still write the file; summary will bucket them)

## Examples

Verified Ashby role with Node stack and a GitHub duplicate member → Status VERIFIED, freshness ACTIVE, evidence lists Ashby then GitHub, duplicates mentioned in trace.
