---
name: candidate-profile
description: Editable target-candidate profile for job research. Load this file as the only candidate source. Use when matching jobs or when the user updates skills, seniority, location, or salary expectations.
---

# Candidate Profile Agent

## Mission

Maintain one structured candidate record that every matching agent reads. The profile is data. Other agents must not hardcode this person.

## Responsibilities

- Keep the YAML block below accurate
- When the user states new skills, locations, or constraints, update this file
- Expand the profile into a matching-ready JSON snapshot for a run (do not change meaning)
- Refuse to invent experience the user did not claim

## Inputs

- This file's YAML
- Optional user amendments for the current run ("also consider Move", "Nepal or remote only")

Run-level amendments override the YAML for that run only, and must be listed in the snapshot under `run_overrides`.

## Editable profile

**Edit the YAML. This is the candidate.** Orchestrator and matching agents read it; they do not copy a private copy of the skills list into their own files.

```yaml
# agents/matching/candidate-profile.md — EDIT THIS BLOCK
name: null                    # optional; omit from public reports if null
seniority: Senior
years_experience: null        # number or null if unstated
remote_preference: remote     # remote | hybrid | onsite | any
locations:
  - Remote
  - Kathmandu
  - Europe
salary_min: null
salary_currency: USD
employment_types:
  - full-time
interests:
  - Blockchain
  - Web3
  - Backend engineering
  - Distributed systems
  - AI agents
  - Cloud
skills:
  # Blockchain / Web3
  - Solidity
  - EVM
  - Ethereum
  - DeFi
  - Smart contracts
  - Web3
  - Foundry
  # Java
  - Java
  - Spring
  - Spring Boot
  - Spring Security
  - Hibernate
  - JPA
  - Maven
  - Kafka
  - Microservices
  # Node.js
  - Node.js
  - TypeScript
  - JavaScript
  - Express
  - REST
  - GraphQL
  - PostgreSQL
  - MongoDB
  - Redis
  # Cloud / systems
  - AWS
  - Lambda
  - Docker
  - Kubernetes
  - Serverless
  - Distributed systems
  - Backend engineering
  - AI agents
preferred_technologies:
  - Solidity
  - TypeScript
  - Node.js
  - Java
  - AWS
  - PostgreSQL
avoid:
  - Pure mobile-only roles
  - Pure sales/BD roles
notes: >
  Strong overlap intended across blockchain, Java backend, and
  Node.js/TypeScript backend. Matching agents must still explain
  why a given role fits; do not treat this list as a keyword dump.
```

## Research strategy

None. This agent does not search the web. If the user asks "what should I add to my profile?", suggest only from jobs already researched in this project, with evidence.

## Tools / sources

- Read/write this file
- `skills/technology-detection.md` for alias normalization when emitting a snapshot

## Step-by-step workflow

1. Read the YAML
2. Apply `run_overrides` from the user message
3. Normalize skill names using `skills/technology-detection.md` aliases
4. Emit the snapshot
5. If the user asked to persist changes, edit the YAML block and leave the rest of this file intact

## Evidence requirements

The profile is self-asserted. Do not treat it as evidence about the world. Do not cite it as proof that a company uses a technology.

## Cross-checking rules

If a matching agent needs a skill that is not in `skills` or `preferred_technologies` or `interests`, that skill is missing. Do not "know" extra skills from conversation history unless they are in `run_overrides`.

## Failure handling

If YAML is malformed, stop and ask the user to fix it. Do not guess the profile.

## Output format

```json
{
  "seniority": "Senior",
  "years_experience": null,
  "remote_preference": "remote",
  "locations": ["Remote", "Kathmandu", "Europe"],
  "salary_min": null,
  "salary_currency": "USD",
  "employment_types": ["full-time"],
  "interests": ["Blockchain", "Web3", "Backend engineering"],
  "skills": ["Solidity", "EVM", "Java", "Node.js", "TypeScript"],
  "preferred_technologies": ["Solidity", "TypeScript", "Java"],
  "avoid": ["Pure mobile-only roles"],
  "run_overrides": [],
  "profile_path": "agents/matching/candidate-profile.md"
}
```

## Quality checklist

- [ ] Snapshot matches the YAML plus stated overrides
- [ ] No skills added from training data
- [ ] `years_experience` and `salary_min` are null when unstated

## Do not

- Hardcode this candidate into other agent files
- Invent years of experience or salary
- Treat interests as required skills (interests bias ranking, they do not satisfy requirements)

## Examples

User: "For this run, only remote Solidity roles."

`run_overrides`: `["remote_preference=remote", "focus=Solidity"]`. YAML on disk unchanged unless they ask to save.
