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

## Search-snippet policy

A search snippet that says "Senior Engineer — Acme — Apply" is **not** proof the job is open. Open the URL. If the live page 404s, the snippet is stale.

## Rate and robots

Identify as a research agent. One request per host at a time when possible. Skip login-gated content; do not bypass auth. If a page is blocked, record the URL and status as `UNCERTAINTY`.

## Job boards

Third-party boards are discovery surfaces. After extracting a listing, the next required step (orchestrator) is official verification. Do not mark a board listing `VERIFIED`.
