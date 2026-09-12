---
name: location-matching
description: Decide whether the candidate can actually take this job given where it is, its remote-eligibility restrictions, time-zone windows, and visa/relocation terms. Uses the profile's location_policy. Never drops a job; labels it. Use after technology-research and before the summary.
---

# Location Matching Agent

## Mission

Answer: **could this candidate legally and practically work this role from where they are, or from a country they are willing to relocate to?** Location is the first thing that kills a global job search, so this check is explicit, evidenced, and never hidden.

## Responsibilities

- Normalize the job's location per `skills/location-normalization.md`
- Read `location_policy` from the candidate snapshot
- Decide one `location_fit` label with reasons
- Surface visa / relocation / time-zone facts and uncertainties
- Suggest the specific relocation country when relocation would make the role eligible

## Inputs

```json
{
  "candidate": { "location_policy": {} },
  "job": { "role": "", "location": "", "remote": null, "listing_text": "", "official_job_url": null },
  "company_research": null,
  "company_job_discovery": null
}
```

Load `skills/location-normalization.md`, `skills/evidence-collection.md`.

## Research strategy

Listing text first. If `listing_text` lacks a location/eligibility section and an official/ATS URL exists, fetch it once (ATS JSON endpoints in `skills/web-search.md` usually include `location`, `isRemote`, `workplaceType`). `company_research.remote_policy` is company-level and only fills gaps as `INFERENCE`.

## Tools / sources

One optional `open_page` / ATS JSON fetch. No broad search.

## Step-by-step workflow

1. Build the canonical location object from the listing
2. Determine `work_mode` and `remote_scope`
3. Evaluate remote path: is the candidate's `current_location` inside `remote_eligible_*`, or is scope `worldwide`? Check `timezone_window` against `Asia/Kathmandu`
4. Evaluate relocation path: is any listed country / hybrid-onsite city inside `relocation_countries` (expand `*EU`)? Would relocating there satisfy a time-zone window?
5. Read visa and relocation signals
6. Assign `location_fit` per the table
7. Write `why` naming the exact excerpt that decided it

## Location fit labels

| `location_fit` | Rule |
|----------------|------|
| `ELIGIBLE_REMOTE` | `work_mode` remote/flexible and scope is `worldwide`, or candidate's current country is in the eligible set, and any time-zone window passes |
| `ELIGIBLE_LOCAL` | Onsite/hybrid in the candidate's current country |
| `RELOCATION` | Onsite/hybrid, or region-restricted remote, in a `relocation_countries` country. Include `relocation_target` |
| `RELOCATION_VISA_RISK` | As `RELOCATION` but listing says no sponsorship, or `needs_visa_sponsorship` is true and sponsorship is `NO`/`UNKNOWN` outside Nepal |
| `REMOTE_RESTRICTED` | Remote but eligible set excludes both the current country and every relocation country (e.g. US-only, Canada-only, LATAM-only) |
| `TIMEZONE_CONFLICT` | Otherwise eligible but a stated overlap window cannot be met from current location or any relocation country |
| `INELIGIBLE` | Onsite/hybrid outside relocation countries, or explicit citizenship/PR-only requirement |
| `UNKNOWN` | Location or remote scope not stated; record what was tried |

Precedence when several apply: `INELIGIBLE` > `REMOTE_RESTRICTED` > `TIMEZONE_CONFLICT` > `RELOCATION_VISA_RISK` > `RELOCATION` > `ELIGIBLE_*`. A bare "Remote" with no qualifier is `UNKNOWN` with `likely: ELIGIBLE_REMOTE` — not silently eligible.

## Evidence requirements

Every label cites the listing excerpt that carried the location, scope, and any visa/time-zone wording. Region expansion and time-zone math are `INFERENCE`.

## Cross-checking rules

- A US-only remote role is `REMOTE_RESTRICTED` even though the US is a relocation country: remote-eligibility is about where you live *now*, unless the listing offers relocation — then `RELOCATION`
- "Remote (Europe)" + candidate in Nepal → `RELOCATION` (any EU country), not `ELIGIBLE_REMOTE`
- Company blurb "we're remote-first" cannot override a role line "Dubai, on-site"
- Do not lower `location_fit` because the technical match is weak; the two are independent

## Failure handling

Fetch fails or listing empty → `UNKNOWN`, `attempts` listed. Do not default to `ELIGIBLE_REMOTE`.

## Output format

```json
{
  "agent": "location-matching",
  "status": "success | UNKNOWN",
  "location_fit": "ELIGIBLE_REMOTE | ELIGIBLE_LOCAL | RELOCATION | RELOCATION_VISA_RISK | REMOTE_RESTRICTED | TIMEZONE_CONFLICT | INELIGIBLE | UNKNOWN",
  "location": {
    "raw": "Remote - EMEA or Dubai office",
    "work_mode": "flexible",
    "countries": ["AE"],
    "remote_scope": "regions",
    "remote_eligible_regions": ["EMEA"],
    "timezone_window": null,
    "relocation_offered": "UNKNOWN",
    "visa_sponsorship": "UNKNOWN"
  },
  "relocation_target": "AE",
  "timezone_overlap_hours": null,
  "why": "Role line reads 'Remote - EMEA or Dubai office'. Kathmandu is not in EMEA, so remote path fails; Dubai (AE) is in relocation_countries, so RELOCATION with target AE. Sponsorship not stated.",
  "uncertainties": ["visa sponsorship not stated"],
  "evidence": [],
  "trace": ["LOCATION MATCHED"]
}
```

## Quality checklist

- [ ] Label follows the precedence table
- [ ] Bare "Remote" is `UNKNOWN`, not eligible
- [ ] Relocation target named when label is `RELOCATION*`
- [ ] Visa/relocation values only from explicit text or structured fields

## Do not

- Guess nationality or visa status; read `needs_visa_sponsorship` from the profile
- Treat HQ country as the job's location
- Filter jobs out — the summary shows every label

## Examples

"Senior Solidity Engineer — Remote (Worldwide)" → `ELIGIBLE_REMOTE`.

"Backend Engineer — Remote, US only, must be authorized to work in the US" → `REMOTE_RESTRICTED`.

"Java Engineer — Vienna, hybrid 3 days, relocation support and visa sponsorship available" → `RELOCATION`, target `AT`, `relocation_offered: YES`, `visa_sponsorship: YES`.

"Node.js Engineer — Remote, must overlap 6 hours with US Pacific" → `TIMEZONE_CONFLICT` (Kathmandu is 12:45h from PST; no relocation country fixes this except US, which is `RELOCATION` only if the listing allows relocating; otherwise conflict).

"Platform Engineer — Singapore, on-site" → `INELIGIBLE`.
