---
name: source-validation
description: Classify job and company URLs by authority. Distinguishes official careers pages, ATS infrastructure, GitHub, job boards, and social posts. Use when labeling source_type or judging verification strength.
---

# Source Validation

Single source of truth for source types and reliability. Agents label every URL with one `source_type` from this file.

## Source types (reliability order)

| `source_type` | Reliability | Examples |
|---------------|-------------|----------|
| `official_company` | 1.00 | `company.com/careers`, `company.com/jobs/123` |
| `verified_job_infra` | 0.92 | Greenhouse, Lever, Ashby, Workday, Workable, SmartRecruiters, iCIMS — company-owned ATS |
| `company_github` | 0.85 | GitHub org that matches the company's verified website |
| `engineering_blog` | 0.80 | Company engineering blog announcing a role or stack |
| `established_job_board` | 0.55 | LinkedIn, Wellfound, web3.career, Remote OK, Nepali portals in `agents.md` |
| `github_community` | 0.45 | awesome-jobs lists, hiring READMEs, community "who is hiring" repos |
| `social_community` | 0.25 | X/Twitter, Discord screenshots, HN comments, Reddit |

Reliability is a prior, not a verdict. A fresh official 404 beats a high-reliability guess that the job exists.

## Official vs third-party

**Official** — the company controls the URL: its domain, or an ATS board whose slug is the company (`boards.greenhouse.io/acme`, `jobs.lever.co/acme`, `jobs.ashbyhq.com/acme`).

**Third-party** — an aggregator or community restates the listing. Treat as a lead, never as verification.

A GitHub community README is third-party even when it links to an official page. Follow the link and classify the landing URL separately.

## ATS host patterns

Classify as `verified_job_infra` only when the path includes a company slug or job id you observed:

- `boards.greenhouse.io/{company}`
- `job-boards.greenhouse.io/{company}`
- `jobs.lever.co/{company}`
- `jobs.ashbyhq.com/{company}`
- `apply.workable.com/{company}`
- `{company}.wd1.myworkdayjobs.com` and other `myworkdayjobs.com` hosts
- `jobs.smartrecruiters.com/{company}`
- `{company}.icims.com`

Do not treat a generic ATS marketing homepage as a job listing.

## Domain resolution (do not guess)

A company domain is official only if obtained from one of:

1. The application URL host, when it is not an ATS or aggregator
2. The ATS company slug plus a website field on that ATS page
3. A GitHub organization's `blog`/`website` field you fetched
4. An explicit `https://...` link next to the company name in the listing
5. The company's page that you already opened and that clearly identifies itself

If none of these exist, `company_url` is `null` and `source_type` stays whatever you actually fetched. Do not invent `acme.com`.

## Spoof checks

Flag and lower confidence when:

- Domain is a lookalike (`acme-careers.com` vs `acme.com`)
- TLS/certificate mismatch or browser interstitial
- Application form posts to an unrelated host
- "Apply via WhatsApp/Telegram" for a supposedly well-known company
- Company name and ATS slug disagree with no explanation

Record these as `CONFLICT` or `UNCERTAINTY`, not as verified official sources.
