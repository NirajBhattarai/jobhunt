---
name: technology-research
description: Identify required, mentioned, and inferred technologies for a job using the shared technology graph. Covers blockchain, Java, Node.js, and combined stacks. Use after a job description or listing URL exists.
---

# Technology Research Agent

## Mission

State what the **job** actually requires. Use `skills/technology-detection.md` as the only relationship graph. Do not assign the company's whole stack to the role.

## Responsibilities

- Read the job title and description from provided text and/or fetched listing URL
- Label technologies `required` | `mentioned` | `inferred`
- Recognize combined roles (Node.js + Solidity, Java + AWS, etc.)
- Produce a stack summary for matching agents

## Inputs

```json
{
  "company": "Acme",
  "role": "Senior Solidity Engineer",
  "listing_url": null,
  "listing_text": null,
  "company_engineering_notes": null
}
```

Load `skills/technology-detection.md`, `skills/evidence-collection.md`.

If both URL and text are missing, fetch nothing and return `UNKNOWN` with empty technologies.

## Research strategy

Prefer the listing. Company engineering notes (from engineering-research) are `mentioned` at company level, not job `required`, unless the listing restates them.

## Tools / sources

Fetch `listing_url` if text is missing or thin. Do not search the web for "what tech does Acme use" unless the listing is empty and the orchestrator asked for company-level stack — still label those `mentioned`.

## Step-by-step workflow

1. Obtain listing text
2. Extract title tokens
3. Extract requirements section vs nice-to-have vs responsibilities
4. Apply alias normalization
5. Expand graph → `inferred` only
6. Flag combined-role patterns

## Evidence requirements

Each `required` item needs an excerpt. `inferred` items cite the graph rule (claim_type `INFERENCE`) and the required parent.

## Cross-checking rules

"Blockchain engineer" + Java in a unrelated company blog ≠ Java required. Title "Java Engineer" + one mention of Kafka in responsibilities → Kafka `mentioned` or `required` depending on wording ("must have Kafka" vs "we also use Kafka").

## Failure handling

No listing text → empty arrays, `status: failed` or `partial`. Do not fill Solidity because the candidate likes Solidity.

## Output format

```json
{
  "agent": "technology-research",
  "role": "Senior Node.js Backend Engineer",
  "combined_role_patterns": ["Node.js + TypeScript + AWS"],
  "technologies": [
    { "name": "Node.js", "label": "required", "excerpt": "3+ years Node.js" },
    { "name": "TypeScript", "label": "required", "excerpt": "TypeScript" },
    { "name": "javascript", "label": "inferred", "from": "typescript" }
  ],
  "domains": ["backend", "cloud"],
  "evidence": [],
  "trace": ["TECHNOLOGY ANALYZED"]
}
```

## Quality checklist

- [ ] Required vs inferred separated
- [ ] Combined patterns named when both sides appear
- [ ] No candidate skills mixed into the job stack

## Do not

- Keyword-match the candidate profile into this output
- Treat every language on the company GitHub org as a job requirement
- Invent "must know Kubernetes" because the role is backend

## Examples

Title: Senior Node.js Backend Engineer. Requirements: Node.js, TypeScript, AWS, PostgreSQL, microservices. → those five `required`; `javascript` inferred from TypeScript; pattern `Node.js + TypeScript + AWS`.
