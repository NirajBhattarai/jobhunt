# Nepali Job Scraper Pipeline - Agents Guide

## Overview

This pipeline scrapes job listings from all major Nepali job portals, stores them in a local database, and provides a unified view of all available jobs and companies in Nepal.

## Target Job Portals

| Portal | URL | Method | Notes |
|--------|-----|--------|-------|
| Merojob | merojob.com | HTML/API | Largest portal, est. 2009 |
| KumariJob | kumarijob.com | HTML | Recruitment + job board |
| JobsNepal | jobsnepal.com | HTML | NGO/INGO focused, est. 2002 |
| NecoJobs | necojobs.com.np | HTML | Growing portal, est. 2019 |
| JobAxle | jobaxle.com | API (JSON) | Next.js app, clean API |
| Froxjob | froxjob.com | HTML | Company-grouped listings |
| RamroJob | ramrojob.com | HTML | Private sector focus |
| MeroRojgari | merorojgari.com | HTML | Banking/NGO/Govt jobs |
| KantipurJob | kantipurjob.com | HTML | Kantipur media group |
| Jobejee | jobejee.com | HTML | General job board |

## Project Structure

```
jobhunt/
├── agents.md                    # This file
├── scrapers/
│   ├── __init__.py
│   ├── base_scraper.py          # Abstract base with retry, rate-limit
│   ├── merojob_scraper.py
│   ├── kumariscraper.py
│   ├── jobsnepal_scraper.py
│   ├── necojobs_scraper.py
│   ├── jobaxle_scraper.py
│   ├── froxjob_scraper.py
│   ├── ramrojob_scraper.py
│   ├── merorojgari_scraper.py
│   ├── kantipurjob_scraper.py
│   └── jobejee_scraper.py
├── config/
│   ├── settings.py              # All portal configs, selectors, URLs
│   └── portal_configs.json      # Per-portal config (rate limits, etc.)
├── database/
│   ├── __init__.py
│   ├── models.py                # SQLAlchemy models
│   └── db_manager.py            # CRUD operations, dedup logic
├── processing/
│   ├── __init__.py
│   ├── cleaner.py               # Data cleaning, normalization
│   ├── skills_extractor.py      # Extract skills from job descriptions
│   └── deduplicator.py          # Cross-portal dedup
├── data/
│   ├── raw/                     # Raw JSON per portal per run
│   ├── processed/               # Cleaned data
│   └── exports/                 # CSV/JSON exports
├── analytics/
│   ├── __init__.py
│   └── reports.py               # Job market analytics
├── scheduler/
│   ├── __init__.py
│   └── auto_scraper.py          # ETL orchestration
├── pipeline.py                  # Main pipeline runner
├── requirements.txt
└── README.md
```

## Pipeline Stages

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   SCRAPE    │ -> │   CLEAN     │ -> │   ENRICH    │ -> │   STORE     │
│  (10 sites) │    │ (dedup,     │    │ (skills,    │    │ (SQLite,    │
│  parallel   │    │  normalize) │    │  company DB)│    │  export)    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### Stage 1: Scrape
- Run all 10 scrapers (parallel where possible)
- Respect robots.txt and rate limits (1-2 req/sec per portal)
- Save raw JSON to `data/raw/{portal}_{date}.json`
- Retry failed requests 3x with exponential backoff

### Stage 2: Clean
- Remove duplicates within each portal
- Normalize job titles, locations, salary formats
- Parse date strings to ISO format
- Handle missing/null fields gracefully

### Stage 3: Enrich
- Extract skills from job descriptions (regex + keyword matching)
- Tag job categories (IT, Finance, Education, etc.)
- Cross-portal deduplication using fuzzy title+company matching
- Attach company metadata (industry, size if available)

### Stage 4: Store
- Insert into SQLite database with conflict handling
- Generate export files (CSV, JSON)
- Update analytics/reports

## Database Schema

```sql
CREATE TABLE companies (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL,
    industry TEXT,
    location TEXT,
    website TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE jobs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    portal TEXT NOT NULL,
    external_id TEXT,
    title TEXT NOT NULL,
    company_id INTEGER REFERENCES companies(id),
    location TEXT,
    salary_min INTEGER,
    salary_max INTEGER,
    salary_currency TEXT DEFAULT 'NPR',
    job_type TEXT,
    experience_level TEXT,
    description TEXT,
    requirements TEXT,
    deadline DATE,
    url TEXT,
    posted_date DATE,
    scraped_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(portal, external_id)
);

CREATE TABLE skills (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL
);

CREATE TABLE job_skills (
    job_id INTEGER REFERENCES jobs(id),
    skill_id INTEGER REFERENCES skills(id),
    PRIMARY KEY (job_id, skill_id)
);
```

## Scraper Interface

Each scraper must implement:

```python
from abc import ABC, abstractmethod

class BaseScraper(ABC):
    def __init__(self, portal_name: str, base_url: str, rate_limit: float = 1.0):
        self.portal_name = portal_name
        self.base_url = base_url
        self.rate_limit = rate_limit  # seconds between requests
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'NepalJobScraper/1.0 (Research Purpose)'
        })

    @abstractmethod
    def scrape_jobs(self) -> list[dict]:
        """Scrape job listings. Returns list of standardized job dicts."""
        pass

    @abstractmethod
    def scrape_job_detail(self, url: str) -> dict:
        """Scrape full details for a single job listing."""
        pass

    def standardize(self, raw_job: dict) -> dict:
        """Convert raw scraped data to standard format."""
        return {
            'portal': self.portal_name,
            'external_id': raw_job.get('id', ''),
            'title': raw_job.get('title', '').strip(),
            'company': raw_job.get('company', '').strip(),
            'location': raw_job.get('location', '').strip(),
            'salary_min': raw_job.get('salary_min'),
            'salary_max': raw_job.get('salary_max'),
            'job_type': raw_job.get('job_type', 'Full-time'),
            'description': raw_job.get('description', ''),
            'url': raw_job.get('url', ''),
            'posted_date': raw_job.get('posted_date'),
            'deadline': raw_job.get('deadline'),
        }
```

## Running the Pipeline

```bash
# Install dependencies
pip install -r requirements.txt

# Run full pipeline
python pipeline.py

# Run specific portal
python pipeline.py --portal merojob

# Run specific stage
python pipeline.py --stage scrape
python pipeline.py --stage clean
python pipeline.py --stage enrich
python pipeline.py --stage store

# Export results
python pipeline.py --export csv
python pipeline.py --export json

# View stats
python pipeline.py --stats
```

## Scheduling (Daily)

```bash
# Add to crontab for daily 6 AM run
0 6 * * * cd /path/to/jobhunt && python pipeline.py >> logs/scraper.log 2>&1
```

## Ethical Scraping Rules

1. **Rate limiting**: Minimum 1 second between requests per portal
2. **Robots.txt**: Always check and respect before scraping
3. **User-Agent**: Identify as research scraper, not impersonate browsers
4. **No login bypass**: Only scrape publicly available listings
5. **Data usage**: Store only job-related data, no personal info
6. **Cache**: Re-scrape only if data is >24 hours old

## Dependencies

```
requests>=2.31.0
beautifulsoup4>=4.12.0
lxml>=5.0.0
sqlalchemy>=2.0.0
schedule>=1.2.0
python-dateutil>=2.8.0
rapidfuzz>=3.0.0
tabulate>=0.9.0
```

## Output Example

```
=== Nepal Job Scraper Results ===
Date: 2026-08-28

Portal Stats:
  Merojob:      156 jobs
  KumariJob:     89 jobs
  JobsNepal:     45 jobs
  NecoJobs:      67 jobs
  JobAxle:       34 jobs
  Froxjob:       23 jobs
  RamroJob:      56 jobs
  MeroRojgari:   78 jobs
  KantipurJob:   41 jobs
  Jobejee:       29 jobs

Total: 618 jobs from 10 portals
Unique Companies: 287
Top Skills: Python, JavaScript, SQL, React, Marketing
Top Locations: Kathmandu, Pokhara, Chitwan

Exported to: data/exports/jobs_2026-08-28.csv
```
