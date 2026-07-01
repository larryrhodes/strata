# Strata

**Strata** is a threat-intelligence aggregator with a forensics twist: it ingests
current cyber-threat intel and maps it to the *artifacts* — the layers of evidence —
that each technique leaves behind on a system. Like geological strata, an intrusion
leaves readable layers; this tool helps you read them.

The MVP pulls the CISA Known Exploited Vulnerabilities (KEV) catalog daily via GitHub
Actions, stores it as versioned JSON in the repo, and publishes a searchable static
dashboard via GitHub Pages.

## Local setup

```bash
# 1. Clone your repo (after you've created it on GitHub and pushed this code)
git clone https://github.com/<your-username>/strata.git
cd strata
```
`git clone` downloads a full copy of the repo (all history, all branches) to your machine.

```bash
# 2. Run the fetch script locally to test it
python3 scripts/fetch_kev.py
```
This should print something like `Fetched 1200 total KEV entries.` and create files under
`data/kev/`. If you see `HTTP Error 403`, check your network/firewall isn't blocking
`cisa.gov` (some corporate networks do).

```bash
# 3. Copy the data into docs/ so you can preview the dashboard locally
mkdir -p docs/data/kev
cp data/kev/latest.json docs/data/kev/latest.json
```

```bash
# 4. Serve the docs/ folder locally to preview the dashboard
cd docs
python3 -m http.server 8000
```
`python3 -m http.server 8000` starts a minimal local web server on port 8000, serving
whatever's in the current directory. This is needed because browsers block `fetch()`
calls to local files (`file://`) for security reasons — you need an actual HTTP server,
even a throwaway one, for the JS `fetch()` call in `index.html` to work.

Then open `http://localhost:8000` in your browser. Ctrl+C in the terminal stops the server.

## Publishing to GitHub

```bash
# From the project root, first time only:
git init
git add .
git commit -m "Initial MVP: KEV fetch script, dashboard, GitHub Actions workflow"
```
- `git init` — turns the current folder into a git repository (creates a hidden `.git/` folder).
- `git add .` — stages all files in the directory for the next commit. The `.` means "everything here."
- `git commit -m "..."` — saves a snapshot of the staged files with a message describing the change.

```bash
# Create the repo on GitHub first (via github.com/new), then:
git remote add origin https://github.com/<your-username>/strata.git
git branch -M main
git push -u origin main
```
- `git remote add origin <url>` — tells your local repo where the "origin" (GitHub) copy lives.
- `git branch -M main` — renames your current branch to `main` (GitHub's default branch name).
- `git push -u origin main` — uploads your commits to GitHub. The `-u` sets `origin main` as
  the default target, so future pushes can just be `git push`.

## Enabling GitHub Pages

1. On GitHub, go to your repo → **Settings** → **Pages**.
2. Under "Build and deployment", set **Source** to "Deploy from a branch".
3. Set **Branch** to `main` and folder to `/docs`.
4. Save. GitHub will publish at `https://<your-username>.github.io/strata/`.

## Testing the automation

Go to the **Actions** tab on GitHub → select "Fetch CISA KEV" → click **Run workflow**
(this only appears because of the `workflow_dispatch` trigger in the workflow file).
This lets you confirm the whole pipeline works without waiting for the daily schedule.

## Project structure

```
strata/
├── scripts/
│   └── fetch_kev.py         # Ingestion logic — pulls & saves CISA KEV data
├── data/kev/                 # Full historical archive (dated snapshots + latest.json)
├── docs/                     # Served by GitHub Pages
│   ├── index.html            # Dashboard (search/filter over KEV data)
│   └── data/kev/latest.json  # Copy of latest data, published for the dashboard to fetch
├── .github/workflows/
│   └── fetch-kev.yml         # Scheduled automation (daily fetch + commit + push)
└── requirements.txt
```

## Next steps (post-MVP)

- Add more sources: `abuse.ch` ThreatFox, NVD API, vendor RSS feeds
- Normalize sources into a common schema
- Start the ATT&CK technique → artifact mapping knowledge base
- Add trend/diff views using the historical dated snapshots in `data/kev/`
