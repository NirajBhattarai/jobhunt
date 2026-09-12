---
name: github-search
description: How to search GitHub for hiring repositories, issues, discussions, READMEs, and organizations. Use during GitHub job discovery or company GitHub intelligence. Do not treat GitHub posts as current.
---

# GitHub Search

Tactics for discovering hiring material on GitHub. This skill does not extract or verify jobs; it finds sources.

## Tools

Prefer, in order:

1. `web_search` with `site:github.com` queries
2. `open_page` / `web_fetch` on GitHub HTML and `raw.githubusercontent.com`
3. `open_page_with_find` when hunting a role title or company name inside a large README
4. GitHub search URLs you construct and then open:

```
https://github.com/search?q=QUERY&type=repositories
https://github.com/search?q=QUERY&type=issues
https://github.com/search?q=QUERY&type=discussions
https://github.com/search?q=QUERY&type=code
```

Code search often requires login. If the page asks to sign in, record `UNCERTAINTY` and continue with repositories, issues, and READMEs.

Do not scrape GitHub with a high request rate. Reuse pages already fetched in this run.

## Dynamic repository discovery

Do not stop at a memorized list. Each run must search, then follow new names that appear.

Seed queries (combine with candidate technologies from `agents/matching/candidate-profile.md`):

```
awesome jobs hiring
remote jobs
web3 jobs in:readme
blockchain jobs hiring
"we are hiring" in:readme
"who is hiring" github
solidity hiring
"java" "hiring" in:readme
"node.js" "we're hiring"
```

From each result, record: `owner/repo`, URL, description, last-updated signal if visible, why it looks like a hiring source.

Then search **from the page itself**: "Related", linked awesome-lists, and other repos mentioned in the README. Cap follow-depth at 2 unless the orchestrator raises the budget.

## What to open inside a repo

| Path / surface | Why |
|----------------|-----|
| README | Most community job lists live here |
| `jobs.md`, `hiring.md`, `CAREERS.md`, `WHO_IS_HIRING.md` | Dedicated lists |
| Issues labeled `hiring`, `job`, `jobs`, `career` | Issue-based postings |
| Discussions (GitHub Discussions tab) | Org hiring threads |
| Recent commits on the listing file | Freshness signal — pass to freshness-analysis |
| Org profile `https://github.com/{org}` | Website field, pinned repos |

## Issue and discussion patterns

Titles worth extracting (pass to job discovery, do not verify here):

- `[Hiring] …`
- `We're hiring …`
- `Job: …`
- `Open position: …`

Ignore issues that are people *seeking* jobs ("hire me", "looking for work") unless the user asked for that.

## Last-updated signals (observe, do not invent)

Valid: `updated` timestamp on the repo page, latest commit datetime on the file, a date written in the README ("Updated on 2026-04-16").

Invalid: guessing from star count, issue number, or "this repo is popular so it must be fresh".

## Hard rule

A GitHub hit is a **lead**. Classification of the job as `ACTIVE` is the freshness and verification agents' job, never this skill's.
