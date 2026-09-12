---
name: web-search
description: Web research tactics for company sites, careers pages, ATS boards, and job boards. Use when finding official URLs or corroborating a GitHub lead. Fetch pages; do not rely on snippets alone.
---

# Web Search

How to find and open pages. Snippets from a search index are pointers, not evidence. Evidence requires a fetch (`open_page`, `web_fetch`, or `open_page_with_find`).

## Query construction

Build queries from observed names, not from invented domains.

```
"{company}" careers
"{company}" jobs {role}
"{company}" greenhouse OR lever OR ashby
"{role}" "{company}" site:boards.greenhouse.io
"{company}" engineering blog
```

If the candidate profile names locations, add them only as extra queries, not as filters that drop remote roles.

## Careers page discovery

Try, in order, only after you have a real domain (see `skills/source-validation.md`):

```
https://{domain}/careers
https://{domain}/jobs
https://{domain}/join
https://{domain}/hiring
https://{domain}/about/careers
```

If those 404, search `{company} careers` and open the result that sits on the same registered domain or on a known ATS host.

Do not loop dozens of path variants. Three to five official-path attempts plus one web search is enough; then `UNCERTAINTY`.

## ATS public JSON endpoints (prefer over JS-rendered careers pages)

Most ATS boards expose unauthenticated JSON. Once you have observed a company slug on an ATS URL, fetch the API instead of scraping HTML. The response is a `FACT` source of type `verified_job_infra`; record the API URL as `source_url`.

| ATS | Endpoint | Notes |
|-----|----------|-------|
| Greenhouse | `https://boards-api.greenhouse.io/v1/boards/{slug}/jobs?content=true` | `jobs[].title`, `location.name`, `absolute_url`, `updated_at`, `id` |
| Lever | `https://api.lever.co/v0/postings/{slug}?mode=json` | `text` (title), `categories.location`, `hostedUrl`, `createdAt` (epoch ms) |
| Ashby | `https://api.ashbyhq.com/posting-api/job-board/{slug}?includeCompensation=true` | `jobs[].title`, `location`, `jobUrl`, `publishedAt`, `isRemote` |
| Workable | `https://apply.workable.com/api/v1/widget/accounts/{slug}` | `jobs[].title`, `url`, `published_on` |
| SmartRecruiters | `https://api.smartrecruiters.com/v1/companies/{slug}/postings` | `content[].name`, `location.city`, `releasedDate`, `ref` |

Rules:

- The slug must be observed on a URL you fetched or copied from a listing. Never guess a slug from the company name.
- A 404 from the API means the slug is wrong or the board is gone — record `UNCERTAINTY`, not `CLOSED`, unless a specific job URL also 404s.
- A job present in the API with a `updated_at` / `publishedAt` field is a date signal for `skills/freshness-analysis.md`.
- Workday and iCIMS have no stable public JSON; fetch the HTML job page directly.

## Job-board public JSON / RSS feeds (prefer over board HTML)

Several boards publish machine-readable feeds. One fetch returns dozens of listings with dates and, on some boards, structured location and visa fields. Feed hits are still `established_job_board` leads; record the feed URL as `source_url` and the listing's own URL as `application_url`.

| Board | Feed | Useful fields |
|-------|------|---------------|
| Remotive | `https://remotive.com/api/remote-jobs?search={term}&limit=50` | `title`, `company_name`, `candidate_required_location`, `publication_date`, `url` |
| Arbeitnow (EU / DACH, relocation) | `https://www.arbeitnow.com/api/job-board-api?page={n}` | `title`, `company_name`, `location`, `remote`, **`visa_sponsorship`**, `created_at`, `url`, `tags` |
| Remote OK | `https://remoteok.com/api?tag={term}` | `position`, `company`, `location`, `date`, `url`, `tags` (first element is metadata; skip it) |
| Jobicy | `https://jobicy.com/api/v2/remote-jobs?count=50&tag={term}` | `jobTitle`, `companyName`, `jobGeo`, `pubDate`, `url` |
| Himalayas | `https://himalayas.app/jobs/api?limit=50&q={term}` | `title`, `companyName`, `locationRestrictions`, `pubDate`, `applicationLink` |
| We Work Remotely | `https://weworkremotely.com/categories/remote-programming-jobs.rss` | RSS `title` ("Company: Role"), `region`, `pubDate`, `link` |
| Hacker News Who is hiring | see `agents/discovery/hn-hiring-discovery.md` (Algolia API) | comment text, `created_at` |

Rules:

- Feeds change. If a URL 404s or returns HTML, record `UNCERTAINTY` for that board and fall back to its HTML search page. Do not fabricate fields the feed did not return.
- Run at most 2–3 `{term}` values per feed (e.g. `solidity`, `node`, `java`); dedupe by `url`.
- `candidate_required_location` / `locationRestrictions` / `jobGeo` / `region` are **remote-scope** signals — copy them verbatim into the lead's `location` so location-matching can classify them.
- web3.career, cryptojobslist and cryptocurrencyjobs.co have no stable public feed; fetch their HTML listing pages.

## Regional board families

Choose by the candidate's `location_policy`. Each family is one `region` value for `job-board-discovery`.

| `region` | Boards (HTML unless a feed exists above) | Notes |
|----------|------------------------------------------|-------|
| `global_remote` | Remotive, Remote OK, Jobicy, Himalayas, We Work Remotely, Working Nomads, HN | Always run |
| `web3` | web3.career, cryptojobslist.com, cryptocurrencyjobs.co, remote3.co | Always run for a blockchain candidate |
| `europe` | Arbeitnow, EU-Remote-Jobs, Landing.jobs, Relocate.me (visa-sponsoring roles), StepStone | `*EU` relocation |
| `austria` | karriere.at, devjobs.at, StepStone.at, Arbeitnow (Vienna/Graz) | |
| `gulf` | Bayt, GulfTalent, LinkedIn public search `Dubai` / `Doha`, naukrigulf | UAE, Qatar |
| `india` | Instahyre, Cutshort, Wellfound (India filter), Naukri public pages, hirist.tech | Public listing pages only; skip login-gated detail |
| `usa` | Built In, Wellfound, Otta / Welcome to the Jungle, LinkedIn public, HN | Most "Remote (US)" leads will be `REMOTE_RESTRICTED`; still record them |
| `australia` | Seek, LinkedIn public `Sydney` / `Melbourne` | |
| `nepal` | Merojob, KumariJob, JobsNepal, Jobaxle, Froxjob (see `docs/nepal-portal-scraper-spec.md`) | Small volume; cap 3 pages |

## Search-snippet policy

A search snippet that says "Senior Engineer — Acme — Apply" is **not** proof the job is open. Open the URL. If the live page 404s, the snippet is stale.

## Rate and robots

Identify as a research agent. One request per host at a time when possible. Skip login-gated content; do not bypass auth. If a page is blocked, record the URL and status as `UNCERTAINTY`.

## Job boards

Third-party boards are discovery surfaces. After extracting a listing, the next required step (orchestrator) is official verification. Do not mark a board listing `VERIFIED`.
