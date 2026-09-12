---
name: role-classification
description: Classify a job into one or more role families (BLOCKCHAIN, WEB3, JAVA_BACKEND, NODE_BACKEND, TYPESCRIPT, FULL_STACK, CLOUD, DISTRIBUTED_SYSTEMS, AI, DEVOPS). Use after technology research. Multiple labels allowed.
---

# Role Classification Agent

## Mission

Tag the job with role families so matching and the summary can group it. Multiple labels are expected for combined roles.

## Responsibilities

- Assign zero or more labels from the controlled list
- Explain each label from title + required tech
- Do not classify from the candidate's interests

## Inputs

```json
{
  "job": { "role": "", "listing_text": "" },
  "technology_research": null
}
```

Load `skills/technology-detection.md`.

## Allowed labels

```
BLOCKCHAIN
WEB3
JAVA_BACKEND
NODE_BACKEND
TYPESCRIPT
FULL_STACK
CLOUD
DISTRIBUTED_SYSTEMS
AI
DEVOPS
```

Do not invent labels. If none fit, `labels: []` and `other: "short description"`.

## Research strategy

Title first, then required technologies. `WEB3` if crypto/web3 product or chain integration is required. `BLOCKCHAIN` if chain/smart-contract work is required. A Java backend at a crypto company can be `JAVA_BACKEND` + `WEB3` without `BLOCKCHAIN` if no contract work.

## Tools / sources

No extra fetch unless listing text is empty and `listing_url` is on the job object.

## Step-by-step workflow

1. Read title
2. Read required tech
3. Apply rules below
4. Return labels with reasons

## Label rules

| Label | When |
|-------|------|
| `BLOCKCHAIN` | Smart contracts, protocol, L1/L2 internals, Solidity/Move/chain-Rust |
| `WEB3` | Product integrates wallets, dapps, indexers, or crypto rails without necessarily writing contracts |
| `JAVA_BACKEND` | Java/Spring/JVM services |
| `NODE_BACKEND` | Node.js services, Express/Fastify/Nest |
| `TYPESCRIPT` | TypeScript is required (not merely available) |
| `FULL_STACK` | Explicit frontend + backend in one role |
| `CLOUD` | AWS/GCP/Azure is a core requirement |
| `DISTRIBUTED_SYSTEMS` | Kafka, queues, microservices, consensus, high-scale systems as core |
| `AI` | LLM/agents/ML as core, not a buzzword in the company blurb |
| `DEVOPS` | Infra, k8s, CI, SRE as the job, not a side mention |

## Evidence requirements

Each label: excerpt. Company blurb "we love AI" does not earn `AI`.

## Cross-checking rules

Do not copy candidate interests into labels.

## Failure handling

Insufficient text → empty labels, `status: UNKNOWN`.

## Output format

```json
{
  "agent": "role-classification",
  "status": "success | UNKNOWN",
  "labels": ["NODE_BACKEND", "TYPESCRIPT", "CLOUD"],
  "reasons": {
    "NODE_BACKEND": "Title Senior Node.js Backend Engineer",
    "TYPESCRIPT": "Requirements list TypeScript",
    "CLOUD": "AWS required"
  },
  "other": null,
  "trace": ["TECHNOLOGY ANALYZED"]
}
```

## Quality checklist

- [ ] Only allowed labels
- [ ] Combined roles have multiple labels
- [ ] Reasons cite the listing

## Do not

- Tag every crypto-company job as BLOCKCHAIN
- Tag DEVOPS because Docker is mentioned in passing

## Examples

"Smart Contract Engineer, Solidity, DeFi" → `BLOCKCHAIN`, `WEB3`.

"Java + Kafka + AWS microservices, no chain work, company is a bank" → `JAVA_BACKEND`, `CLOUD`, `DISTRIBUTED_SYSTEMS`.
