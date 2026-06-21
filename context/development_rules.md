# development_rules.md
Universal Project Architecture + Operating Rules — BayesTransitEquity
AI-Assisted Development (Cursor / Claude / LLMs)

This file is the **single operational contract** for how this repository is structured,
how work is performed, and how results are logged so that:
- Humans can understand and reproduce everything later.
- AI agents can safely extend the repo without drifting the structure.
- Analysis, production code, frontend, and paper artifacts stay cleanly separated.
- Every claim in the paper can be traced to code + data + artifacts.

If this file conflicts with any other doc, **this file wins** unless `DECISIONS.md` explicitly supersedes it.

---

## 0) Non-Negotiable Principles

1. **Reproducibility beats convenience**
   - Anything important must be reproducible from code + config + seed. No manual steps.
   - MCMC runs are expensive — every run must be logged with its seed and config snapshot.

2. **Artifact-first, notebook-second**
   - Notebook outputs are not the deliverable.
   - Deliverables are exported artifacts (figures / GeoJSON / posteriors / reports) + a written log.

3. **One source of truth for structure**
   - `structure.md` describes the repo layout and indexes all produced results.

4. **Separation of concerns**
   - `notebooks/eda/` = exploratory EDA (separate numbering from pipeline)
   - `notebooks/*.ipynb` (01–07) = sequential pipeline notebooks after EDA
   - `src/` = reusable Python code (imported by notebooks and scripts)
   - `scripts/` = CLI entrypoints (pipeline runners, data downloaders, frontend exporters)
   - `app/` = Next.js Vercel frontend (reads from `data/processed/geojson/`)
   - `artifacts/` = all generated outputs (never hand-edited)
   - `paper/` = academic paper (figures pulled from artifacts)
   - `context/` = AI + human project memory

5. **Agent-safe iteration**
   - Every meaningful change must update logs (DECISIONS / ASSUMPTIONS / STATUS).
   - Avoid silent changes that break continuity across sessions.
   - Always read `context/AGENT.md` at the start of a new session.

6. **Config-driven geography**
   - No city, bounding box, or agency ID is hardcoded anywhere in `src/` or `scripts/`.
   - All geographic and agency-specific parameters live in `configs/<city>.yaml`.
   - This is what makes SD → California → National a config change, not a rewrite.

---

## 1) Repository Layout

```
BayesTransitEquity/
├── README.md
├── development_rules.md          # this file
├── structure.md                  # repo map + artifact index + results log
├── DECISIONS.md                  # why we chose X over Y
├── ASSUMPTIONS.md                # modeling/data assumptions + caveats
├── CHANGELOG.md                  # milestones
├── requirements.txt
├── .gitignore
│
├── context/                      # AI + human long-running memory
│   ├── AGENT.md                  # onboarding doc for new AI sessions
│   ├── PROJECT_BRIEF.md          # what, why, success criteria
│   ├── GLOSSARY.md               # domain terms + acronyms
│   ├── DATASETS.md               # data sources, schemas, licensing
│   ├── INTERFACES.md             # I/O contracts, GeoJSON schemas
│   ├── STATUS.md                 # current state, what's done, what's next
│   ├── RISKS.md                  # known risks + mitigations
│   └── PAPER.md                  # paper framing, argument, figure plan
│
├── configs/
│   ├── defaults.yaml             # shared defaults across cities
│   └── san_diego.yaml            # SD-specific: bbox, agencies, FIPS, thresholds
│
├── data/
│   ├── raw/                      # immutable source data (gitignored)
│   │   ├── gtfs/                 # GTFS feeds per agency
│   │   ├── census/               # tract shapefiles + ACS demographics
│   │   └── osm/                  # OpenStreetMap pedestrian walk networks
│   ├── interim/                  # cached transforms (always gitignored)
│   └── processed/                # final derived datasets (selective tracking)
│       ├── accessibility/        # computed accessibility metrics per tract
│       ├── posteriors/           # exported posterior samples (Parquet)
│       └── geojson/              # frontend-ready GeoJSON (consumed by app/)
│
├── notebooks/
│   ├── eda/                      # exploratory EDA (see eda/README.md)
│   │   ├── README.md
│   │   └── 01_… through 07_… .ipynb
│   ├── 01_data_exploration.ipynb
│   ├── 02_gtfs_processing.ipynb
│   ├── 03_accessibility_computation.ipynb
│   ├── 04_bayesian_model.ipynb
│   ├── 05_posterior_analysis.ipynb
│   ├── 06_divergence_metrics.ipynb
│   └── 07_intervention_simulation.ipynb
│
├── src/                          # reusable Python package
│   ├── io/                       # data loaders
│   ├── preprocessing/            # cleaning + spatial joins
│   ├── accessibility/            # accessibility metric computation
│   ├── modeling/                 # Bayesian model definitions
│   ├── metrics/                  # divergence, entropy, equity metrics
│   ├── evaluation/               # MCMC diagnostics + validation
│   ├── viz/                      # plotting + GeoJSON export helpers
│   └── utils/                    # paths, seeds, logging
│
├── scripts/                      # CLI entrypoints
│   ├── download_data.py
│   ├── run_pipeline.py
│   ├── run_model.py
│   └── export_frontend.py        # converts posteriors → GeoJSON for app/
│
├── app/                          # Next.js Vercel frontend
│   ├── components/
│   │   ├── Map/
│   │   ├── PosteriorPanel/
│   │   ├── DivergenceView/
│   │   └── InterventionSlider/
│   ├── public/data/              # static GeoJSON served by Vercel
│   └── pages/
│
├── artifacts/                    # ALL generated outputs (never hand-edited)
│   ├── figures/
│   ├── tables/
│   ├── models/                   # serialized PyMC traces + posteriors
│   ├── reports/
│   └── logs/
│       ├── runs/                 # per-run metadata bundles
│       ├── prompts/              # LLM prompt/response snapshots
│       └── provenance/           # input hashes + manifests
│
├── paper/                        # academic paper
│   ├── main.md
│   ├── sections/
│   ├── figures/                  # paper-specific figure exports
│   └── references.bib
│
└── tests/
```

---

## 2) Data Governance

- `data/raw/`: immutable. Never modify in place. Gitignored due to size.
- `data/interim/`: always gitignored. Throwaway caches.
- `data/processed/`: track only if small + legally allowed. Otherwise export manifests.
- Every preprocessing step must be in code (`src/preprocessing/` or `scripts/`), configured (`configs/`), and logged.

### GTFS-specific rules
- Each agency feed goes in `data/raw/gtfs/<agency_id>/`.
- Feed version + download date logged in `context/DATASETS.md`.
- Never modify raw GTFS files. All cleaning happens in `src/preprocessing/gtfs_clean.py`.

---

## 3) Experiment Tracking

Run ID convention: `YYYY-MM-DD__HHMM__<short_tag>__<git_shortsha>`

Every run writes a bundle to `artifacts/logs/runs/<run_id>/`:
- `meta.json` — run_id, timestamp, git hash, machine info
- `config.yaml` — exact config snapshot
- `seeds.json` — RNG seeds (critical for MCMC reproducibility)
- `inputs.json` — input dataset paths + hashes
- `outputs.json` — produced artifacts with relative paths
- `metrics.json` — key metrics (R-hat, ESS, exceedance probabilities)
- `notes.md` — short narrative (what / why / results / next steps)

---

## 4) Artifact Rules

- Never hand-edit generated artifacts. Change code → rerun → regenerate.
- Naming: `<stage>__<short_description>__<run_id>.<ext>`
- Figures → `artifacts/figures/`
- Tables → `artifacts/tables/`
- Models/traces → `artifacts/models/`
- Reports → `artifacts/reports/`
- Run logs → `artifacts/logs/runs/<run_id>/`

---

## 5) Notebook Rules

- Must run top-to-bottom without manual state.
- Heavy logic moves into `src/`. Notebooks import from `src/`.
- Always use `src/utils/paths.py` for path resolution — no hardcoded absolute paths.
- A notebook is not "done" unless:
  - outputs exported to `artifacts/`
  - run bundle exists for reportable results
  - `structure.md` updated
  - assumptions/decisions updated if anything changed

---

## 6) Code Standards

- Small functions with docstrings (what it does, inputs/outputs, key assumptions).
- No magic numbers — everything in `configs/*.yaml`.
- Set and log RNG seeds whenever randomness affects results.
- No hardcoded absolute paths anywhere in `src/` or `scripts/`.
- Minimal prints; use lightweight logging.

---

## 7) Frontend Rules (app/)

- `app/` is a standalone Next.js project with its own `package.json`.
- It reads data only from `app/public/data/` (static GeoJSON) or `data/processed/geojson/`.
- The bridge between the Python pipeline and the frontend is `scripts/export_frontend.py`.
- Never copy figures or data manually into `app/`. Always use the export script.
- Frontend deployment (Vercel) is independent of the Python pipeline.

---

## 8) Paper Rules (paper/)

- Paper figures must be exported artifacts — never created manually.
- Every figure referenced in the paper must exist in `artifacts/figures/` and be traceable to a run bundle.
- `context/PAPER.md` tracks the argument, which results map to which figures, and what claims still need empirical support.
- Paper and code evolve together. If a modeling decision changes, update both `DECISIONS.md` and `paper/sections/` accordingly.

---

## 9) LLM / Agent Rules

- Read `context/AGENT.md` at the start of every new session.
- When an LLM output influences design or code, log a snapshot in `artifacts/logs/prompts/<date>__<topic>/`.
- Every meaningful architectural or modeling choice goes in `DECISIONS.md`.
- All modeling/data assumptions go in `ASSUMPTIONS.md`.
- Never store secrets or API keys anywhere in the repo.

---

## 10) Definition of Done

A milestone is done only if:
- Results are exported to `artifacts/` (not only notebook output).
- Run bundles exist for final selected results.
- `structure.md` references everything important.
- Assumptions and decisions are documented.
- A short narrative exists (markdown or paper section).
