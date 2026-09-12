---
name: contradiction-analysis
description: How to record disagreements between agents or sources. Never silently resolve conflicts. Use when GitHub, careers pages, and job boards disagree about a role's existence, title, location, or status.
---

# Contradiction Analysis

Single source of truth for conflict records. The contradiction agent applies this file. Other agents may flag a possible conflict; they must not drop a source to hide it.

## What is a contradiction

Two claims about the same job or company that cannot both be true:

- Exists vs does not exist
- Open vs closed
- Title A vs title B (not a seniority synonym)
- Location remote vs a single on-site city with "no remote"
- Official domain A vs official domain B

Disagreement between "not found" and "found" **is** a contradiction if both agents searched. "Not found" vs "not searched" is `UNCERTAINTY`, not `CONFLICT`.

## Output object

```json
{
  "status": "CONFLICT",
  "subject": "Acme / Senior Solidity Engineer / existence",
  "claims": [
    {
      "agent": "github-job-discovery",
      "claim": "Job listed in awesome-web3-jobs README",
      "source_url": "https://github.com/example/repo",
      "claim_type": "FACT"
    },
    {
      "agent": "job-verification",
      "claim": "Official careers page has no matching role",
      "source_url": "https://acme.example/careers",
      "claim_type": "FACT"
    }
  ],
  "analysis": "Community list is a lead; official page does not currently show the role. Likely stale.",
  "resolution": "UNRESOLVED",
  "suggested_freshness": "STALE",
  "remaining_uncertainty": "The role may live on an ATS URL not linked from /careers.",
  "confidence": 0.82
}
```

## Resolution values

| Value | When |
|-------|------|
| `UNRESOLVED` | Default. Keep both claims |
| `LEAD_STALE` | Official absence plus old community date |
| `OFFICIAL_WINS` | Live official/ATS page contradicts a third-party |
| `THIRD_PARTY_ONLY` | No official page found at all |
| `POSSIBLE_SPOOF` | Company identity or apply URL looks fraudulent |

`resolution` is an `INFERENCE`. It does not delete the losing claim.

## Prohibited

- Picking the "nicer" story so the job can be recommended
- Averaging two statuses into `PARTIALLY_VERIFIED` without a conflict record
- Treating a failed fetch as proof of absence (that is `UNCERTAINTY` unless HTTP 404/410 was observed)
