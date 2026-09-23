# Strata

Strata is a threat-intelligence platform that aggregates Known Exploited Vulnerabilities (KEVs) and Indicators of Compromise (IOCs) by malware family. It gives blue teamers a single, continuously-updated view of the active threat landscape — from vulnerabilities being exploited right now to the malware families and infrastructure behind current attacks.

**[View the live dashboards →](https://blkwizard20z.github.io/strata/index.html)**

---

## Features

- **Two independent data feeds** — CISA Known Exploited Vulnerabilities and ThreatFox malware IOCs
- **KEV dashboard** — searchable table of actively-exploited vulnerabilities, each linked to its NVD detail page
- **ThreatFox dashboard** — malware IOCs grouped and ranked by family, so the most active threats surface first
- **Per-family deep-dives** — statistics, activity timelines, top destination ports, and sample indicators for each major family
- **Automatic enrichment** — family descriptions and threat-actor attribution pulled from Malpedia
- **Defanged indicators** — live malicious URLs, domains, and IPs are neutralized in the display so they can't be clicked by accident
- **Daily automation** — the entire pipeline (fetch → enrich → deep-dive → publish) runs itself via GitHub Actions

---

## How it works

Strata has no server and no database. Python scripts fetch data from public threat-intelligence APIs and write it to flat JSON files stored in the repository. GitHub Actions runs those scripts on a daily schedule, and GitHub Pages serves static HTML dashboards that read the JSON directly in the browser.

For the ThreatFox feed, IOCs are grouped by malware family and ranked by activity. Each family is then enriched with a description and known threat-actor attribution from Malpedia. To keep this efficient and respectful of a free service, enrichment results are cached — each family is only looked up once — and the daily automation run keeps everything current.

This "static + scheduled" design means the whole platform costs nothing to run, has no infrastructure to maintain, and uses the repository itself (with git history) as its versioned data store.

---

## Tech stack

**Built with:**
- Python (standard library + `certifi`)
- GitHub Actions (scheduled automation)
- GitHub Pages (static hosting)
- Vanilla HTML / CSS / JavaScript (no framework, no build step)

**Data sources:**
- [CISA Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [ThreatFox](https://threatfox.abuse.ch/) (abuse.ch)
- [Malpedia](https://malpedia.caad.fkie.fraunhofer.de/) (Fraunhofer FKIE)

---

## Project structure

```
strata/
├── .github/workflows/          # Automation (GitHub Actions)
│   ├── fetch-kev.yml           #   daily KEV job
│   └── fetch-threatfox.yml     #   daily ThreatFox job (fetch → enrich → deep-dive → publish)
│
├── scripts/                    # All the Python logic
│   ├── fetch_kev.py            #   pulls CISA KEV
│   ├── fetch_threatfox.py      #   pulls + groups + ranks ThreatFox IOCs
│   ├── enrich_families.py      #   adds Malpedia descriptions/attribution (cached)
│   └── fetch_family_details.py #   builds per-family deep-dive stats
│
├── data/                       # Canonical data archive (owned by automation)
│   ├── kev/
│   │   ├── latest.json         #   current snapshot the dashboard reads
│   │   └── YYYY-MM-DD.json     #   dated history (written only in CI)
│   ├── threatfox/
│   │   ├── latest.json
│   │   ├── YYYY-MM-DD.json
│   │   └── families/           #   per-family deep-dive profiles
│   │       ├── _index.json     #     list of families that have a profile
│   │       └── <malware_id>.json
│   └── enrichment/
│       └── families.json       #   Malpedia cache = the knowledge base
│
├── docs/                       # What GitHub Pages serves (the live site)
│   ├── index.html              #   KEV dashboard
│   ├── threatfox.html          #   ThreatFox family leaderboard
│   ├── family.html             #   per-family deep-dive page
│   └── data/                   #   published copies the live pages fetch
│       ├── kev/latest.json
│       └── threatfox/
│           ├── latest.json
│           └── families/*.json
│
├── .env                        # Local-only secrets (gitignored, never committed)
├── .gitignore
├── requirements.txt            # Python dependencies (certifi)
└── README.md
```

---

## Getting started

**Prerequisites:** Python 3.10 or newer, and a free ThreatFox Auth-Key (from [auth.abuse.ch](https://auth.abuse.ch/)).

```bash
# 1. Clone the repository
git clone https://github.com/BlkWizard20z/strata.git
cd strata

# 2. Install dependencies
pip install -r requirements.txt
```

**3. Set up your ThreatFox Auth-Key.** Create a file named `.env` in the project root containing:

```
THREATFOX_AUTH_KEY=your-key-here
```

This file is gitignored and never committed. (Optionally, add `MALPEDIA_APITOKEN=your-token` for fuller Malpedia access — enrichment also works without it.)

```bash
# 4. Run the pipeline (order matters: enrich reads the fetched data,
#    and the deep-dive reads the enriched data)
python3 scripts/fetch_kev.py
python3 scripts/fetch_threatfox.py
python3 scripts/enrich_families.py
python3 scripts/fetch_family_details.py
```

> On Windows, use `python` instead of `python3`.

To preview the dashboards locally, serve the `docs/` folder over HTTP (the pages use `fetch()`, which browsers block on raw file paths):

```bash
cd docs
python3 -m http.server 8000
```

Then open `http://localhost:8000` in your browser.

---

## Data sources & credits

Strata is built on the work of these public threat-intelligence providers:

- **CISA** — Known Exploited Vulnerabilities catalog
- **abuse.ch / ThreatFox** — malware IOC data (licensed CC0)
- **Malpedia** (Fraunhofer FKIE) — malware family reference data

All data belongs to its respective providers and is used in accordance with their terms.

---

## Roadmap

Planned future expansion:

- Additional data feeds (URLhaus, MalwareBazaar)
- A unified overview / landing page summarizing all feeds
- MITRE ATT&CK integration for technique mapping and richer attribution
- An artifact knowledge base — mapping malware families and techniques to the forensic artifacts they leave behind

---

## Author

Built by [BlkWizard20z](https://github.com/BlkWizard20z).
