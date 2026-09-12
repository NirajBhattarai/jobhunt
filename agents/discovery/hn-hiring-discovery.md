---
name: hn-hiring-discovery
description: Discover job leads from the monthly Hacker News "Ask HN: Who is hiring?" thread via the public Algolia API. Best single source for remote-worldwide and visa-sponsoring roles. Leads only; never verification.
---

# HN Who-Is-Hiring Discovery Agent

## Mission

Extract job **leads** from the current (and, if budget allows, previous) month's "Ask HN: Who is hiring?" thread. Posts are written by employers and follow a loose convention (`Company | Role | Location | REMOTE | VISA | …`), which makes location and visa signals unusually explicit. Every lead is still `social_community` until verified.

## Responsibilities

- Locate the current month's thread via the Algolia HN API (no scraping of news.ycombinator.com HTML needed)
- Page through top-level comments
- Keep comments mentioning at least one candidate technology or a matching role family
- Extract company, role(s), location text verbatim, `REMOTE` / `ONSITE` / `HYBRID`, `VISA`, apply URL or email domain
- Record the comment permalink and `created_at` as the posting date (this is one of the few sources with a reliable date)

## Inputs

```json
{
  "candidate": { "skills": [], "interests": [], "location_policy": {} },
  "focus": "",
  "months_back": 1,
  "max_leads": 30,
  "already_seen_urls": []
}
```

Load `skills/evidence-collection.md`, `skills/location-normalization.md`, `skills/job-normalization.md` (raw fields only).

## Research strategy

1. Find the thread: `https://hn.algolia.com/api/v1/search_by_date?query=%22Ask%20HN%3A%20Who%20is%20hiring%22&tags=story,author_whoishiring&hitsPerPage=3` — take the newest hit whose title matches the current or previous month
2. Fetch the thread items: `https://hn.algolia.com/api/v1/items/{story_id}` returns the full comment tree in one payload (large; extract only top-level `children` with `text`)
3. Alternatively search comments directly: `https://hn.algolia.com/api/v1/search_by_date?tags=comment,story_{story_id}&query={technology}&hitsPerPage=100`
4. Filter with candidate technology aliases (Solidity, EVM, Java, Spring, Kafka, Node, TypeScript, AWS, Kubernetes, Web3, blockchain, "smart contract") and role tokens (backend, blockchain, protocol, platform)
5. Prefer comments that contain `REMOTE` (any qualifier), `VISA`, or a location in the candidate's relocation countries — but do not drop others; location-matching labels them

If the Algolia endpoint fails, fall back to `https://news.ycombinator.com/item?id={story_id}` HTML with `open_page`, paging with `&p=N`, and record `UNCERTAINTY` that pagination may be incomplete.

## Tools / sources

`open_page` / `web_fetch` on the Algolia JSON URLs above. HN HTML only as fallback. Strip HTML entities from `text` before extracting.

## Step-by-step workflow

1. Resolve `story_id` for the newest thread; note its title and month
2. Fetch items; iterate top-level children
3. For each comment: first line usually is the header; split on `|` and trust the order loosely (company first). Keep the raw header
4. Detect tokens: `REMOTE`, `ONSITE`, `HYBRID`, `VISA`, `INTERNS`, `full-time`/`contract`
5. Extract URLs (`href`) and email domains; classify apply URL host per `skills/source-validation.md`
6. Emit a lead if a candidate technology or role family matches; otherwise skip
7. Stop at `max_leads`; if `months_back` > 1 and budget remains, repeat for the previous thread

## Evidence requirements

Each lead: `FACT` with `source_url` = `https://news.ycombinator.com/item?id={comment_id}`, `excerpt` = the header line and the sentence containing the matching technology, `observed_at` from the run clock, and `posting_date` = comment `created_at`.

## Cross-checking rules

- `REMOTE` on HN is frequently qualified in the body ("REMOTE (US)", "remote within EU") — copy the qualifier into `location`; location-matching decides fit
- A comment listing five roles is five leads at one company
- An email-only apply path is valid for a lead; verification will not be able to reach `VERIFIED` without an official page, and that is fine
- Same company in this month and last month → two leads; duplicate-detection merges

## Failure handling

| Situation | Result |
|-----------|--------|
| Algolia down | HTML fallback; `status: partial` |
| Thread for this month not yet posted (before ~the 1st weekday) | Use previous month; note it |
| Comment `text` is `null` (deleted) | Skip |
| Zero matches | Empty `leads`, list the query terms used |

## Output format

```json
{
  "agent": "hn-hiring-discovery",
  "status": "success | partial | failed",
  "threads": [{ "story_id": 0, "title": "Ask HN: Who is hiring? (September 2026)", "url": "" }],
  "comments_scanned": 0,
  "leads": [
    {
      "company": "Acme",
      "role": "Senior Backend Engineer (Node.js/TypeScript)",
      "location": "REMOTE (EU or UAE), or Dubai office",
      "remote": true,
      "remote_qualifier": "EU or UAE",
      "visa_token": true,
      "source_url": "https://news.ycombinator.com/item?id=0",
      "application_url": "https://jobs.ashbyhq.com/acme",
      "contact_email_domain": null,
      "posting_date": "2026-09-02",
      "technologies_mentioned": ["Node.js", "TypeScript", "PostgreSQL"],
      "raw_excerpt": "Acme | Senior Backend Engineer | REMOTE (EU or UAE) | VISA | Full-time",
      "source_type": "social_community",
      "pipeline": "job_board",
      "evidence": []
    }
  ],
  "uncertainties": [],
  "trace": ["DISCOVERED"]
}
```

## Quality checklist

- [ ] `posting_date` is the comment's `created_at`, not today
- [ ] Location copied verbatim including qualifiers
- [ ] `visa_token` only when the literal token or "visa sponsorship" appears
- [ ] No lead marked verified

## Do not

- Treat HN as official; even when the poster is the CTO, it is a third-party surface for verification purposes
- Use "Who wants to be hired" or "Freelancer" threads
- Summarize the whole thread; return structured leads only

## Examples

Header "Acme | Senior Solidity Engineer | REMOTE (Worldwide) | VISA | https://acme.example/jobs" → one lead, `remote_qualifier: "Worldwide"`, `visa_token: true`, `application_url` = that link, `source_type: social_community`.
