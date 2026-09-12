---
name: technology-detection
description: Technology relationship graph and extraction rules for job listings. Covers blockchain/EVM, Java/Spring, Node.js/TypeScript, cloud, and combined stacks. Use when identifying required vs mentioned technologies.
---

# Technology Detection

Single source of truth for stack relationships. Matching and research agents read this file instead of maintaining their own keyword lists.

## Required vs mentioned vs inferred

| Label | Meaning |
|-------|---------|
| `required` | Stated as must-have, in a requirements list, or in the title |
| `mentioned` | Appears in the description without being required |
| `inferred` | Follows from a required technology via the graph below. Always `INFERENCE` |

Do not promote `inferred` to `required`.

## Relationship graph

Each key implies the values as **related**, not as substitutes. Direction is "if you see X, also consider Y".

### Blockchain / Web3

- `solidity` → `evm`, `ethereum`, `smart contracts`, `foundry`, `hardhat`, `web3`
- `evm` → `ethereum`, `solidity`, `layer 2`
- `defi` → `evm`, `solidity`, `uniswap`, `aave`, `amm`
- `rust` + blockchain context → `solana`, `substrate`, `cosmos`, `systems`
- `move` → `sui`, `aptos`
- `layer 2` → `optimism`, `arbitrum`, `base`, `zk`

### Java backend

- `java` → `jvm`, `spring`, `maven` or `gradle`
- `spring boot` → `java`, `spring`, `spring security` (only if security is mentioned)
- `hibernate` / `jpa` → `java`, relational databases
- `kafka` → `distributed systems`, `event-driven`
- `microservices` → `distributed systems`, `api`

### Node.js / TypeScript

- `node.js` → `javascript`, `typescript` (typescript only if mentioned or title says TypeScript)
- `typescript` → `javascript`
- `express` / `fastify` / `nestjs` → `node.js`
- `graphql` → `api`
- `websockets` → `realtime`

### Data / cloud / runtime

- `postgresql` → `sql`, `relational`
- `mongodb` → `document store`
- `redis` → `cache`
- `aws` → `cloud` (not every AWS service)
- `lambda` → `aws`, `serverless`
- `docker` → `containers`
- `kubernetes` → `containers`, `orchestration`

## Combined roles

Treat these as intersection requirements, not a single keyword:

| Pattern | Both sides required unless the listing says otherwise |
|---------|------------------------------------------------------|
| Node.js + Solidity | Backend JS/TS and smart contracts |
| TypeScript + Web3 | TS application layer plus chain integration |
| Java + AWS | JVM services on AWS |
| Java + Kafka | JVM plus event streaming |
| Java + Blockchain | JVM backend; chain work only if also named |
| Node.js + AWS | Node services plus cloud |
| Node.js + Blockchain | Node backend plus chain integration |

A "Blockchain Engineer" title does **not** automatically require Java or Node. Read the listing.

## Extraction order

1. Title tokens (`Solidity`, `Java`, `Node.js`, `TypeScript`)
2. Requirements / qualifications section
3. Responsibilities
4. Company stack pages (engineering-research), labeled `mentioned` unless the job restates them
5. Graph expansion → `inferred`

Normalize aliases before comparing: `nodejs` / `Node` / `node.js` → `node.js`; `postgres` → `postgresql`; `k8s` → `kubernetes`; `smart contract` → `smart contracts`.
