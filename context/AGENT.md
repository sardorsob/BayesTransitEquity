# AGENT.md
Onboarding document for any new AI session on this project.
Read this first. Do not ask the user to re-explain decisions already captured here.

---

## What This Project Is

**BayesTransitEquity** is a Bayesian spatial analysis of urban transit accessibility and equity,
starting with San Diego, CA. The project has three tightly coupled outputs:

1. **A data science pipeline** — ingests GTFS transit data + census demographics, computes
   accessibility metrics per census tract, and runs hierarchical Bayesian models to produce
   posterior distributions over accessibility and transit desert probability for each neighborhood.

2. **An interactive Vercel web app** — a Next.js + Deck.gl map where users select a neighborhood
   and see its full posterior distribution, exceedance probability, uncertainty width, and
   information-theoretic divergence metrics relative to other areas or a reference distribution.
   Includes an intervention simulation layer where users can explore how route/frequency changes
   would shift the posterior.

3. **An academic paper** — framed around the argument that deterministic accessibility metrics
   mislead policy because they hide uncertainty, and that Bayesian methods change which
   neighborhoods are identified as underserved. Target venues: Environment and Planning B,
   Computers Environment and Urban Systems, or Transportation Research Part B.

---

## The Core Scientific Argument

Standard transit equity tools (Conveyal, EPA metrics, city dashboards) give single-number
accessibility scores per tract. These hide uncertainty that is especially large in data-sparse
neighborhoods. Our claim: treating accessibility as a posterior distribution rather than a point
estimate changes which tracts are flagged as underserved, and that difference is large enough to
matter for resource allocation.

The model produces:
- **P(transit desert | data)** — posterior probability of being underserved per tract
- **Exceedance probability** — P(accessibility < threshold), the primary map layer
- **Credible interval width** — uncertainty layer; wide = sparse data, narrow = confident
- **Wasserstein distance** — how far each tract's distribution is from a well-served reference
- **Entropy / surprisal** — characterizes uncertainty structure of the posterior
- **Intervention posteriors** — how distributions shift under hypothetical route/frequency changes

KL divergence and Jensen-Shannon divergence are methodologically interesting but secondary —
Wasserstein is preferred for spatial comparisons because it has geometric intuition and is
well-behaved when distributions don't overlap.

---

## Tech Stack (Decided — Do Not Relitigate)

| Layer | Tool | Reason |
|---|---|---|
| Accessibility computation | r5py (Python wrapper for Conveyal R5) | Built for city-to-regional scale, GTFS-native |
| Bayesian modeling | PyMC (MCMC) → NumPyro (if scale demands) | PyMC for SD, swap sampler if CA/national |
| Spatial modeling | CAR or BYM spatial random effects in PyMC | Standard for areal data |
| Geospatial | geopandas, pyproj | Universal, scales to national |
| Census data | censusdatadownloader or Census API | Tract-level, works nationally |
| OSM walk networks | osmnx | Walk-to-stop burden layer |
| Frontend map | Deck.gl + Mapbox | GPU-accelerated, scales to CA/national |
| Frontend framework | Next.js on Vercel | Static export of precomputed GeoJSON |
| Data serialization | Parquet (posteriors), GeoJSON (frontend) | Parquet for large posterior samples |
| GTFS aggregation | Mobility Database (formerly TransitLand) | National feed aggregator, enables scaling |

---

## Architecture: How Data Flows

```
data/raw/gtfs/          GTFS feeds (immutable)
data/raw/census/        Census tract shapefiles + ACS demographics
data/raw/osm/           OpenStreetMap walk networks
        |
        v
src/preprocessing/      Clean, join, validate
        |
        v
src/accessibility/      Compute accessibility metrics per tract (via r5py)
data/processed/accessibility/
        |
        v
src/modeling/           Hierarchical Bayesian model (PyMC)
                        Posterior samples → data/processed/posteriors/ (Parquet)
        |
        v
src/metrics/            Compute divergence, entropy, exceedance probabilities
        |
        v
scripts/export_frontend.py   Convert posteriors → GeoJSON → data/processed/geojson/
        |
        v
app/public/data/        Static GeoJSON consumed by Next.js
        |
        v
Vercel                  Interactive map served to users
```

The key seam: `scripts/export_frontend.py` is the only bridge between the Python pipeline
and the Next.js app. The frontend never touches raw data or posterior samples directly.

---

## Scalability Design (Critical)

The project starts with San Diego but is architected to scale to California and eventually
the whole US. What makes this possible:

- **Everything geographic is in configs/** — bounding box, GTFS agency IDs, census FIPS codes,
  accessibility thresholds. `src/` and `scripts/` contain zero hardcoded geography.
- **City config pattern**: `configs/san_diego.yaml` is the template. Adding California = new config
  + updated GTFS agency list + bigger compute job (variational inference instead of full MCMC).
- **GTFS via Mobility Database** — national aggregator, so multi-agency ingestion is config-driven.
- **Deck.gl + PMTiles** — frontend stack handles national-scale data without choking.
- SD → CA: medium effort (GTFS aggregation + VI instead of MCMC for 8k tracts).
- CA → National: significant data engineering but no modeling rewrite (~74k tracts, NumPyro).

---

## Folder Reference (Quick)

| Folder | What it is |
|---|---|
| `context/` | This folder. AI + human long-running memory. |
| `configs/` | City-specific + shared YAML configs. No magic numbers elsewhere. |
| `data/raw/` | Immutable source data. Gitignored. Never modify in place. |
| `data/interim/` | Throwaway cached transforms. Always gitignored. |
| `data/processed/` | Final derived datasets. Posteriors in Parquet, frontend outputs in GeoJSON. |
| `notebooks/eda/` | Exploratory EDA only (`eda/README.md`). Numbered 01–07 inside this folder. |
| `notebooks/` (01–07) | Pipeline notebooks after EDA. Import from `src/`. |
| `src/` | Reusable Python package. The science lives here. |
| `scripts/` | CLI entrypoints. Run these, don't run notebooks in production. |
| `app/` | Next.js frontend. Standalone project. Reads only from `app/public/data/`. |
| `artifacts/` | All generated outputs. Never hand-edited. Every figure is traceable to a run. |
| `paper/` | Academic paper. Figures pulled from `artifacts/figures/`. |
| `tests/` | Minimal unit tests for core loaders, transforms, and metrics. |

---

## Notebook Sequence (Planned)

Run **`notebooks/eda/`** first (see `notebooks/eda/README.md`), then the pipeline below.

| Notebook | Purpose |
|---|---|
| eda/01–07 | Inventory, GTFS, ACS/MOE–disadvantage analysis, spatial alignment + tract service table, OSM, destinations, cross-source stop coverage |
| 01_data_exploration | Pipeline entry after EDA — bridge toward processing and modeling notebooks |
| 02_gtfs_processing | Clean, validate, and join GTFS data |
| 03_accessibility_computation | Compute cumulative opportunity metrics via r5py (**complete** for SD — see `STATUS.md` / **F020**) |
| 04_bayesian_model | Hierarchical Bayesian transit desert model in PyMC |
| 05_posterior_analysis | Analyze posteriors, exceedance probabilities, credible intervals |
| 06_divergence_metrics | Compute Wasserstein, entropy, surprisal across tracts |
| 07_intervention_simulation | Counterfactual inference — how route changes shift posteriors |

---

## Key Decisions Already Made

Full rationale in `DECISIONS.md`. Summary:
- Start with San Diego, not California — faster iteration, cleaner validation.
- Precomputed posteriors served as static GeoJSON — no live inference API needed on Vercel.
- Wasserstein distance preferred over KL/JS for neighborhood comparisons.
- Exceedance probability P(accessibility < threshold) is the primary map layer.
- r5py over custom routing — handles GTFS natively at scale.
- PyMC first, NumPyro if performance demands it at larger scale.
- EDA under `notebooks/eda/` vs pipeline 01–07 at `notebooks/` root (D008).

---

## Current Status

See `context/STATUS.md` for up-to-date progress.
See `context/PAPER.md` for paper argument and figure plan.
See `DECISIONS.md` for all architectural decisions with rationale.
See `ASSUMPTIONS.md` for all modeling and data assumptions.

---

## Rules for This Session

- Read `development_rules.md` for the full operating contract.
- Never hardcode geography. Use `configs/`.
- Never hand-edit anything in `artifacts/`.
- Log every meaningful decision in `DECISIONS.md`.
- Update `context/STATUS.md` when work is done.
- The frontend (`app/`) and the pipeline (`src/` + `scripts/`) are separate concerns.
  Do not mix them.
