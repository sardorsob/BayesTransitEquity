# STATUS.md
Current state of the project. Updated at the end of every meaningful work session.

---

## Current Phase
**Phase 4 — All pipeline notebooks complete; ready for paper writing + dashboard frontend** — nb04 (`fit_raw_zscore_x`), nb05 (all 23 cells, all 7 paper figures), nb06 (all 11 sections, equity decomposition + model validation), and **nb07 (intervention simulation, all 10 sections) are finished and production-ready**. All 9 paper figures exist (Fig 1 — study area, Fig 2 — deterministic access, Figs 3–8 — Bayesian posteriors + Wasserstein, Fig 9 — intervention effect). Next step: write paper sections using F020 / F024–F031 findings, and wire `data/processed/geojson/*` into the Next.js dashboard.

## What Is Done
- [x] Project concept defined and scoped
- [x] Reference literature reviewed (see `refrences/`)
- [x] Folder structure and operating rules established
- [x] Tech stack decided (see `DECISIONS.md`)
- [x] Paper framing drafted (see `context/PAPER.md`)
- [x] `AGENT.md` written — new sessions can onboard without re-explaining context
- [x] `notebooks/eda/` scaffold — seven notebooks + `eda/README.md`
- [x] `scripts/download_data.py` written and verified (dry-run + logic tested)
  - Downloads: MTS GTFS, NCTD GTFS, TIGER/Line tracts, ACS via Census API, LODES, OSM
  - Freeze logic: skips existing files unless `--force`
  - Writes provenance manifest to `artifacts/logs/provenance/`
  - Extracts GTFS zips + reads `feed_info.txt` for version/validity window
  - See `context/DATASETS.md` for all confirmed download URLs
- [x] Raw data downloaded successfully on local machine
  - MTS feed: version "2601 unmerged v2", valid 2026-01-25 → 2026-06-06 (current)
  - NCTD feed: version "Version_20250205", valid 2025-02-05 → 2025-05-17 (**EXPIRED ~10 months**)
  - TIGER/Line tracts: downloaded, extracted, 737 SD county tracts confirmed
  - ACS 5-year estimates: downloaded (737 tracts)
- [x] `notebooks/eda/01_inventory_and_provenance.ipynb` — artifact export cells added
  - Exports: `eda__file_inventory__<date>.csv`, `eda__gtfs_feed_info__<date>.csv`,
    `eda__manifest_summary__<date>.csv`, `eda__census_artifact_check__<date>.csv`
- [x] `notebooks/eda/02_gtfs_schedule_exploration.ipynb` — artifact export cells added
  - Figure: `eda__gtfs_stops_map__<date>.png` (dpi=200)
  - Tables: `eda__gtfs_feed_summary__`, `eda__gtfs_bbox_coverage__`, `eda__gtfs_calendar__`,
    `eda__gtfs_headways__`, `eda__gtfs_data_quality__` (all `<date>.csv`)
  - Key finding: NCTD only 18.76% of stops fall inside study bbox (342 / 1,823)
  - Key finding: MTS best headway 5 min (route 201), worst 70 min (route 83)
  - Key finding: NCTD best headway 20 min, worst 180 min (route 395)
- [x] **2026-03-29** — GTFS force refresh + Mobility catalog refresh executed; `python -m nbconvert --execute` run on EDA notebooks **01** and **02** (artifacts written under `artifacts/tables/` and `artifacts/figures/eda__gtfs_stops_map__2026-03-29.png`)
- [x] `notebooks/eda/03_census_tracts_and_acs.ipynb` — executed via `python -m nbconvert --execute` (tables + figures dated 2026-03-29)
  - Figures: `eda__acs_moe_vs_disadvantage__2026-03-29.png`, `eda__acs_choropleth_rel_moe_pop__2026-03-29.png`
  - Tables: `eda__acs_sd_tract_attributes__`, `eda__acs_tiger_alignment__`, `eda__acs_moe_disadvantage_spearman__`, `eda__acs_summary_stats__` (see `context/structure.md` artifact index)
- [x] `notebooks/eda/04_spatial_alignment_and_coverage.ipynb` — CRS manifest, tract×bbox overlap, in-bbox stops→tract join, route headway rollups, centroid→nearest-stop distance, optional merge from latest `eda__acs_sd_tract_attributes__*.csv`
  - Figures: `eda__tract_stops_choropleth__2026-03-29.png`, `eda__tract_nearest_stop_distance__2026-03-29.png`
  - Tables: `eda__spatial_layers_crs__2026-03-29.csv`, `eda__tract_gtfs_service__2026-03-29.csv` (see `context/structure.md`)
- [x] `notebooks/eda/05_osm_pedestrian_network.ipynb` — GraphML stats, tract walk-edge density (centroid proxy), Spearman vs `disadvantage_z` (needs `data/raw/osm/sd_walk_network.graphml`)

- [x] `notebooks/eda/06_opportunities_and_destinations.ipynb` — executed (2026-03-29)
  - Figures: `eda__osm_destinations_map__2026-03-29.png`
  - Tables: `eda__opportunities_destination_summary__`, `eda__osm_destinations_fetch_status__`,
    `eda__osm_destinations_groceries/hospitals/schools__`, `eda__tract_osm_destination_counts__`,
    `eda__lodes_wac_run_status__` (all `2026-03-29.csv`)
  - Key finding: 143 hospitals, 886 groceries, 1,000 schools (capped) in bbox; 85.3% of tracts have zero hospitals; 13.7% have zero of all three
  - Key finding: Destination counts positively correlated with disadvantage (density confound, F017)
  - **LODES:** resolved 2026-03-30 — `ca_wac_2021.csv.gz` downloaded; used in pipeline nb03 for job accessibility
- [x] `notebooks/eda/07_cross_source_sanity_joins.ipynb` — executed (2026-03-30)
  - Tables: `eda__cross_source_join_diagnostics__2026-03-30.csv`,
    `eda__tracts_zero_or_few_stops_summary__2026-03-30.csv`,
    `eda__tracts_zero_or_few_stops_within_0p5km__2026-03-30.csv`
  - Figures: `eda__nearest_stop_distance_hist__2026-03-30.png`,
    `eda__tracts_zero_or_few_stops_within_0p5km__2026-03-30.png`
  - Key finding: 4-source master join clean — 737/737 GEOIDs, zero orphans (F018)
  - Key finding: Only 1 in-bbox tract combines high disadvantage (z>0.5) with zero stops
    within 0.5 km — transit access deserts are overwhelmingly in affluent suburbs (F019)
- [x] **EDA PHASE COMPLETE** — all 7 EDA notebooks executed; artifacts saved; findings F001–F019
- [x] **2026-03-30** — **EDA 04–07 re-run** with expanded bbox `[-117.40, 32.53, -116.80, 33.35]`. Key updated numbers:
  - 726 of 737 tracts in-bbox (was 571); 189 tracts with NCTD stops (was ~0 in old bbox)
  - OSM network: 337,824 nodes, 906,988 edges, 65,750 km, single component (was 241,423 nodes, 43,841 km)
  - Density confound correlations confirmed county-wide: ρ(stop count)=+0.391, ρ(dist to stop)=−0.535, ρ(headway)=−0.397, ρ(walk density)=+0.371
  - Destination counts: 181 hospitals, 1,170 groceries, 1,273 schools (school cap from old run resolved)
  - Zero-stop tracts within 0.5 km: 98 (county-wide); 2 in-bbox high-disadvantage exceptions (Tract 209.03 rural fringe, Tract 139.07)
  - FINDINGS.md updated: F002 (note), F012–F019 all updated with expanded-bbox numbers
  - structure.md artifact index updated with all 2026-03-30 artifact entries
  recorded in `context/FINDINGS.md`. Master analysis table ready for pipeline.
- [x] **DENSITY CONFOUND confirmed county-wide** — most disadvantaged tracts have more stops (ρ=+0.391),
  better headways (ρ=−0.397 on min headway), closer stops (ρ=−0.535 on distance), denser walk
  networks (ρ=+0.371), and more within-tract destinations (ρ=+0.258 groceries, ρ=+0.345 hospitals).
  The paper's central reframe: supply metrics say transit is pro-poor; r5py accessibility will tell
  whether actual opportunity reach follows (see F014, F015, F017, PAPER.md).
- [x] **2026-03-30** — **D009 resolved**: study bbox expanded to `[-117.40, 32.53, -116.80, 33.35]` in `configs/san_diego.yaml` (county-wide MTS + NCTD). `configs/ASSUMPTIONS.md` **A009** updated. **LODES** downloaded → `data/raw/external/lodes/ca_wac_2021.csv.gz` (provenance `artifacts/logs/provenance/data_manifest_san_diego_2026-03-30_1827.json`). **OSM** regeneration for the new bbox: run `python scripts/download_data.py --config configs/san_diego.yaml --sources osm --force` and allow **extra time** (expanded window is much larger than the original EDA bbox; Overpass may 504-retry).

- [x] **2026-04-01** — **Pipeline nb01–nb03 debugged and unblocked.** Multiple critical bugs fixed across two sessions:
  - `scripts/download_data.py` — osmium `.EXE` uppercase suffix bug on Windows fixed; `_find_osmium()` helper returns bare `"osmium"` on Windows to avoid Windows argument-zero misparse.
  - `configs/san_diego.yaml` — file was truncated (960 bytes, missing `r5:`, `census:`, NCTD blocks); restored from git.
  - `notebooks/02_gtfs_processing.ipynb` — `prefix_feed()` fixed with two patches:
    1. Now prefixes `parent_station` in stops table (was causing R5 `NullPointerException`).
    2. Now synthesises `agency_id` on routes tables that lack the column (NCTD's `routes.txt` omits `agency_id` for single-agency GTFS; all 47 NCTD routes had `agency_id = NaN` → R5 NPE).
  - `data/processed/gtfs/sd_merged_bbox.zip` — patched in place: NCTD agency (`nctd:NCTD`) added to `agency.txt`; all 47 NCTD routes assigned correct `agency_id`; all 306 orphan `parent_station` refs cleared. Full referential integrity verified (14/14 checks pass).
  - `notebooks/03_accessibility_computation.ipynb` — Java setup block merged into r5py import cell so it cannot be skipped; TIGER FIPS comparison fixed to string-based `str.zfill()` comparison (was brittle `Int64` cast).
  - `src/utils/paths.py` — restored (was empty on disk); `find_osmium_executable()` handles uppercase `.EXE` suffix.
  - **R5 `TransportNetwork` load confirmed working** (no NPE) after all GTFS patches.
- [x] **2026-04-01 / 2026-04-03** — **`notebooks/03_accessibility_computation.ipynb` run to completion** (chunked tract×tract OD, jobs + POI cumulative accessibility, bundle export).
  - **Canonical run ID:** `2026-04-03` (see `artifacts/tables/pipeline__03_accessibility_summary__2026-04-03.csv`; duplicate `2026-04-01` run is same logic, earlier fingerprint).
  - **Outputs:** `data/processed/accessibility/tract_tract_od_traveltime__2026-04-03.parquet`, `tract_jobs_accessibility__…`, `tract_poi_*` OD + counts, **`tract_accessibility_bundle__2026-04-03.parquet`** (primary input for nb04); figure `artifacts/figures/pipeline__03_accessibility_choropleth__2026-04-03.png`.
  - **Scale:** **726** in-box tract origins; thresholds **30 / 45 / 60** min; primary choropleth uses config default **45** min.
  - **Equity diagnostic:** Spearman ρ ≈ **0.467** between `jobs_C000_45min` and `disadvantage_z` (positive → more disadvantaged tracts reach more jobs at 45 min; **F020**).
- [x] **2026-04-03** — **`notebooks/04_bayesian_model.ipynb` — BYM2 overhaul + re-run** (standardised `log1p` jobs at 45 min, Riebler-scaled ICAR, back-transformed exports). Artifacts: `data/processed/posteriors/2026-04-03_*`, `artifacts/tables/pipeline__04_model_diagnostics__2026-04-03.csv`, `pipeline__04_equity_spearman__2026-04-03.csv`, trace + maps under `artifacts/figures/pipeline__04_*`.
  - **Historical note:** That run used **estimated** `sigma_obs` and shorter tuning; **`sigma` / `sigma_obs` mixed poorly** and had many divergences — documented as **F022** (archive). **Superseded** by **2026-04-05+** settings (**F025**).
- [x] **Semantic run ids** — **`fit_raw_zscore_x`** / **`fit_spatial_plus_x`** (`src.utils.config`, nb04/nb05 pin cells). **Two production fits** archived under those stems. **`configs/defaults.yaml`:** `draws: 8000`, `tune: 4000`, `target_accept: 0.99`, Student-t ν=4, **`obs_noise: fixed`**, `beta_sigma: 0.3`, ADVI init. **Four** covariates: `disadvantage_z`, `no_vehicle_hh_rate`, `log_median_income`, `log_pop_density`.
  - **`fit_raw_zscore_x`** — BYM2 on **z-scored raw X**: **0 divergences**; **ρ**/**σ** R-hat ≈ 1.00, ESS_bulk ~6k–7k; **β[disadvantage_z]** mean **+0.10** (wide HDI). Spearman posterior mean vs disadvantage **≈ 0.47**; exceedance vs disadvantage **≈ −0.47** (**F024**, **F025**).
  - **`fit_spatial_plus_x`** — **Spatial+**: **β[disadvantage_z]** **≈ 0**; **σ** larger; **Moran's I** of posterior-mean residuals **≈ 0.55** (**F026**). Notebook default **~5%** `k_remove`; **`draws` ≥ 10k** when Spatial+ on; **`PIPELINE_NO_SPATIAL_PLUS=1`** for **primary** raw **X** (**D011**).
  - **F021** CSV (04-05) still describes an **older seven-predictor** block in the same notebook era; see **F021** note + **F024** for the **live** coefficient story.
- [x] **2026-04-08** — **`05_posterior_analysis`:** generator adds KDE **fan** (jobs), **Wasserstein** map+CSV, **multi-threshold** table, **hook** candidates; **Fig 8** → nb05 per **D003**.
- [x] **2026-04-17** — **`07_intervention_simulation` run to completion** (all 10 sections, primary estimand `fit_raw_zscore_x`; see **F031**). The notebook closes the pipeline and produces the two remaining paper figures (Fig 1 + Fig 9) plus the dashboard GeoJSON bundle.
  - **Scenario A (Bayesian top-20 by exceedance)** vs **Scenario B (deterministic bottom-20 by det_jobs)**: overlap of 6/20. Parametric Δ calibrated to close 50% of the log1p(jobs) gap to the well-served median, anchored on Spearman(headway, det_jobs) = −0.53 from MTS GTFS.
  - **Headline result: 12 threshold crossings (A) vs 8 (B); 1.99% vs 1.08% population-weighted reach → Bayesian targeting ≈ 50% more efficient per intervened tract**. **11 of the 12 crossings under A are spillover-induced** on queen-contiguous, boundary-ambiguous neighbours — the policy signal is in the marginal tracts, not the extreme-desert targets.
  - **Hook tract (06073013317, not targeted)**: baseline exceedance 0.50 → 0.027 under Scenario A, Δ = −0.47 (95% CI [−0.478, −0.467]), `prob_improve = 1.00`. Spillover alone flips the coin flip to confident adequacy.
  - **Det-vs-Bayes classification**: **178 TP, 2 FP, 0 FN, 540 TN** — det and Bayes *agree on who is a desert*; they disagree on *priority ordering among the 178*. Reframes the paper's claim #4 from "Bayesian reveals hidden deserts" to "Bayesian re-ranks priority + exposes boundary-ambiguous intervention-responsive tracts".
  - **Artifacts**: 9 tables under `artifacts/tables/pipeline/pipeline__07_*__fit_raw_zscore_x.csv` (summary, intervention_targets, gtfs_coverage_targets, calibration, intervention_strength_sensitivity, intervention_posterior, equity_impact, hook_tract_intervention, det_vs_bayes_classification); 2 figures (`pipeline__07_study_area_map.png`, `pipeline__07_intervention_fig9__fit_raw_zscore_x.png`); 3 GeoJSONs + dashboard_manifest.json under `data/processed/geojson/` (INTERFACES.md D002/D007 contract).
  - **CI-column NaN note**: the summary table's `ci_lower` / `ci_upper` columns are empty on every row except the hook tract — **by design**. Per-tract 95% bootstrap CIs live in `pipeline__07_intervention_posterior__fit_raw_zscore_x.csv` (zero NaNs there); the summary table reserves the CI columns but most of its metrics are scalar point estimates (counts, Spearman-on-posterior-means, percentages) that do not carry a bootstrap CI in this framework.
- [x] **2026-04-17** — **`06_equity_decomposition` run to completion** (all 11 sections, primary estimand `fit_raw_zscore_x`; see **F028, F029, F030**, supersedes **D012** via **D013**). Notebook repurposed from the originally-planned "divergence metrics" (deprecated per D012, already covered by nb05) to a comprehensive equity decomposition + model validation stage:
  - **Multi-destination equity (F028):** Jobs/Hospitals/Groceries/Schools at 30/45/60 min all pro-poor (Spearman range **+0.35 → +0.58**, groceries strongest); composite-deficit set (all 4 destinations simultaneously > 50% desert probability) = **120 tracts**, **80% affluent** (mean disadvantage_z = −0.32). Q4 disadvantaged quartile has **2.8× more jobs** on average than Q1 and a **10× lower** desert-exceedance rate (1.7% vs 17.8%).
  - **Robustness (F030):** equity Spearman = **0.4699** exactly across 9 prior configs (`beta_sigma × nu`) and 4 drop-one covariate specs — equity result is invariant to prior and covariate set.
  - **Diagnostics concerns (F029):** PPC Bayesian-p ∈ {0, 1} extremes on mean/sd/min/max (fixed `obs_sigma=0.05` artefact); PSIS-LOO **catastrophic** — 100% of tracts k > 0.7, 82% > 1.0 (elpd = −524 uninterpretable; documented pathology for per-tract spatial random effects); raw-X residual Moran's I = **0.6432** (p=0.001) — **higher** than Spatial+ (0.5724), contradicting D011's prediction. None of these block nb07 (intervention sim draws directly from converged posterior draws), but Methods §6 needs honest framing.
- [x] **2026-04-08** — **`05_posterior_analysis` run to completion** (all 23 cells, primary estimand `fit_raw_zscore_x`). All 7 paper figures produced. Key results:
  - **Rank stability**: Spearman(rank_det, rank_bayes) = **0.9999758**; max rank delta = **±13** out of 720 tracts. No rank reversals — Bayesian value is uncertainty quantification, not reordering.
  - **Primary equity Spearman**: posterior_mean_jobs vs disadvantage_z = **+0.4699** (p=8×10⁻⁴¹); exceedance_prob_45min vs disadvantage_z = **−0.4669** (p=3×10⁻⁴⁰).
  - **Uncertainty null finding**: posterior_sd ρ = −0.045 (p=0.23, NS); ci_width ρ = −0.059 (p=0.11, NS) — uncertainty is NOT larger in disadvantaged neighborhoods.
  - **Multi-threshold (30/45/60 min)**: det jobs ρ weakens (+0.484 → +0.414); exceedance ρ strengthens (−0.301 → −0.481) — equity gradient is threshold-dependent in exceedance metric.
  - **Perfect paper hook**: GEOID **06073013317** — det_jobs=4,467 (just above Q25), exceedance_prob=**0.4998** (coin flip), disadvantage_z=−0.476. "The spreadsheet says fine; the model says we genuinely don't know."
  - **Wasserstein range**: 13,500 (dense urban core) → 86,000+ (suburban periphery). Outlier: GEOID 06073005801 (high disadvantage, high mean) W=61,515 — distributional dissimilarity despite above-average expected access.
  - **Figures**: Fig 3 (posterior mean), Fig 4 (exceedance), Fig 5 (CI width), Fig 6 (KDE fan — NEW), Fig 7 (rank divergence), Fig 8 (Wasserstein — NEW). Fig 9 awaits nb07.
  - See **F027** in `context/FINDINGS.md` for full interpretation.

## ✅ nb04 Model Overhaul (landed 2026-04-03; production MCMC 2026-04-05+)
Implemented: `rho ~ Beta(2,2)`, PC-style `sigma`, standardised **y**, fixed or estimated observation noise (currently **fixed** in defaults), `src/modeling/spatial.py` id_order fix, `tract_bym.py` rebuild, notebook diagnostics + Spearman + maps. **Outcome (current):** **F025** — global variance and **ρ** mix well under **fixed** `obs_noise` + long sampling; **F024** — side-by-side **raw X** vs **Spatial+** for interpretation.

## What Is Next
1. **Write the paper** — all pipeline notebooks complete; Results can now be drafted from F020, F024, F025, F026, F027, **F028, F029, F030, F031**. Suggested section order:
   - **Introduction / Hook**: GEOID 06073013317 (F027d) — deterministic "adequate" vs Bayesian "coin flip" — now reinforced by F031's intervention result (spillover-only flip from 0.50 → 0.03 with full posterior certainty).
   - **Results §5.1 (Rank stability)**: F027c (Spearman 0.9999758, max delta ±13). Bayesian contribution is uncertainty, not reordering.
   - **Results §5.2 (Equity gradient)**: F024 / F027a (posterior mean ρ = +0.47, exceedance ρ = −0.47), then F027b for multi-threshold asymmetry.
   - **Results §5.3 (Multi-destination generalisation)**: F028 (Jobs / Hospitals / Groceries / Schools pro-poor at every threshold; composite deficit → 120 tracts, 80% affluent).
   - **Results §5.4 (Intervention simulation — NEW, F031)**: 12 vs 8 crossings (Bayesian vs deterministic targeting); 11/12 spillover-induced; hook tract 0.50 → 0.03. Reframe claim #4 to **"different priority ordering + ambiguous-boundary intervention responsiveness"**, not "hidden desert identification" (FN = 0).
   - **Results §5.5 (Wasserstein)**: F027e — range 13,500–86,000; outlier 06073005801.
   - **Results §5.6 (Robustness)**: F030 — equity Spearman invariant across priors and covariate drops.
   - **Methods §6 (Model validation — honest limitations)**: F029 framed as fixed-obs-sigma + PSIS-on-spatial-random-effects pathology, not misfit. Spatial+ sensitivity comparison.
   - **Fig 1**: `pipeline__07_study_area_map.png` (F031); **Fig 9**: `pipeline__07_intervention_fig9__fit_raw_zscore_x.png` (F031). All 9 paper figures now exist.
2. **Dashboard / frontend (`app/` Next.js)** — `data/processed/geojson/sd_tracts_equity_baseline.geojson` + `scenarios/freq_double_bayesian_top20.geojson` + `scenarios/freq_double_det_top20.geojson` + `dashboard_manifest.json` are all produced to the INTERFACES.md contract (D002 / D007). Wire the baseline layer into `app/components/Map/` and the scenario toggle into `app/components/InterventionSlider/`.
3. **Optional supplementary analyses** (reviewer-proof, none blocking submission):
   - **Spillover weight sensitivity** (F031 next question a): re-run nb07 with `SPILLOVER_WEIGHT ∈ {0, 0.05, 0.10, 0.15}` and export a crossings-vs-weight table. The 11/12 spillover-induced-crossings finding is the most reviewer-exposed number.
   - **Budget sensitivity** (F031 next question b): re-run at N_INTERVENTION_TRACTS ∈ {10, 40} to show efficiency A-vs-B ordering is robust.
   - **Optional diagnostic refit** (F029): one sensitivity MCMC with `obs_noise: estimated`, `sigma_obs ~ HalfNormal(0.2)` — confirms PPC-p moves off {0, 1} and whether residual Moran's I drops. ~80 min MCMC.
   - **~~nb06 DEPRECATED~~ — SUPERSEDED by D013 (2026-04-17):** nb06 was repurposed as `06_equity_decomposition.ipynb` (now complete).