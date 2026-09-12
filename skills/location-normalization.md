---
name: location-normalization
description: Rules for normalizing job locations, detecting remote-eligibility restrictions (hiring regions, time-zone windows, "US only"), and reading relocation / visa-sponsorship signals. Use when extracting location from a listing or judging location fit against the candidate's location policy.
---

# Location Normalization

Single source of truth for location fields. Discovery agents copy location text verbatim; the location-matching agent applies this file. Do not guess a country the listing does not state.

## Canonical location object

```json
{
  "raw": "Remote - EMEA or Dubai office",
  "work_mode": "remote | hybrid | onsite | flexible | null",
  "countries": ["AE"],
  "regions": ["EMEA"],
  "cities": ["Dubai"],
  "remote_scope": "worldwide | regions | countries | timezone | null",
  "remote_eligible_regions": ["EMEA"],
  "remote_eligible_countries": [],
  "timezone_window": null,
  "relocation_offered": "YES | NO | UNKNOWN",
  "visa_sponsorship": "YES | NO | UNKNOWN",
  "excerpts": ["Remote - EMEA or Dubai office"]
}
```

Country codes are ISO-3166 alpha-2. Unknown → `null` or empty array; never fill from the company's HQ unless the listing says the role is at HQ.

## Work mode detection

| Phrase | `work_mode` |
|--------|-------------|
| remote, fully remote, distributed, work from anywhere, WFH | `remote` |
| hybrid, N days in office, remote-friendly with office days | `hybrid` |
| onsite, on-site, in-office, relocation required, based in {city} with no remote wording | `onsite` |
| "remote or {city}", "remote / hybrid" | `flexible` |

"Remote-first company" in the company blurb does not make **this role** remote. Use the role's own location line.

## Remote scope (the part most agents get wrong)

`remote` is not one thing. Classify `remote_scope`:

| Listing wording | `remote_scope` | Fill |
|-----------------|----------------|------|
| anywhere, worldwide, globally remote, "we hire in any country" | `worldwide` | — |
| Remote (US), US-only, must be authorized to work in the US, Remote - Canada | `countries` | `remote_eligible_countries` |
| Remote - EMEA / Europe / LATAM / APAC / Americas | `regions` | `remote_eligible_regions` |
| within ±N hours of CET/EST/UTC, overlap with PST 9–1 | `timezone` | `timezone_window` |
| "Remote" with no qualifier | `null` | record `UNCERTAINTY: remote scope unstated` |

Also treat as restrictions: "must reside in", "eligible to work in", "no visa sponsorship", "we use an EOR in these countries: …", "contractor in {countries}".

## Region expansion

| Token | Expands to |
|-------|------------|
| `*EU` / Europe / EMEA (Europe part) | All European countries incl. UK, Switzerland, Norway, Balkans, Ukraine, Turkey (European side counted). EMEA additionally includes Middle East + Africa |
| Middle East / GCC / Gulf | AE, QA, SA, KW, BH, OM |
| South Asia | NP, IN, PK, BD, LK |
| APAC | AU, NZ, IN, SG, JP, and other Asia-Pacific incl. NP |
| Americas | US, CA, LATAM |
| DACH | DE, AT, CH |
| Nordics | SE, NO, DK, FI, IS |
| Benelux | BE, NL, LU |

City → country aliases: Dubai, Abu Dhabi → AE; Doha → QA; Kathmandu, Lalitpur, Pokhara → NP; Bangalore/Bengaluru, Hyderabad, Pune, Gurgaon/Gurugram, Noida, Mumbai, Chennai → IN; Vienna/Wien, Graz, Linz → AT; Sydney, Melbourne, Brisbane, Perth → AU.

## Time-zone window check

Candidate time zone comes from the profile (`Asia/Kathmandu`, UTC+5:45). Compute overlap honestly:

- CET (UTC+1/+2): offset 3:45–4:45h → most "±4h CET" windows pass; "±3h CET" fails by 45 min
- GST (UAE, UTC+4): 1:45h → passes almost any Gulf window
- IST (UTC+5:30): 0:15h → passes
- AEST (UTC+10): 4:15h → passes "±4h" barely, fails "±3h"
- US Eastern (UTC-4/-5): 9:45–10:45h → fails any "core overlap" requirement unless the listing says async-friendly
- US Pacific (UTC-7/-8): 12:45–13:45h → fails

If relocation to a listed country would satisfy the window, mark `RELOCATION` (see location-matching), not `INELIGIBLE`.

## Relocation and visa signals

`relocation_offered: YES` only from explicit text ("relocation package", "we support relocation", "visa and relocation assistance"). `visa_sponsorship: NO` only from explicit text ("no sponsorship", "must have existing right to work", "citizens/PR only"). Everything else `UNKNOWN`.

Boards with a structured sponsorship field (Arbeitnow `visa_sponsorship`, Relocate.me listings) count as `FACT` for that field with the API/listing URL as source.

## Prohibited

- Inferring "US only" from a US HQ, or "worldwide" from "remote"
- Treating "Remote (Europe)" as eligible for a candidate in Nepal without relocation
- Dropping a lead because location is unknown — that is `UNKNOWN`, reported, not filtered
