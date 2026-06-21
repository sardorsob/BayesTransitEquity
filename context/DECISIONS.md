# DECISIONS.md
Every meaningful architectural, modeling, or tooling decision with rationale.
Format: decision → alternatives considered → rationale → consequences.

---

## D001 — Start with San Diego, not California
**Date**: 2026-03-28
**Decision**: Phase 1 is San Diego only (~400 census tracts, 2 GTFS agencies).
**Alternatives**: Start directly with California (~8,000 tracts, 40+ agencies).
**Rationale**: Faster iteration, cleaner spatial validation, manageable MCMC compute,
better ability to sanity-check results against known neighborhoods.
**Consequences**: Must design code to be config-driven from day one so SD → CA requires no rewrite.

---

## D002 — Precomputed posteriors, no live inference API
**Date**: 2026-03-28
**Decision**: Run MCMC offline, export posteriors as static GeoJSON served by Vercel.
**Alternatives**: FastAPI backend on Render/Railway running inference on demand.
**Rationale**: Simpler deployment, free to host, faster frontend, no cold-start issues.
**Consequences**: Map data is not real-time. Updating requires re-running the pipeline
and redeploying. Acceptable for a planning tool (transit data changes infrequently).

---

## D003 — Wasserstein distance preferred over KL/JS for neighborhood comparisons
**Date**: 2026-03-28
**Decision**: Use Wasserstein (Earth Mover's Distance) as the primary distributional
divergence metric for comparing neighborhoods.
**Alternatives**: KL divergence, Jensen-Shannon divergence.
**Rationale**: Wasserstein has geometric intuition, is symmetric, well-behaved when
distributions don't overlap, and more natural for spatial comparisons.
KL/JS referenced in methods but not the primary UI metric.
**Consequences**: Implement in `src/metrics/divergence.py`. scipy.stats.wasserstein_distance handles 1D.

---

## D004 — Exceedance probability P(access < threshold) as primary map layer
**Date**: 2026-03-28
**Decision**: Primary heatmap variable is exceedance probability, not posterior mean.
**Alternatives**: Posterior mean, posterior median, raw scores.
**Rationale**: Most planning-actionable output. Directly answers "how likely is this
neighborhood to be underserved?" Maps cleanly to 0-1 color scale.
**Consequences**: Accessibility threshold must be defined in config and documented in
ASSUMPTIONS.md. Sensitivity analysis at 30/45/60 min required.

---

## D005 — r5py for accessibility computation
**Date**: 2026-03-28
**Decision**: Use r5py (Python wrapper for Conveyal R5) for travel time + accessibility.
**Alternatives**: Custom OpenTripPlanner wrapper, gtfs_functions, manual routing.
**Rationale**: GTFS-native, designed for regional-scale accessibility, handles multi-modal
routing and departure-window sampling.
**Consequences**: Requires Java runtime. Document setup carefully in README.

---

## D006 — PyMC first, NumPyro if performance demands
**Date**: 2026-03-28
**Decision**: Use PyMC for the Bayesian model in SD phase.
**Alternatives**: Stan, NumPyro, Bambi.
**Rationale**: Best Python integration, good spatial model support, well-documented for
hierarchical models. NumPyro is the fallback for California+ scale.
**Consequences**: Write model structure documentation separate from implementation so
PyMC → NumPyro transition stays clean.

---

## D007 — Deck.gl + Next.js for frontend
**Date**: 2026-03-28
**Decision**: Next.js on Vercel with Deck.gl for the map.
**Alternatives**: Leaflet/React-Leaflet (simpler), Kepler.gl (prebuilt).
**Rationale**: Deck.gl is GPU-accelerated and handles large GeoJSON without performance
issues. Critical for CA/national scaling.
**Consequences**: More complex setup than Leaflet but scales correctly.

---

## D008 — EDA notebooks live in `notebooks/eda/`, pipeline notebooks in `notebooks/`
**Date**: 2026-03-28
**Decision**: Separate EDA notebooks (numbered 01–07 under `notebooks/eda/`) from the
pipeline notebooks (numbered 01–07 at `notebooks/` root). EDA is exploratory and
disposable; pipeline notebooks are the reproducible analytical record.
**Alternatives**: Single flat `notebooks/` folder, numbered sequentially.
**Rationale**: Prevents numbering collision. EDA notebooks may be rerun non-linearly;
pipeline notebooks must run in strict order.
**Consequences**: Two README files needed. Any agent must check which set they are working
in before referencing notebook numbers.

---

## D009 — NCTD vs study bbox: scope decision (RESOLVED)
**Date**: 2026-03-29 (opened) · **Resolved**: 2026-03-30
**Status**: **RESOLVED** — expand bbox for a county-wide study (Option 1).
**Context**: The original bbox `[-117.28, 32.53, -116.93, 33.11]` left only ~18.76% of NCTD
stops inside the window (F002, former A009), which misrepresented NCTD if the paper claims
county-wide equity.
**Decision**: **Expand bbox** to `[-117.40, 32.53, -116.80, 33.35]` in `configs/san_diego.yaml`
(west to −117.40, east to −116.80, north to 33.35) to capture Escondido–Oceanside–Vista
corridors and most NCTD core service while staying in San Diego County–relevant geography.
**Alternatives not chosen**: (2) keep small bbox + document partial NCTD; (3) MTS-only Phase 1.
**Consequences**:
- **OSM walk network** must be re-downloaded for the new extent:  
  `python scripts/download_data.py --config configs/san_diego.yaml --sources osm --force`
- **EDA notebooks 04–07** (and any artifact that used the old bbox) should be re-executed before
  treating tract counts, stop coverage, and destination tables as current; pipeline notebooks
  01–07 consume config-driven bbox and do not require a code change for the expansion.
- Paper Data section should state the expanded window and that both MTS and NCTD are in scope.


---

## D010 — GTFS ID namespacing scheme: `{agency_id}:{original_id}`
**Date**: 2026-04-01
**Decision**: When merging multiple GTFS feeds, all IDs (stop_id, route_id, trip_id, service_id, shape_id, agency_id) are prefixed with the short agency tag and a colon — e.g. `mts:12345`, `nctd:COASTER`. `parent_station` and `agency_id` in routes are also remapped.
**Alternatives**: UUID suffix, sequential renumbering, agency-index suffix.
**Rationale**: Human-readable in debug output. Prefix is recoverable (strip up to first `:`). Collision-proof across any number of agencies. Consistent with how Mobility Database recommends multi-feed merging.
**Consequences**:
- `prefix_feed()` in `notebooks/02_gtfs_processing.ipynb` is the single source of truth for this logic.
- `parent_station` must be remapped alongside `stop_id` (bug encountered and fixed 2026-04-01).
- Feeds whose `routes.txt` omits `agency_id` (GTFS-legal for single-agency feeds) must have the column synthesised before merge — `prefix_feed()` now handles this.
- Downstream code must never strip the prefix; the prefix is part of the ID in all processed files.

---

## D011 — Primary BYM2 estimand: raw z-scored X; Spatial+ as sensitivity with light eigen removal
**Date**: 2026-04-08
**Decision**:
1. **Paper primary:** BYM2 fit on **z-scored raw covariates** **X** (no eigenvector residualization). Enable in `notebooks/04_bayesian_model.ipynb` with env **`PIPELINE_NO_SPATIAL_PLUS=1`** (documented in notebook header).
2. **Sensitivity / supplemental:** **Spatial+** **X** (project covariates off the smoothest Laplacian eigenvectors, then re-standardize). Default **k_remove = max(1, int(0.05 × n_tracts))** (~**5%**), **not** the regression-only literature band **14–21%**, because **ICAR already models spatial structure**; aggressive removal produced **Moran's I ≈ 0.55** on posterior-mean residuals for an archived **15%** run (**F026**).
3. When Spatial+ is **on**, **MCMC `draws` floor at 10 000** (unless `PIPELINE_FAST_MCMC`) to mitigate low **α**/**β** ESS seen at 8k draws.
**Alternatives**: Treat Spatial+ slopes as primary; keep **15%** k_remove; dual automated fits in one notebook run.
**Rationale**: Publication-ready diagnostics on **2026-04-05 raw X** (R-hat, ESS, 0 divergences); equity Spearman stable across specs (**F024**). Residual spatial autocorrelation under heavy Spatial+ indicates **misspecified sharing of variance** between design matrix and ICAR.
**Consequences**: Export **`pipeline__04_moran_residual_summary__<RID>.csv`** every run; compare Moran's I across estimands. Optional nb04 cell can recompute Moran from another saved **`idata.nc`** (same cohort) → **`pipeline__04_moran_residual_from_idata_compare__<RID>.csv`** without refitting. **PAPER.md** and **FINDINGS** reference this split explicitly.

**Naming (2026-04-09):** Prefer semantic **`PIPELINE_RUN_ID`** values **`fit_raw_zscore_x`** and **`fit_spatial_plus_x`** (see `src.utils.config`) instead of calendar dates. Legacy date filenames can be renamed to those stems using **`LEGACY_BYM2_RUN_ID_TO_SEMANTIC`** in `config.py` as a guide.

---

## D013 — nb06 repurposed as `06_equity_decomposition.ipynb` (supersedes D012's deprecation)
**Date**: 2026-04-17
**Decision**: The `notebooks/06_*.ipynb` slot is now `06_equity_decomposition.ipynb` — a comprehensive equity-decomposition + model-validation stage that runs after nb05 and before nb07. The D012 deprecation of the originally-planned `06_divergence_metrics.ipynb` stands (those metrics are covered by nb05 extension cells per D003), but the notebook *file slot* is now occupied by a different purpose.
**What nb06 contains (11 sections)**:
- §1 Gini + Lorenz (posterior mean jobs, population-weighted)
- §2 Concentration Index (posterior + deterministic) vs disadvantage_z
- §3 Subgroup posteriors (disadvantage quartiles + vehicle-ownership)
- §4 Multi-destination equity table (Jobs / Hospitals / Groceries / Schools × 30 / 45 / 60 min)
- §5 Composite deficit (tracts that exceed desert probability on all four destinations)
- §6 Posterior predictive checks (overall + subgroup)
- §7 PSIS-LOO comparison + Pareto-k diagnostics + pointwise map
- §8 Prior sensitivity (importance-reweighting grid over `beta_sigma` × Student-t `nu`)
- §9 Moran's I on posterior-mean residuals, raw X (comparison to Spatial+ from F026)
- §10 Drop-one covariate robustness (importance-reweighted)
- §11 Summary export (one-row-per-metric roll-up)
**Alternatives considered**: (a) leave nb06 blank and build nb07 directly, (b) fold all of this into a long nb05. Both rejected: (a) loses reviewer-expected validation content; (b) nb05 is already 23 cells and scope-tight around paper Figs 3–8. nb06 is the cleanest home for multi-destination + PPC + LOO + prior sensitivity.
**Rationale**: A paper reviewer will ask for every one of the 11 sections. Running them inside one notebook — with shared data loading, shared estimand pin cell, and a single summary CSV — is cheaper than answering the questions post-submission. The artefact family `pipeline__06_*__fit_raw_zscore_x.{csv,png}` also establishes the pattern for the Spatial+ sensitivity run if we execute it later.
**Consequences**:
- D012's "do not delete nb06, leave as a stub" instruction is now moot — the file has real content.
- `STATUS.md` + `CHANGELOG.md` + `structure.md` + `PAPER.md` all updated to reference nb06 and F028/F029/F030.
- Both the Wasserstein-by-quartile table and the Wasserstein-equity-metrics CSV live under `artifacts/tables/pipeline/pipeline__06_wasserstein_*__fit_raw_zscore_x.csv` (extending nb05's per-tract Wasserstein CSV with quartile roll-ups).
- **F029 is a new honest limitation**: the paper must disclose PPC-p extremes, LOO Pareto-k failure, and raw-X residual Moran's I = 0.64. Methods §6 framing is specified in F029's paper implication section.

---

## D012 — nb06 divergence metrics notebook: DEPRECATED in favor of nb05 Wasserstein coverage (superseded by D013)
**Date**: 2026-04-08
**Decision**: The originally planned `notebooks/06_divergence_metrics.ipynb` (a standalone notebook for distributional divergence analysis) is **deprecated**. Its scope has been fully absorbed by nb05 extension cells added in the April 7–8 production run.
**What nb05 now covers (rendering nb06 redundant)**:
- §7: KDE fan chart for 4 case-study tracts (Fig 6) — posterior distribution shapes on jobs scale
- §8: Wasserstein-1 distance map (Fig 8) + per-tract CSV vs top-quartile reference pool (D003)
- §9: Multi-threshold equity table (30/45/60 min) — Spearman for both deterministic and exceedance
- §10: Hook candidates CSV — near-Q25 tracts with ambiguous exceedance (≈ 50%)
**Alternatives considered**: Keep nb06 for additional metrics (KL divergence, Jensen-Shannon). These are covered by D003's rationale for preferring Wasserstein over KL/JS; no additional implementation needed.
**Consequences**: If a reviewer requests KL/JS comparison, it can be added as an optional cell in nb05 §8, not a new notebook. The `notebooks/06_divergence_metrics.ipynb` file, if it exists, should be left as a stub with a deprecation note in its header cell pointing to nb05. Do not delete it — the pipeline numbering (01–07) should remain contiguous for the paper's Methods section reference.
