# Copilot instructions for this repository

Purpose
- Provide concise, actionable guidance for AI-assisted sessions so future Copilot runs understand how to build, run, and reason about this repo.

---

1) Build, install, test, and lint commands

- Python venv and deps (project uses Playwright + scraping libs):
  - python -m venv venv
  - source venv/bin/activate
  - pip install -r autoalpha/requirements.txt
  - playwright install

- Run a single script (examples):
  - Capture FB payloads (interactive, persists storage_state):
    - python autoalpha/scripts/test_fb_marketplace.py --query "BMW" --zip 28202 --radius 150 --duration 20
    - Add --headless to run headless.
    - Storage state is persisted to `storage_state.json` and browser profile to `.playwright`.

  - Extract Facebook listings from saved JSON payloads (single run):
    - python autoalpha/scripts/extract_fb_listings.py --output autoalpha/data/fb_listings.csv

  - Scrape Cars.com search page (single run):
    - python autoalpha/scripts/extract_carsdotcom.py --zip 28202 --radius 150 --make BMW --model "" --page 1

- Tests & linters:
  - No test runner (pytest/nose) or linter (flake8/ruff) is present by default. Use the script-run commands above to exercise behavior.

---

2) High-level architecture (big picture)

- Top-level package: `autoalpha/` — a small proof-of-concept data-collection project.
  - autoalpha/scripts/ — executable scripts:
    - test_fb_marketplace.py — Uses Playwright to open FB Marketplace, intercept XHR/fetch JSON responses, and save them to `autoalpha/fb_payloads/`.
    - extract_fb_listings.py — Walks saved JSON payloads, finds candidate listing objects, normalizes fields, and writes `autoalpha/data/fb_listings.csv`.
    - extract_carsdotcom.py — Requests Cars.com search pages, parses HTML with BeautifulSoup, extracts listing cards, normalizes fields, and writes `autoalpha/data/carsdotcom.csv`.
  - fb_payloads/ — captured JSON payloads from Playwright (raw GraphQL / XHR responses).
  - data/ — normalized CSV outputs used for downstream analysis/notebooks.
  - notebooks/ — exploratory analysis (not required to run scripts).

- Data flow:
  1. test_fb_marketplace.py (browser capture) → JSON files in fb_payloads/
  2. extract_fb_listings.py (parser) → normalized CSV in data/
  3. extract_carsdotcom.py (scraper) → normalized CSV in data/

- Normalized schema (used by extractors):
  - source, listing_id, title, price, year, make, model, mileage, location, url, description, timestamp

---

3) Key conventions and repo-specific patterns

- Script root resolution: each script sets ROOT = Path(__file__).resolve().parents[1] and writes outputs relative to that ROOT (data/, fb_payloads/, .playwright, storage_state.json). When editing scripts, preserve this pattern.

- Filename sanitization and storage
  - test_fb_marketplace.py sanitizes captured URLs into filesystem-safe JSON filenames (SAFE_FILENAME_RE + truncation). The capture pipeline writes files as `{NNN}_{sanitized_url}.json` to `fb_payloads/`.
  - Playwright persistent context is used: user profile is stored under `.playwright/` and session cookies are saved to `storage_state.json` (persisted between runs). Review and handle these artifacts carefully (they may contain credentials/cookies); add to .gitignore if needed.

- Deduplication in extractors
  - extract_fb_listings.py deduplicates by key (listing_id, url) before writing CSV.
  - Field parsing helpers (parse_number/parse_price/parse_year_from_title) are intentionally permissive; normalized outputs may require extra cleaning downstream.

- Output CSVs
  - Scripts use csv.DictWriter with a fixed FIELD_NAMES order. Downstream code expects that schema and column order.

- Error handling
  - Payload ingestion silently skips invalid JSON files (JSONDecodeError). Extraction tolerates missing fields and uses fallbacks (title from description, year from title, etc.).

- External dependencies
  - See autoalpha/requirements.txt (playwright, pandas, beautifulsoup4, lxml, requests). Installing these is required to run scripts.

---

4) Files to check when troubleshooting
- autoalpha/scripts/test_fb_marketplace.py — capture logic and storage_state handling
- autoalpha/fb_payloads/ — raw captured payloads
- autoalpha/scripts/extract_fb_listings.py — JSON walking and normalization
- autoalpha/scripts/extract_carsdotcom.py — HTML parsing rules and CSS selectors

---

5) Existing docs and notes incorporated
- This file pulls essential instructions from the project note ` carfinder.md` (project goal, script list, unified schema) and from `autoalpha/requirements.txt`.

---

If updates are made to script CLI flags or output paths, update this file accordingly so Copilot sessions remain accurate.
