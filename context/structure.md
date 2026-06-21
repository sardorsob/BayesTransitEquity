# structure.md
Single source of truth for repository structure, artifact index, and results log.
All referenced paths are relative to repo root.

---

## PROJECT MAP

```
BayesTransitEquity/
├── README.md                         Project entry point
├── development_rules.md              Operating contract for this repo
├── structure.md                      THIS FILE — repo map + artifact index + results
├── DECISIONS.md                      Architectural + modeling decisions with rationale
├── ASSUMPTIONS.md                    Modeling and data assumptions
├── CHANGELOG.md                      Milestones
├── requirements.txt                  Python dependencies
├── .gitignore
│
├── context/                          Long-running memory for AI + human
│   ├── AGENT.md                      Read first in every new session
│   ├── PROJECT_BRIEF.md              What, why, success criteria
│   ├── GLOSSARY.md                   Domain terms + acronyms
│   ├── DATASETS.md                   Data sources, schemas, licensing
│   ├── INTERFACES.md                 I/O contracts and GeoJSON schema
│   ├── STATUS.md                     Current state, what's next
│   ├── RISKS.md                      Known risks + mitigations
│   ├── PAPER.md                      Paper argument + figure plan
│   └── FINDINGS.md                   Empirical results log with interpretation (updated each notebook run)
│
├── configs/
│   ├── defaults.yaml                 Shared defaults (accessibility thresholds, model settings)
│   ├── san_diego.yaml                SD-specific: bbox, agencies, census FIPS
│   └── ASSUMPTIONS.md                Modeling + data assumptions (A001–A009)
│
├── data/
│   ├── raw/                          Immutable. Gitignored. Never modify.
│   │   ├── gtfs/mts/                 MTS GTFS feed
│   │   ├── gtfs/nctd/                NCTD GTFS feed
│   │   ├── census/                   Tract shapefiles + ACS demographics
│   │   └── osm/                      OSM pedestrian walk network
│   ├── interim/                      Cached transforms. Always gitignored.
│   └── processed/
│       ├── accessibility/            Computed accessibility metrics per tract
│       ├── posteriors/               MCMC posterior samples (Parquet). Gitignored.
│       └── geojson/                  Frontend-ready GeoJSON. Consumed by app/.
│
├── notebooks/
│   ├── eda/                          Exploratory EDA only (see eda/README.md)
│   │   ├── README.md                 Overview + sequencing; avoids numbering clash with pipeline
│   │   ├── 01_inventory_and_provenance.ipynb
│   │   ├── 02_gtfs_schedule_exploration.ipynb
│   │   ├── 03_census_tracts_and_acs.ipynb
│   │   ├── 04_spatial_alignment_and_coverage.ipynb
│   │   ├── 05_osm_pedestrian_network.ipynb
│   │   ├── 06_opportunities_and_destinations.ipynb
│   │   └── 07_cross_source_sanity_joins.ipynb
│   ├── 01_data_exploration.ipynb     Pipeline entry after EDA; bridge raw → processed exploration
│   ├── 02_gtfs_processing.ipynb      Clean, validate, join GTFS data
│   ├── 03_accessibility_computation.ipynb  Cumulative opportunity metrics via r5py
│   ├── 04_bayesian_model.ipynb       Hierarchical Bayesian transit desert model (PyMC)
│   ├── 05_posterior_analysis.ipynb   Posteriors, exceedance probabilities, CIs
│   ├── 06_equity_decomposition.ipynb Gini/CI, subgroup, multi-destination, composite deficit, PPC, LOO, prior sensitivity, Moran (F028/F029/F030)
│   └── 07_intervention_simulation.ipynb   Counterfactual inference — route change posteriors
│
├── src/                              Reusable Python package
│   ├── io/                           Data loaders (GTFS, Census, OSM)
│   ├── preprocessing/                Cleaning + spatial joins
│   ├── accessibility/                Accessibility metric computation (r5py wrappers)
│   ├── modeling/                     Bayesian model definitions (PyMC)
│   ├── metrics/                      Divergence, entropy, equity metrics
│   ├── evaluation/                   MCMC diagnostics + posterior predictive checks
│   ├── viz/                          Plotting + GeoJSON export helpers
│   └── utils/                        Paths, seeds, logging
│
├── scripts/
│   ├── download_data.py              Fetch GTFS + census + OSM data
│   ├── run_pipeline.py               End-to-end pipeline runner
│   ├── run_model.py                  Run Bayesian model, produce posterior samples
│   └── export_frontend.py           Convert posteriors → GeoJSON for app/
│
├── app/                              Next.js Vercel frontend (standalone project)
│   ├── components/Map/               Deck.gl interactive map
│   ├── components/PosteriorPanel/    Distribution visualization panel
│   ├── components/DivergenceView/    Wasserstein / entropy comparison view
│   ├── components/InterventionSlider/ Scenario simulation UI
│   └── public/data/                  Static GeoJSON served by Vercel
│
├── artifacts/                        ALL generated outputs — never hand-edited
│   ├── figures/                      Exported plots (PNG, dpi=200+)
│   ├── tables/                       Exported tables (CSV / HTML)
│   ├── models/                       Serialized PyMC traces + posteriors
│   ├── reports/                      Auto-generated markdown/HTML reports
│   └── logs/
│       ├── runs/                     Per-run metadata bundles
│       ├── prompts/                  LLM prompt/response snapshots
│       └── provenance/               Input hashes + manifests
│
├── paper/                            Academic paper
│   ├── main.md                       Master document
│   ├── sections/                     Individual section files
│   ├── figures/                      Paper-specific figure exports
│   └── references.bib
│
└── tests/                            Unit tests for core logic
```

---

## ARTIFACT INDEX

### Figures
| Path | Date | Purpose | Source notebook | Finding |
|------|------|---------|----------------|---------|
| `artifacts/figures/eda__gtfs_stops_map__2026-03-29.png` | 2026-03-29 | GTFS stop locations (MTS + NCTD) vs study bbox | eda/02 | F002 |
| `artifacts/figures/eda__acs_moe_vs_disadvantage__2026-03-29.png` | 2026-03-29 | Scatter: composite disadvantage_z vs population relative MOE, colored by Hispanic share | eda/03 | F007 |
| `artifacts/figures/eda__acs_choropleth_rel_moe_pop__2026-03-29.png` | 2026-03-29 | Choropleth: population estimate relative MOE across SD tracts (TIGER geometry) | eda/03 | F007, F008 |
| `artifacts/figures/eda__tract_stops_choropleth__2026-03-29.png` | 2026-03-29 | SD tracts colored by in-bbox GTFS stop count; bbox + stop overlay | eda/04 | F012, F013 |
| `artifacts/figures/eda__tract_nearest_stop_distance__2026-03-29.png` | 2026-03-29 | Tract centroid → nearest in-bbox stop distance (m); study bbox | eda/04 | F012, F013 |
| `artifacts/figures/eda__osm_tract_walk_density_choropleth__2026-03-29.png` | 2026-03-29 | Tract choropleth: OSM walk edge length density (km/km²); centroid proxy | eda/05 | F014 |
| `artifacts/figures/eda__osm_walk_network_sample_map__2026-03-29.png` | 2026-03-29 | Sample of walk edges (EPSG:3310) + study bbox | eda/05 | F014 |
| `artifacts/figures/eda__tract_stops_choropleth__2026-03-30.png` | 2026-03-30 | SD tracts colored by in-bbox GTFS stop count; expanded bbox (726 tracts) | eda/04 | F012, F013 |
| `artifacts/figures/eda__tract_nearest_stop_distance__2026-03-30.png` | 2026-03-30 | Tract centroid → nearest in-bbox stop distance (m); expanded bbox | eda/04 | F013, F014 |
| `artifacts/figures/eda__osm_tract_walk_density_choropleth__2026-03-30.png` | 2026-03-30 | Tract choropleth: OSM walk edge density (km/km²); 337,824-node network | eda/05 | F015 |
| `artifacts/figures/eda__osm_walk_network_sample_map__2026-03-30.png` | 2026-03-30 | Sample walk edges (EPSG:3310) + expanded study bbox | eda/05 | F015 |
| `artifacts/figures/eda__osm_destinations_map__2026-03-30.png` | 2026-03-30 | Map of OSM destination points (hospitals, groceries, schools) in expanded bbox | eda/06 | F016 |
| `artifacts/figures/pipeline__03_accessibility_choropleth__2026-04-03.png` | 2026-04-03 | Choropleth: LODES jobs reachable within 45 min (`jobs_C000_45min`), in-bbox tracts | notebooks/03 | F020 |
| `artifacts/figures/pipeline__06_lorenz_curve__fit_raw_zscore_x.png` | 2026-04-17 | Lorenz curve of posterior-mean jobs (population-weighted); Gini = 0.1445 | notebooks/06 | F028 |
| `artifacts/figures/pipeline__06_subgroup_forest_plot__fit_raw_zscore_x.png` | 2026-04-17 | Forest plot: posterior mean jobs by disadvantage quartile + vehicle-ownership subgroup | notebooks/06 | F028 |
| `artifacts/figures/pipeline__06_multidestination_spearman_heatmap__fit_raw_zscore_x.png` | 2026-04-17 | Heatmap: Spearman × {Jobs, Hospitals, Groceries, Schools} × {30, 45, 60} min | notebooks/06 | F028 |
| `artifacts/figures/pipeline__06_composite_deficit_map__fit_raw_zscore_x.png` | 2026-04-17 | Choropleth: per-tract composite deficit score across 4 destinations | notebooks/06 | F028 |
| `artifacts/figures/pipeline__06_multidesert_map__fit_raw_zscore_x.png` | 2026-04-17 | Choropleth: number of destinations each tract is in desert status for (0-4) | notebooks/06 | F028 |
| `artifacts/figures/pipeline__06_ppc_overall__fit_raw_zscore_x.png` | 2026-04-17 | Posterior-predictive check overall — mean/sd/frac_below_q25 of y_rep vs y | notebooks/06 | F029 |
| `artifacts/figures/pipeline__06_ppc_subgroup__fit_raw_zscore_x.png` | 2026-04-17 | Posterior-predictive check by disadvantage quartile | notebooks/06 | F029 |
| `artifacts/figures/pipeline__06_loo_pointwise_map__fit_raw_zscore_x.png` | 2026-04-17 | Choropleth: per-tract Pareto-k; flags 720/720 tracts as k > 0.7 | notebooks/06 | F029 |
| `artifacts/figures/pipeline__06_prior_sensitivity_beta__fit_raw_zscore_x.png` | 2026-04-17 | β[disadvantage_z] across 9 (β prior × Student-t df) cells — near-flat | notebooks/06 | F030 |
| `artifacts/figures/pipeline__07_study_area_map.png` | 2026-04-17 | **Fig 1** — SD county tracts + 4,373 MTS stops (blue) + 1,823 NCTD stops (orange) + hook tract 06073013317 highlighted | notebooks/07 | F031 |
| `artifacts/figures/pipeline__07_intervention_fig9__fit_raw_zscore_x.png` | 2026-04-17 | **Fig 9** — 3-panel intervention effect: baseline exceedance / Δ exceedance under Bayesian targeting (★ = 12 threshold crossings, ○ = 14 det-only targets) / efficiency bar chart (12 vs 8) | notebooks/07 | F031 |

| `artifacts/figures/eda__nearest_stop_distance_hist__2026-03-30.png` | 2026-03-30 | Histogram of nearest stop distance (m) for all 737 tracts | eda/07 | F019 |
| `artifacts/figures/eda__tracts_zero_or_few_stops_within_0p5km__2026-03-30.png` | 2026-03-30 | Map of in-bbox tracts flagged as zero/few stops within 0.5 km threshold | eda/07 | F019 |
### Tables
| Path | Date | Purpose | Source notebook | Finding |
|------|------|---------|----------------|---------|
| `artifacts/tables/eda__file_inventory__2026-03-29.csv` | 2026-03-29 | Full recursive listing of data/raw/ + mobility catalog cache | eda/01 | F006 |
| `artifacts/tables/eda__gtfs_feed_info__2026-03-29.csv` | 2026-03-29 | Feed version, validity window, MD5, extraction count per agency | eda/01 | F001 |
| `artifacts/tables/eda__manifest_summary__2026-03-29.csv` | 2026-03-29 | Download manifest metadata (city, freeze date, n_sources) | eda/01 | — |
| `artifacts/tables/eda__census_artifact_check__2026-03-29.csv` | 2026-03-29 | Existence check for TIGER zip, extracted dir, ACS JSON | eda/01 | F006 |
| `artifacts/tables/eda__gtfs_feed_summary__2026-03-29.csv` | 2026-03-29 | Routes / trips / stops / stop_times row counts per agency | eda/02 | F003 |
| `artifacts/tables/eda__gtfs_bbox_coverage__2026-03-29.csv` | 2026-03-29 | Stop count inside study bbox per agency (n + %) | eda/02 | F002 |
| `artifacts/tables/eda__gtfs_calendar__2026-03-29.csv` | 2026-03-29 | Service ID counts + weekday IDs + date range per agency | eda/02 | F001 |
| `artifacts/tables/eda__gtfs_headways__2026-03-29.csv` | 2026-03-29 | Median weekday headway by route + direction (both agencies); best MTS 5 min, worst 70 min | eda/02 | F004 |
| `artifacts/tables/eda__gtfs_data_quality__2026-03-29.csv` | 2026-03-29 | Data quality flags: orphan trips, zero-trip routes, unused stops per agency | eda/02 | F005 |
| `artifacts/tables/eda__acs_sd_tract_attributes__2026-03-29.csv` | 2026-03-29 | Full 737-tract table: ACS estimates + MOEs + poverty/vehicle/race rates + disadvantage_z | eda/03 | F008, F009, F010 |
| `artifacts/tables/eda__acs_tiger_alignment__2026-03-29.csv` | 2026-03-29 | ACS/TIGER row counts; confirms 737/737/737 match — no unmatched GEOIDs | eda/03 | F011 |
| `artifacts/tables/eda__acs_moe_disadvantage_spearman__2026-03-29.csv` | 2026-03-29 | Spearman rho + p-values for six MOE vs disadvantage/race pairings (n=683-735) | eda/03 | F007 |
| `artifacts/tables/eda__acs_summary_stats__2026-03-29.csv` | 2026-03-29 | Count/mean/std/quartiles for all ACS + derived numeric columns (737 tracts) | eda/03 | F008, F009, F010 |
| `artifacts/tables/eda__spatial_layers_crs__2026-03-29.csv` | 2026-03-29 | CRS manifest: TIGER=EPSG:4269, bbox/GTFS=EPSG:4326, analysis=EPSG:3310 | eda/04 | F012 |
| `artifacts/tables/eda__tract_gtfs_service__2026-03-29.csv` | 2026-03-29 | Per-tract GTFS: stop counts (MTS/NCTD), route-direction counts, headway stats, dist to nearest stop, ACS merge | eda/04 | F012, F013 |
| `artifacts/tables/eda__osm_walk_graph_summary__2026-03-29.csv` | 2026-03-29 | Node/edge counts, total walk edge length (km), weakly connected components | eda/05 | F014 |
| `artifacts/tables/eda__osm_walk_component_sizes__2026-03-29.csv` | 2026-03-29 | Full distribution of weakly connected component sizes (sorted desc.) | eda/05 | F014 |
| `artifacts/tables/eda__osm_tract_walk_edges__2026-03-29.csv` | 2026-03-29 | Per tract: OSM walk edge count + length + density; merge from `eda__tract_gtfs_service__` | eda/05 | F014 |
| `artifacts/tables/eda__osm_walk_disadvantage_spearman__2026-03-29.csv` | 2026-03-29 | Spearman: walk edge density vs `disadvantage_z` (n=737) | eda/05 | F014 |
| `artifacts/tables/eda__spatial_layers_crs__2026-03-30.csv` | 2026-03-30 | CRS manifest (expanded bbox): TIGER=EPSG:4269, bbox/GTFS=EPSG:4326, analysis=EPSG:3310 | eda/04 | F012 |
| `artifacts/tables/eda__tract_gtfs_service__2026-03-30.csv` | 2026-03-30 | Per-tract GTFS: stop counts (MTS/NCTD/total), headway stats, nearest-stop distance; 726 in-bbox tracts | eda/04 | F012, F013, F014 |
| `artifacts/tables/eda__osm_walk_graph_summary__2026-03-30.csv` | 2026-03-30 | OSM walk network stats: 337,824 nodes, 906,988 edges, 65,750 km, 1 component (expanded bbox) | eda/05 | F015 |
| `artifacts/tables/eda__osm_walk_component_sizes__2026-03-30.csv` | 2026-03-30 | Single component of 337,824 nodes confirmed (no second component) | eda/05 | F015 |
| `artifacts/tables/eda__osm_tract_walk_edges__2026-03-30.csv` | 2026-03-30 | Per tract: OSM walk edge count + length + density (km/km²); 737 tracts (expanded bbox) | eda/05 | F015 |
| `artifacts/tables/eda__osm_walk_disadvantage_spearman__2026-03-30.csv` | 2026-03-30 | Spearman: walk edge density vs `disadvantage_z`; ρ=+0.371 (n=737, expanded bbox) | eda/05 | F015 |
| `artifacts/tables/eda__opportunities_destination_summary__2026-03-30.csv` | 2026-03-30 | OSM destination counts: 181 hospitals, 1,170 groceries, 1,273 schools in expanded bbox | eda/06 | F016 |
| `artifacts/tables/eda__osm_destinations_fetch_status__2026-03-30.csv` | 2026-03-30 | Overpass fetch status per category (all "ok") for expanded bbox | eda/06 | F016 |
| `artifacts/tables/eda__osm_destinations_groceries__2026-03-30.csv` | 2026-03-30 | Raw OSM grocery features (1,170 points/polygons) in expanded bbox | eda/06 | F016 |
| `artifacts/tables/eda__osm_destinations_hospitals__2026-03-30.csv` | 2026-03-30 | Raw OSM hospital features (181 points/polygons) in expanded bbox | eda/06 | F016 |
| `artifacts/tables/eda__osm_destinations_schools__2026-03-30.csv` | 2026-03-30 | Raw OSM school features (1,273 points/polygons) in expanded bbox; no longer capped | eda/06 | F016 |
| `artifacts/tables/eda__tract_osm_destination_counts__2026-03-30.csv` | 2026-03-30 | Per-tract destination counts: n_osm_hospitals, n_osm_groceries, n_osm_schools (737 tracts) | eda/06 | F016, F017 |
| `artifacts/tables/eda__lodes_wac_run_status__2026-03-30.csv` | 2026-03-30 | LODES WAC download status: file_not_found — download still needed | eda/06 | — |

| `artifacts/tables/eda__cross_source_join_diagnostics__2026-03-30.csv` | 2026-03-30 | Row counts per source layer confirming 737/737 GEOID match across all four datasets | eda/07 | F018 |
| `artifacts/tables/eda__tracts_zero_or_few_stops_summary__2026-03-30.csv` | 2026-03-30 | 726 in-bbox tracts; 98 with zero stops within 0.5 km; 134 with few stops; x_km=0.5 | eda/07 | F019 |
| `artifacts/tables/eda__tracts_zero_or_few_stops_within_0p5km__2026-03-30.csv` | 2026-03-30 | Per-tract flag table: zero_stops_within_x_km, few_stops_within_x_km, dist_nearest_stop, disadvantage_z join | eda/07 | F019 |
| `artifacts/tables/pipeline__03_accessibility_summary__2026-04-03.csv` | 2026-04-03 | Pipeline 03 manifest: Parquet paths, n_tracts=726, Spearman `jobs_C000_45min` vs `disadvantage_z`, thresholds | notebooks/03 | F020 |
| `artifacts/tables/pipeline__06_summary__fit_raw_zscore_x.csv` | 2026-04-17 | Master roll-up of all 11 nb06 sections (Gini, CI, multi-destination Spearman, composite-deficit n, PPC flags, Moran) | notebooks/06 | F028, F029 |
| `artifacts/tables/pipeline__06_gini_ci_equity__fit_raw_zscore_x.csv` | 2026-04-17 | Gini + Concentration Index (posterior + deterministic) vs disadvantage_z | notebooks/06 | F028 |
| `artifacts/tables/pipeline__06_subgroup_posterior_summary__fit_raw_zscore_x.csv` | 2026-04-17 | Posterior-mean jobs + frac(exceedance > 0.5) by disadvantage quartile + vehicle-ownership subgroup | notebooks/06 | F028 |
| `artifacts/tables/pipeline__06_multidestination_equity__fit_raw_zscore_x.csv` | 2026-04-17 | Spearman × {Jobs, Hospitals, Groceries, Schools} × {30, 45, 60} min vs disadvantage_z (12 rows) | notebooks/06 | F028 |
| `artifacts/tables/pipeline__06_composite_deficit_ranked__fit_raw_zscore_x.csv` | 2026-04-17 | Per-tract composite deficit score across 4 destinations (720 rows) ranked desc | notebooks/06 | F028 |
| `artifacts/tables/pipeline__06_multidesert_tracts__fit_raw_zscore_x.csv` | 2026-04-17 | Binary desert flags per destination + n_deserts + multi_desert indicator (720 rows) | notebooks/06 | F028 |
| `artifacts/tables/pipeline__06_ppc_bayesian_pvalues__fit_raw_zscore_x.csv` | 2026-04-17 | Bayesian p-values for mean/sd/skewness/min/max/frac_below_q25 test statistics | notebooks/06 | F029 |
| `artifacts/tables/pipeline__06_loo_comparison__fit_raw_zscore_x.csv` | 2026-04-17 | elpd_loo + SE for fit_raw_zscore_x (single-model row; Spatial+ not refit in nb06) | notebooks/06 | F029 |
| `artifacts/tables/pipeline__06_pareto_k_diagnostics__fit_raw_zscore_x.csv` | 2026-04-17 | Per-tract Pareto k from PSIS-LOO (720 rows); 100% k > 0.7, 82% k > 1.0 (catastrophic) | notebooks/06 | F029 |
| `artifacts/tables/pipeline__06_prior_sensitivity__fit_raw_zscore_x.csv` | 2026-04-17 | Importance-reweighted β[disadv], equity Spearman across β-prior × Student-t df grid (9 rows) | notebooks/06 | F030 |
| `artifacts/tables/pipeline__06_drop_one_covariate__fit_raw_zscore_x.csv` | 2026-04-17 | Equity Spearman + β[disadv] under each drop-one covariate spec (5 rows) | notebooks/06 | F030 |
| `artifacts/tables/pipeline__06_moran_comparison__fit_raw_zscore_x.csv` | 2026-04-17 | Moran's I on posterior-mean residuals, raw X vs Spatial+ (I=0.6432 vs 0.5724) | notebooks/06 | F029 |
| `artifacts/tables/pipeline/pipeline__06_wasserstein_equity_metrics__fit_raw_zscore_x.csv` | 2026-04-17 | Spearman(Wasserstein, disadvantage_z) = −0.360; Gini + CI on Wasserstein across tracts | notebooks/06 | F028 |
| `artifacts/tables/pipeline/pipeline__06_wasserstein_by_disadv_quartile__fit_raw_zscore_x.csv` | 2026-04-17 | Wasserstein mean/median/p25/p75 + pop-weighted mean by disadvantage quartile (4 rows) | notebooks/06 | F028 |
| `artifacts/tables/pipeline/pipeline__06_inputs_index__fit_raw_zscore_x.csv` | 2026-04-17 | nb06 input-file index (nb05 artefacts + ACS table paths) — reproducibility manifest | notebooks/06 | F028 |
| `artifacts/tables/pipeline/pipeline__07_summary__fit_raw_zscore_x.csv` | 2026-04-17 | **nb07 master roll-up** — targeting (overlap, FP/FN), intervention (crossings A/B, pop-weighted reach), equity (Spearman baseline / A / B + Δρ), classification (TP=178, FP=2, FN=0, TN=540), hook tract 0.50→0.03 with CI, calibration headway↔jobs ρ=−0.53. `ci_lower`/`ci_upper` populated only on hook row — by design; per-tract CIs in intervention_posterior CSV | notebooks/07 | F031 |
| `artifacts/tables/pipeline/pipeline__07_intervention_targets__fit_raw_zscore_x.csv` | 2026-04-17 | Bayesian top-20 by exceedance + deterministic bottom-20 by det_jobs (40 rows); overlap=6; Bayesian-only=14; det-only=14 | notebooks/07 | F031 |
| `artifacts/tables/pipeline/pipeline__07_gtfs_coverage_targets__fit_raw_zscore_x.csv` | 2026-04-17 | Per-target tract: n_routes, AM-peak min/median headway (min), trip count, current frequency tier — grounds parametric Δ in observed MTS schedule | notebooks/07 | F031 |
| `artifacts/tables/pipeline/pipeline__07_calibration__fit_raw_zscore_x.csv` | 2026-04-17 | Spearman(AM-peak headway, det_jobs) = −0.53; Q25 jobs = 4,470; Q25 log1p = 8.405; coverage = 20/20 targets with ≥1 MTS route within 400 m | notebooks/07 | F031 |
| `artifacts/tables/pipeline/pipeline__07_intervention_strength_sensitivity__fit_raw_zscore_x.csv` | 2026-04-17 | 3×2 grid over strength ∈ {0.25, 0.5, 0.75} × {Bayesian, deterministic}; median Δ_log1p and Δ_jobs per cell; monotone, preserves A-vs-B ordering | notebooks/07 | F031 |
| `artifacts/tables/pipeline/pipeline__07_intervention_posterior__fit_raw_zscore_x.csv` | 2026-04-17 | **720 rows** — per-tract baseline exceedance + scenario A/B exceedance + Δ + **95% bootstrap CI on Δ** (400 resamples × 32,000 draws) + `prob_improve` + `targeted_by_{A,B}` + `crossed_threshold_{A,B}`. **Zero NaNs on CI columns** | notebooks/07 | F031 |
| `artifacts/tables/pipeline/pipeline__07_equity_impact__fit_raw_zscore_x.csv` | 2026-04-17 | Spearman(exceedance, disadvantage) baseline −0.467 → A −0.463, B −0.460; Δρ tiny; crossings 12 vs 8; pop-weighted reach 1.99% vs 1.08%; efficiency 0.60 vs 0.40 per target | notebooks/07 | F031 |
| `artifacts/tables/pipeline/pipeline__07_hook_tract_intervention__fit_raw_zscore_x.csv` | 2026-04-17 | Hook 06073013317: baseline 0.500, scenario A 0.027, Δ = −0.472, CI [−0.478, −0.467], prob_improve = 1.0; not targeted by either rule — spillover-only | notebooks/07 | F031 |
| `artifacts/tables/pipeline/pipeline__07_det_vs_bayes_classification__fit_raw_zscore_x.csv` | 2026-04-17 | All 720 tracts × {det_desert, bayes_desert, classification ∈ TP/FP/FN/TN}; TP=178, FP=2, FN=0, TN=540 — **zero false negatives** reframes paper claim #4 | notebooks/07 | F031 |

### Datasets (processed)
| Path | Run ID | Purpose | Produced by |
|------|--------|---------|-------------|
| `data/processed/accessibility/tract_tract_od_traveltime__2026-04-03.parquet` | 2026-04-03 | Tract–tract travel times (r5py; elementwise min of transit+walk vs walk-only) | notebooks/03 |
| `data/processed/accessibility/tract_jobs_accessibility__2026-04-03.parquet` | 2026-04-03 | Per-tract LODES `C000` jobs reachable by threshold; columns `jobs_C000_{30,45,60}min`, origin jobs | notebooks/03 |
| `data/processed/accessibility/tract_poi_groceries_od__2026-04-03.parquet` | 2026-04-03 | Tract–destination OD for groceries (counts / times at 30/45/60 min) | notebooks/03 |
| `data/processed/accessibility/tract_poi_hospitals_od__2026-04-03.parquet` | 2026-04-03 | Tract–destination OD for hospitals | notebooks/03 |
| `data/processed/accessibility/tract_poi_schools_od__2026-04-03.parquet` | 2026-04-03 | Tract–destination OD for schools | notebooks/03 |
| `data/processed/accessibility/tract_poi_counts__2026-04-03.parquet` | 2026-04-03 | Per-tract rollup: reachable POI counts at 30/45/60 min by category | notebooks/03 |
| `data/processed/accessibility/tract_accessibility_bundle__2026-04-03.parquet` | 2026-04-03 | **Primary table for nb04:** merge of jobs accessibility + POI counts on `GEOID` | notebooks/03 |

An earlier run with the same schema exists dated **2026-04-01**; treat **2026-04-03** as the latest pipeline fingerprint unless a new `artifact_run_id()` is written.

### Dashboard GeoJSON (INTERFACES.md D002 / D007 contract, produced by nb07)
| Path | Generated | Schema / Purpose | Consumed by |
|------|-----------|------------------|-------------|
| `data/processed/geojson/sd_tracts_equity_baseline.geojson` | 2026-04-17 | **720 features.** Baseline tract layer: `geoid`, `tract_name`, `posterior_mean`, `posterior_sd`, `ci_lower_95`, `ci_upper_95`, `posterior_mean_jobs`, `exceedance_prob`, `p_transit_desert`, `wasserstein_dist`, `entropy`, `pop_total`, `pct_no_vehicle`, `median_income`, `pct_poverty`. Derived from nb05 posterior summary + nb05 Wasserstein table + ACS attributes | `app/components/Map/` baseline layer |
| `data/processed/geojson/scenarios/freq_double_bayesian_top20.geojson` | 2026-04-17 | **720 features.** Baseline schema + `exceedance_prob_scenario`, `exceedance_prob_delta`, `p_transit_desert_scenario`, `crossed_threshold`, `exceedance_delta_ci_lower`, `exceedance_delta_ci_upper`. Scenario A = 20 Bayesian-ranked targets, strength=0.5, spillover weight 0.1 | `app/components/InterventionSlider/` scenario toggle |
| `data/processed/geojson/scenarios/freq_double_det_top20.geojson` | 2026-04-17 | Same schema as Scenario A; Scenario B = 20 deterministic-ranked targets (bottom-20 by det_jobs) | `app/components/InterventionSlider/` comparison mode |
| `data/processed/geojson/dashboard_manifest.json` | 2026-04-17 | Top-level manifest: `run_id`, `generated`, baseline + scenario paths, `n_threshold_crossings` (12 vs 8), `efficiency_ratio_vs_B` = 1.5, `key_tracts = [06073013317]` with baseline + scenario A exceedance values | `app/` entry point; read once on page load |

### Models
*(none yet — begins at `notebooks/04_bayesian_model.ipynb`)*

### Results log — `notebooks/03_accessibility_computation.ipynb` (2026-04-03)

- **Inputs:** `data/processed/gtfs/sd_merged_bbox.zip`, OSM PBF from `configs/san_diego.yaml` (`r5.osm_pbf`), LODES WAC `data/raw/external/lodes/ca_wac_2021.csv.gz`, latest EDA OSM destination CSVs under `artifacts/tables/`, TIGER tracts intersecting study bbox (**726** tract origins).
- **Settings:** Deep-merged `defaults.yaml` + `san_diego.yaml`; cumulative opportunity at **30, 45, 60** minutes; primary map column uses `accessibility.travel_time_threshold_min` (default **45**). Departure datetime from `r5.departure_date` / `r5.departure_hhmm` and morning window from `accessibility.departure_window_*`.
- **Routing:** r5py `TransportNetwork`; per notebook, **transit+walk** and **walk-only** matrices are combined with an **elementwise minimum** so walk-only paths count when transit is unavailable.
- **Outputs:** Parquets above; choropleth PNG; `pipeline__03_accessibility_summary__2026-04-03.csv`; OD checkpoints under `data/interim/accessibility_od/<run_id>/` (ephemeral, gitignored).
- **Key result (equity diagnostic):** Spearman ρ ≈ **0.467** between `jobs_C000_45min` and `disadvantage_z` (**positive**): tracts with higher composite disadvantage tend to reach **more** jobs within 45 minutes, consistent with the EDA “density confound” story carrying through to **realized schedule-based accessibility** for employment (see **F020**). **6** tracts have zero jobs reachable (including in-tract jobs); median jobs reachable ≈ **10,740**.
- **Caveats:** If `r5.departure_date` falls outside a merged agency’s GTFS calendar, r5py may warn (e.g. MTS validity vs NCTD staleness); treat as a **schedule alignment** caveat for absolute levels. Cross-sectional **ranking** comparisons remain informative but should mention feed validity in Data/Discussion (F001, NCTD).

### Results log — `notebooks/07_intervention_simulation.ipynb` (2026-04-17, primary estimand `fit_raw_zscore_x`)

- **Inputs:** `data/processed/posteriors/fit_raw_zscore_x_posterior_summary.parquet` (720 tracts) + `fit_raw_zscore_x_posterior_samples.parquet` (23M rows, pivoted to 720×32,000 matrix), `data/processed/accessibility/tract_accessibility_bundle__2026-04-03.parquet` (det_jobs, Q25 anchors), `data/raw/census/tl_2023_06_tract.zip` (tract geometries), `data/raw/gtfs/mts/extracted/{stops,stop_times,trips}.txt` (headway calibration + Fig 1 symbology), `data/raw/gtfs/nctd/extracted/stops.txt` (Fig 1 only), `artifacts/tables/eda/eda__acs_sd_tract_attributes__2026-03-29.csv` (disadvantage_z + pop_total), `artifacts/tables/pipeline/pipeline__05_wasserstein_tract__fit_raw_zscore_x.csv` (baseline GeoJSON Wasserstein + entropy fields).
- **Config pin (`nb07_pin`):** `RID = fit_raw_zscore_x`, `N_INTERVENTION_TRACTS = 20`, `INTERVENTION_STRENGTH = 0.5`, `RNG_SEED = 42`, `TRAVEL_TIME_MIN = 45`, `N_BOOT = 400`, `SPILLOVER_WEIGHT = 0.1`, `WELL_SERVED_EXCEEDANCE_THR = 0.05`, `DESERT_EXCEEDANCE_THR = 0.5`.
- **Design:** **Parametric scenario** (not r5py re-route — blocked by 10-month-stale NCTD feed, out of paper scope). Δ_log1p per target = `strength × (well-served median log1p − target log1p)`, with well-served = median log1p(posterior mean jobs) over tracts with exceedance < 0.05 (n ≈ 50). Queen-contiguous neighbours receive `SPILLOVER_WEIGHT × Δ`. Posterior predictive computed by shifting all 32,000 draws for each target (+ spillover on neighbours) and recomputing `P(log1p(jobs_draw) < Q25_log1p)`. 95% CI on Δ_exceedance via 400-resample bootstrap over the 32,000 draws.
- **Scenario definitions:** **A = Bayesian targeting** (top-20 by exceedance_prob, all at P=1.0); **B = Deterministic targeting** (bottom-20 by det_jobs, range 76–405). Overlap = 6/20.
- **Outputs:** 9 tables (`pipeline__07_*__fit_raw_zscore_x.csv`), 2 figures (Fig 1 study area + Fig 9 3-panel intervention effect), 3 GeoJSONs + `dashboard_manifest.json` (INTERFACES.md D002/D007 contract).
- **Key results (F031):**
  - 12 threshold crossings (A) vs 8 (B) at ≥ 80% posterior certainty — **50% efficiency advantage for Bayesian targeting**.
  - **11/12 Scenario-A crossings are spillover-induced**: targeted extreme-desert tracts cannot move below 0.5 from P=1.0; the policy signal sits in boundary-ambiguous neighbours.
  - Hook tract **06073013317** (not targeted): 0.500 → 0.027 under A, Δ = −0.472, 95% CI [−0.478, −0.467], prob_improve = 1.00 — spillover-only flip.
  - **Det-vs-Bayes classification = 178 TP / 2 FP / 0 FN / 540 TN** → the two methods *agree on WHO is a desert*; the divergence is in *priority ranking* within the 178-tract desert set.
  - Calibration: Spearman(headway, det_jobs) = −0.53; 20/20 Bayesian targets + 20/20 det targets have ≥1 MTS route within 400 m buffer.
  - Equity Spearman Δρ tiny (+0.003, +0.007) — 20/720 tracts is too small a budget to shift the county-wide gradient.
- **Caveats:** (a) `SPILLOVER_WEIGHT = 0.1` drives 11/12 crossings — sensitivity supplement recommended. (b) Parametric Δ is not an r5py re-route (design choice, not limitation). (c) `ci_lower`/`ci_upper` empty on most summary rows is **by design**: per-tract CIs live in `pipeline__07_intervention_posterior__fit_raw_zscore_x.csv` (zero NaN); summary-row metrics are point estimates (counts, Spearman, percentages). (d) GEOID dtype in CSVs defaults to int64 on CSV round-trip (loses leading zero) — load with `dtype={'GEOID': str}` + `str.zfill(11)`; the GeoJSON features carry string `geoid` correctly.