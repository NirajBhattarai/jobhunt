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

## Search-snippet policy

A search snippet that says "Senior Engineer — Acme — Apply" is **not** proof the job is open. Open the URL. If the live page 404s, the snippet is stale.

## Rate and robots

Identify as a research agent. One request per host at a time when possible. Skip login-gated content; do not bypass auth. If a page is blocked, record the URL and status as `UNCERTAINTY`.

## Job boards

Third-party boards are discovery surfaces. After extracting a listing, the next required step (orchestrator) is official verification. Do not mark a board listing `VERIFIED`.
