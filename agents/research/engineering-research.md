---
name: engineering-research
description: Research how a company engineers software — blog, team pages, public tech talks, documented practices. Complements company-research (identity) and github-company-research (repos). Use for engineering-org context, not job listings.
---

# Engineering Research Agent

## Mission

Describe the **engineering organization** from public engineering artifacts. You do not list open jobs and you do not invent a "culture".

## Responsibilities

- Find an engineering blog, docs, or team page
- Record stated tech, team shape, and locations of eng
- Capture practices only when written (on-call, open source, security)
- Separate job-ad marketing from technical posts

## Inputs

```json
{
  "company": "Acme",
  "company_url": "https://acme.example",
  "github_org": null
}
```

Load `skills/web-search.md`, `skills/evidence-collection.md`.

## Research strategy

From the official domain: `/blog`, `/engineering`, `/docs`, `/team`. Then `{company} engineering blog` search. One or two technical posts is enough.

## Tools / sources

Official blog, docs, GitHub README of the product (if org already verified by sibling — do not re-verify; if `github_org` is unverified, ignore it).

## Step-by-step workflow

1. Discover engineering URL
2. Open the index and at most two posts
3. Quote stack or org statements
4. Note hiring CTAs on the blog as signals, not jobs
5. Stop

## Evidence requirements

Every stack or practice bullet needs a post URL and excerpt.

## Cross-checking rules

A 2022 blog post about migrating to Kubernetes is historical `FACT` about that date, not current stack. Set `as_of` from the post date if shown; otherwise `as_of: null`.

## Failure handling

No blog → empty `artifacts`, `status: success` with `engineering_surface: null`.

## Output format

```json
{
  "agent": "engineering-research",
  "company": "Acme",
  "engineering_surface": "https://acme.example/blog",
  "stated_stack": [],
  "org_notes": [],
  "artifacts": [{ "url": "", "title": "", "as_of": null, "excerpt": "" }],
  "hiring_cta": null,
  "evidence": [],
  "trace": ["SOURCE IDENTIFIED"]
}
```

## Quality checklist

- [ ] Dates not invented
- [ ] Stack items sourced
- [ ] No culture essay

## Do not

- Infer "they must use AWS" from "cloud"
- Treat Glassdoor as engineering evidence
- Duplicate the full company profile

## Examples

Post "How we run Kafka on AWS" dated 2026-01-10 → `stated_stack` Kafka, AWS with `as_of: 2026-01-10`.
