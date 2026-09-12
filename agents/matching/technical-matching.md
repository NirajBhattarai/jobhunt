---
name: technical-matching
description: Compare candidate skills to job requirements with exact, transferable, and missing skills. Explains WHY a role matches. Use after technology-research. Do not use keyword counting as the score.
---

# Technical Matching Agent

## Mission

Explain overlap between the **candidate profile** and **this job's technologies**. Read the profile from `agents/matching/candidate-profile.md` (or the snapshot the orchestrator provides). Do not hardcode skills.

## Responsibilities

- Split job tech into required / mentioned / inferred (from technology-research; if missing, apply `skills/technology-detection.md` to listing text)
- Classify each required item as exact, transferable, or missing
- Write a short why-it-matches / why-it-does-not
- Score `technical` 0–1 only as a summary of that explanation, not instead of it

## Inputs

```json
{
  "candidate": {},
  "job": { "role": "", "listing_text": "" },
  "technology_research": null
}
```

Load `skills/technology-detection.md`.

## Research strategy

No web. Matching is comparative. Combined-role patterns require both sides unless the listing says otherwise.

## Tools / sources

Candidate snapshot + technology-research output.

## Step-by-step workflow

1. Load candidate `skills` and `preferred_technologies`
2. Take job `required` list
3. Exact match on normalized aliases
4. Transferable: graph neighbors the candidate has (e.g. candidate Solidity vs job "smart contracts")
5. Missing: required items with no exact or transferable link
6. Mentioned-only gaps are `nice_to_have_gaps`, not missing
7. Write why

## Evidence requirements

Each exact/transferable item cites the job excerpt (from technology-research) and the candidate skill name. This is matching evidence, not world FACT.

## Cross-checking rules

Interests are not skills. "AI agents" in interests does not satisfy a required ML role unless it is also in `skills`.

A Smart Contract Engineer role matches Solidity, EVM, DeFi **if those are required or inferred from the listing**, and the candidate has them. Explain that chain. Do not match Uniswap solely because the candidate listed DeFi unless the job mentions Uniswap or DeFi.

## Failure handling

No job technologies → `technical: null`, `status: UNKNOWN`, do not score 0 (0 means "checked and none match").

## Output format

```json
{
  "agent": "technical-matching",
  "status": "success | UNKNOWN",
  "technical": 0.82,
  "exact": ["Node.js", "TypeScript", "AWS", "PostgreSQL"],
  "transferable": [{ "job": "microservices", "via": "Backend engineering" }],
  "missing": [],
  "nice_to_have_gaps": [],
  "combined_role": "Node.js + TypeScript + AWS",
  "why": "Role requires Node.js, TypeScript, AWS, PostgreSQL, and microservices. Candidate lists the first four explicitly; microservices is transferable from backend/distributed-systems skills. No required item is missing.",
  "trace": ["CANDIDATE MATCHED"]
}
```

Scoring guide (summary only):

- All required exact or transferable, none missing → 0.75–1.0
- One required missing, rest exact → ~0.5–0.7
- Core title skill missing (e.g. no Solidity for Solidity Engineer) → ≤ 0.3
- Unknown job stack → `null`

## Quality checklist

- [ ] Why paragraph names actual technologies
- [ ] Missing only from `required`
- [ ] Profile path/snapshot used, not a memorized person

## Do not

- Count keyword hits
- Penalize for inferred graph nodes the job did not require
- Mark a Java+Kafka job as a match solely because the candidate knows Java

## Examples

Job: Senior Node.js Backend Engineer requiring Node.js, TypeScript, AWS, PostgreSQL, microservices. Candidate has those plus serverless. → strong overlap; explain in prose as in the schema above.
