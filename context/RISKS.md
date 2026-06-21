# RISKS.md
Known risks, failure modes, and mitigations.

---

## Data Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| GTFS feed is outdated or incomplete | Medium | High | Log feed version + date. Validate stop/trip coverage before modeling. |
| Census tract boundaries don't align with GTFS service areas | Medium | Medium | Spatial join with buffer. Document misalignments in ASSUMPTIONS.md. |
| ACS estimates have high MOE in small tracts | High | Medium | This is a feature, not a bug — the Bayesian model handles it via hierarchical shrinkage. Document. |
| OSM walk network is incomplete in some areas | Medium | Medium | Validate coverage. Use a generous walk speed assumption and document. |
| GTFS-RT data unavailable or inconsistent | Medium | Low | Deferred to Phase 2. Initial model uses schedule-only. |

## Modeling Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| MCMC chains fail to converge | Medium | High | Monitor R-hat and ESS. Use more tuning steps. Document divergences. |
| Spatial model (CAR/BYM) computationally intractable for SD | Low | High | SD has ~400 tracts — should be fine. Switch to VI if needed. |
| Prior choices drive results more than data | Medium | High | Sensitivity analysis on priors. Document in ASSUMPTIONS.md. |
| Accessibility threshold (45 min) is arbitrary | High | Medium | Run sensitivity analysis at 30/45/60 min thresholds. Report all. |
| Model overfits to spatial autocorrelation structure | Low | Medium | Posterior predictive checks. Cross-validation if needed. |
| **PSIS-LOO Pareto k catastrophic** for per-tract spatial random effects (F029) | **Realised** | Medium | Drop numeric LOO claim; keep Pareto-k map as influential-tract diagnostic; optional spatial K-fold CV for model comparison. Documented in paper Methods §6. |
| **Fixed `obs_sigma` = 0.05 produces degenerate PPC** (Bayesian p ∈ {0, 1} on mean/sd/min/max; F029) | **Realised** | Medium | Report PPC on posterior-mean replicates rather than y_rep; optional sensitivity refit with `obs_noise: estimated` before submission. |
| **Residual Moran's I ≈ 0.64 on raw X, higher than Spatial+ (F029)** — contradicts D011 prediction | **Realised** | Medium | Own it: report both numbers (raw X + Spatial+) in Supplement. Discuss as BYM2-plus-fixed-noise artefact. Does not affect equity Spearman (F028/F030 invariant). |

## Engineering Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| GeoJSON files too large for Vercel static serving | Medium | Medium | Use PMTiles or split by bounding box. Document in INTERFACES.md. |
| r5py setup is complex (Java dependency) | **Mitigated** | Medium | Java setup now embedded in nb03 cell 4 (cannot be skipped). Searches CONDA_PREFIX for jvm.dll/libjvm.so and sets JAVA_HOME automatically. |
| GTFS multi-feed merge silently drops required FK fields | **Mitigated** | High | `prefix_feed()` now remaps all FK columns including `parent_station` and synthesises missing `agency_id`. Full 14-point referential integrity check embedded before R5 load. |
| Posterior Parquet files too large for git | High | Low | Gitignore `data/processed/posteriors/`. Store separately or in artifact store. |
| Scaling to California breaks config-driven design | Low | High | Enforce no hardcoded geography in code reviews. Test with a second city config early. |

## Paper Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Ranking reversal result is not compelling | **Realised** | Medium | F027c confirmed Spearman 0.9999758 (no reversals). Reframed as **uncertainty near the desert threshold** — hook tract 06073013317 (F027d); paper now leads with coin-flip exceedance, not rank flips. |
| Uncertainty-correlates-with-disadvantage claim doesn't hold | **Realised** | Medium | F027a: ρ = −0.045 (NS). Reframed as a **positive null finding** — Bayesian model does not compound existing data-quality inequities into wider CIs; uncertainty is driven by spatial position + model structure, not SES. |
| Venue rejection due to methods being perceived as incremental | Medium | Medium | nb07 / F031 delivers the intervention novelty. Reframed headline: **"Bayesian re-orders priority + exposes boundary-ambiguous intervention-responsive tracts"** (not "Bayesian reveals hidden deserts" — FN=0 rules that out). Claim is supported by 12-vs-8 crossings + hook tract 0.50→0.03 with CI. |
| **F031 classification reframe**: 0 false negatives means det and Bayes agree on WHICH tracts are deserts | **Realised** | Medium | Reframe claim #4 to priority-ordering + boundary-ambiguity exposure (see PAPER.md §Notes on Contribution Framing point 6). Paper Results §5.4 should lead with the 50%-more-crossings + 11/12-spillover + hook-tract story, NOT a "hidden deserts" claim. |
| **nb07 parametric Δ is a calibrated scenario, not an r5py re-route** — reviewer may ask for physical validation | Medium | Medium | Honest framing in Methods §3.7: calibration anchored on empirical Spearman(headway, det_jobs) = −0.53; r5py re-route blocked by 10-month-stale NCTD feed and is out of paper scope (we are not claiming a routing-engineering contribution). Optional one-tract r5py sanity check is pre-specified in F031 next-question (d) for reviewer response. |
| **Spillover weight (0.1) drives 11/12 of Scenario A crossings** — sensitive parameter | Medium | Medium | Pre-submission supplemental: re-run nb07 with `SPILLOVER_WEIGHT ∈ {0, 0.05, 0.10, 0.15}` → crossings-vs-weight table in Supplement. F031 next-question (a) documents the need. The qualitative A > B ordering is robust (A > B at any non-zero spillover; degenerate at spillover=0 because all targets start at P=1.0). |
| **Intervention budget (20 tracts) is small relative to 720-tract county** → Δρ barely moves | **Realised** | Low | Paper must not claim equity-gradient reduction (Δρ ≈ +0.003 for A). Cite crossings (12 vs 8) and pop-weighted reach (1.99% vs 1.08%) instead. Optional N ∈ {10, 40} budget sweep documented as F031 next-question (b). |
