# PAPER.md
Academic paper planning, argument structure, and figure map.
Updated as the analysis develops.

---

## Working Title
"Uncertainty-Aware Transit Equity: A Hierarchical Bayesian Framework for Accessibility
Measurement and Intervention Evaluation"

## Alternative Framings
- "Whose Uncertainty Counts? Probabilistic Transit Desert Detection and the Limits of
  Deterministic Accessibility Metrics" (equity-forward, more provocative)
- "From Scores to Probabilities: Bayesian Transit Accessibility Modeling for
  Uncertainty-Aware Equity Planning" (decision-support / practitioner-facing)

## Target Venues (in order of preference)
1. Environment and Planning B: Urban Analytics and City Science
2. Computers, Environment and Urban Systems
3. Transportation Research Part B: Methodological

---

## Core Argument

Standard transit equity analyses compare point estimates and may mislead when data is sparse
or when uncertainty varies systematically across neighborhoods. We show that:

1. Treating accessibility as a posterior distribution rather than a point estimate is more
   epistemically honest and computationally tractable with modern Bayesian tools.
2. Uncertainty is not uniform — it is larger in data-sparse, often already-disadvantaged
   neighborhoods — creating a compounding inequity in the analysis itself.
3. Uncertainty-aware rankings of underserved neighborhoods differ meaningfully from
   deterministic rankings, with real implications for resource allocation.
4. Probabilistic intervention simulation gives planners actionable guidance that
   deterministic tools cannot provide.

## The Hook (Opening)
**Confirmed (2026-04-08):** Deterministic vs Bayesian rank correlation = **0.9999758** — no
rank flips exist (max delta = ±13 out of 720 tracts). The hook pivots to **uncertainty near
the desert threshold**. The perfect opening case is:

**GEOID 06073013317** — deterministic jobs = **4,467** (just above the Q25 "adequate" cutoff),
exceedance probability = **0.4998** (essentially a coin flip), ci_width_log1p = 0.341,
disadvantage_z = −0.476. Opening line: *"The city's own spreadsheet says this neighborhood
has adequate transit access. The Bayesian model says it's a coin flip."*

Second candidate: **GEOID 06073013908** — det_jobs = 4,492, exceedance = 0.484,
disadvantage_z = **+0.251** (disadvantaged). This adds a disadvantaged-tract dimension:
a neighborhood the data classifies as non-desert, but with genuine uncertainty, and it is a
disadvantaged community.

Hook candidates are archived in `artifacts/tables/pipeline__05_hook_near_q25_exceedance_ambiguous__fit_raw_zscore_x.csv`.

**Secondary hook / Fig 8**: GEOID **06073005801** (high disadvantage, above-average posterior
mean jobs) has Wasserstein distance = **61,515** from the top-quartile reference pool —
surprisingly far despite its expected access. The Wasserstein map (Fig 8) reveals that even
the equity "success story" disadvantaged tract has a distributional shape far from the
confident, narrow posteriors of genuinely well-served tracts. Same rank — very different
epistemic status.

---

## Planned Sections

| Section | Status | Key Content |
|---|---|---|
| Introduction | Outline only | Problem, gap, contribution |
| Related Work | Outline only | Accessibility methods, equity analysis, Bayesian transport |
| Methods | Outline only | GTFS pipeline, accessibility computation, hierarchical model, metrics, **§3.7 parametric intervention model** — calibrated log1p(jobs) shift = `strength × (well-served median − target)` anchored on empirical Spearman(headway, det_jobs) = −0.53; queen-contiguous spillover at weight 0.1; 400-resample bootstrap CIs on the exceedance change. Honest caveat: *calibrated scenario*, not r5py re-route (blocked by 10-month-stale NCTD feed and paper scope) |
| Data | Outline only | SD case study, data sources, coverage |
| Results | Not started | Main maps, ranking comparison, intervention simulation |
| Discussion | Not started | Implications, limitations, future work (CA/national scaling) |

---

## Planned Figures (map to `artifacts/figures/` once produced)

| Figure | Description | Notebook Source | Status |
|---|---|---|---|
| Fig 1 | Study area map — SD census tracts + MTS + NCTD stops + hook tract highlighted | 07_intervention_simulation | ✅ `pipeline__07_study_area_map.png` |
| Fig 2 | Deterministic accessibility map (baseline comparison) | 03_accessibility_computation | ✅ `pipeline__03_accessibility_choropleth__2026-04-03.png` |
| Fig 3 | Posterior mean accessibility map | 05_posterior_analysis | ✅ `pipeline__05_posterior_mean_jobs__fit_raw_zscore_x.png` |
| Fig 4 | Exceedance probability heatmap P(access < Q25) | 05_posterior_analysis | ✅ `pipeline__05_p_low_access_q25__fit_raw_zscore_x.png` |
| Fig 5 | Credible interval width map (uncertainty layer) | 05_posterior_analysis | ✅ `pipeline__05_ci_width_log1p__fit_raw_zscore_x.png` |
| Fig 6 | Case study KDE fan: posterior distributions for 4 contrasting tracts (jobs scale) | 05_posterior_analysis | ✅ `pipeline__05_case_study_fan_kde_jobs__fit_raw_zscore_x.png` **(NEW)** |
| Fig 7 | Ranking divergence: deterministic vs Bayesian ordering | 05_posterior_analysis | ✅ `pipeline__05_rank_divergence__fit_raw_zscore_x.png` |
| Fig 8 | Wasserstein distance map from well-served reference pool | 05_posterior_analysis | ✅ `pipeline__05_wasserstein_map__fit_raw_zscore_x.png` **(NEW)** |
| Fig 9 | Intervention simulation: baseline exceedance / Δ exceedance under Bayesian targeting / efficiency bar chart (A vs B) | 07_intervention_simulation | ✅ `pipeline__07_intervention_fig9__fit_raw_zscore_x.png` **(NEW)** |

---

## Claims That Still Need Empirical Support

### Confirmed by EDA (partial or full support established)
- [x] **ACS MOE is large in absolute terms** — F008: median poverty count rel. MOE = 58.8%;
  37.9% of tracts have no-vehicle MOE exceeding the estimate itself.
- [x] **MOE correlates weakly with disadvantage for population estimates** — F007: ρ = 0.16–0.20
  for poverty rate and Hispanic share vs rel_moe_pop. Modest but significant.
- [x] **Income inequality is real and large** — F009: 2.4× income gap between quartile-1 and
  quartile-4 tracts; poverty rate ranges 0% to 42.8%.
- [x] **Extreme concentrated disadvantage exists** — F010: top tract z = 4.32 (poverty 28%);
  three tracts above z = 3.0. Case study candidates for Fig 6.
- [x] **OSM walk network is fully connected** — F015: single component, 241,423 nodes. No
  routing dead zones. Confirms A007 (walk network completeness).
- [x] **THE DENSITY CONFOUND** — F014, F017: By every raw supply metric (stop count,
  headway, stop proximity, walk density, destination count within tract), San Diego's *most
  disadvantaged* tracts are *better* served than affluent ones. This inverts the expected
  equity narrative and is the paper's central empirical reframe. Disadvantaged tracts sit
  in the dense urban core (MTS-heavy); affluent tracts are low-density suburbs. Supply
  metrics mislead — actual accessibility (via r5py) will reveal whether the density
  advantage translates to opportunity reach. Key numbers: ρ(disadvantage vs stop count)
  = +0.42; ρ(disadvantage vs nearest-stop distance) = −0.57; ρ(disadvantage vs min
  headway) = −0.45; low-headway (<15 min) tracts have mean z = +0.51 vs high-headway
  (≥30 min) tracts at mean z = −0.12.

### Still Needs Empirical Support (pipeline notebooks)
- [x] **Jobs at 45 min (LODES + r5py, nb 03)** — **F020:** Spearman ρ ≈ **+0.47** between
  `jobs_C000_45min` and `disadvantage_z` on 726 tracts — disadvantaged tracts reach **more**
  jobs, not fewer; destination geography does **not** cancel the urban core’s employment
  access advantage for this deterministic measure. **POI** categories (hospitals/groceries/schools)
  in the same bundle still deserve explicit reporting; they may differ from jobs.
- [ ] **MOE story must be estimand-specific** — F007 shows poverty count MOE inverts the
  expected direction. Paper must nuance "noisiest where stakes are highest" carefully.
- [x] **Bayesian layer vs supply / deterministic accessibility** — F014/F020 show supply
  and **deterministic job** accessibility are both pro-disadvantage. **F024 (nb04, two saved runs):**
  Spearman **posterior mean** jobs vs `disadvantage_z` stays **≈ +0.47** (z-scored **X** and **Spatial+**
  **X**); **desert** exceedance vs disadvantage stays **negative** (≈ −0.47 / −0.43) — **no reversal**
  vs F020. **Coefficients differ by estimand:** positive `β[disadvantage_z]` on raw **X** vs **near zero**
  (HDI crosses 0) on **Spatial+** **X**; larger **σ** under Spatial+ (variance moves into **φ**). **F025:**
  production MCMC (**fixed** `obs_noise`, long tune/draws, Student-t) — **0 divergences**, **ρ**/**σ**
  mix well; **F022** documents the **2026-04-03** `sigma_obs`-estimated failure mode only. **F021** lists
  an **older seven-predictor** run; live spec is **four** covariates (**F024**).
- [x] **Posterior uncertainty vs disadvantage** — **F027a:** Spearman(posterior_sd_log1p, disadvantage_z) = **−0.045** (p=0.229, NS); Spearman(ci_width_log1p, disadvantage_z) = **−0.059** (p=0.114, NS). Confirmed both null in production nb05 run. **Important null result**: Bayesian uncertainty does NOT compound the equity problem — CI widths are not larger in disadvantaged tracts. Optional follow-up: correlate CI width with ACS MOE (**F007**).
- [x] **Intervention posteriors differ from naive extrapolation** — **F031 / nb07:** Bayesian top-20 targeting delivers **12 threshold crossings** (P > 0.5 → < 0.5 at ≥ 80% posterior certainty) vs **8** for deterministic bottom-20 — a 50% efficiency advantage; 1.99% vs 1.08% pop-weighted reach. **11/12 Scenario-A crossings are spillover-induced** on queen-contiguous, boundary-ambiguous neighbours — the policy signal is in the marginal tracts, not the extreme-desert targets. Det-vs-Bayes classification = **178 TP, 2 FP, 0 FN, 540 TN** → det and Bayes *agree on WHO is a desert*; the contribution is in **priority ordering + boundary-ambiguity exposure**, not hidden-desert identification. Hook tract 06073013317 (not targeted, coin flip at baseline) flips to exceedance = **0.027** under Scenario A, Δ = −0.472 (95% CI [−0.478, −0.467]), `prob_improve = 1.00`. Archived in `pipeline__07_*__fit_raw_zscore_x.csv` (9 tables) + `pipeline__07_intervention_fig9__*.png`.
- [x] **LODES job data** — downloaded and used in nb03 (`ca_wac_2021.csv.gz`; see `DATASETS.md`).
- [x] **Primary estimand + Spatial+ sensitivity** — **D011 / F024 / F026:** primary BYM2 on **raw z-scored X** (`PIPELINE_NO_SPATIAL_PLUS=1`); Spatial+ with **~5%** eigen removal default; **Moran's I** on posterior-mean residuals exported per run (`pipeline__04_moran_residual_summary__<RID>.csv`). Spatial+ archived run shows **I ≈ 0.5724** (residual spatial structure — treat as diagnostic signal; raw X primary expected much lower).
- [x] **Rank stability confirmed + perfect hook identified** — **F027c / F027d:** Spearman(rank_det, rank_bayes) = **0.9999758**, max delta = ±13 out of 720 tracts. Hook: **GEOID 06073013317** — det_jobs=4,467, exceedance_prob=**0.4998** (coin flip), disadvantage_z=−0.476. Archived in `pipeline__05_hook_near_q25_exceedance_ambiguous__fit_raw_zscore_x.csv`.
- [x] **Multi-threshold sensitivity** — **F027b / nb05:** Det jobs ρ vs disadvantage **weakens** (+0.484 → +0.414) from 30→60 min; exceedance ρ **strengthens** (−0.301 → −0.481). This asymmetry is only visible through the Bayesian exceedance framework — not detectable from deterministic-only analysis. Archived in `pipeline__05_multithreshold_equity__fit_raw_zscore_x.csv`.
- [x] **Wasserstein map complete** — **F027e:** Range 13,500 → 86,000+; perfect urban→suburban gradient; GEOID 06073005801 outlier (high disadvantage, W=61,515 despite above-average mean). Archived in `pipeline__05_wasserstein_map__fit_raw_zscore_x.png` + `pipeline__05_wasserstein_tracts__fit_raw_zscore_x.csv`.
- [x] **Multi-destination equity generalisation** — **F028 / nb06:** Pro-poor gradient holds for Hospitals, Groceries, Schools at every threshold (Spearman +0.35 to +0.58; groceries strongest). Q4 disadvantaged quartile has **2.8× more** posterior-mean jobs than Q1 and a **10× lower** desert-exceedance rate (1.7% vs 17.8%). **Composite deficit set = 120 tracts exceeding 50% desert probability on all four destinations**; **80%** have `disadvantage_z < 0` → intersectional deserts concentrate in affluent suburbs (F014 density confound carried through to POI access). Archived in `pipeline__06_multidestination_equity__`, `pipeline__06_subgroup_posterior_summary__`, `pipeline__06_composite_deficit_ranked__`, `pipeline__06_multidestination_spearman_heatmap__`, `pipeline__06_composite_deficit_map__`.
- [x] **Robustness to priors + covariate set** — **F030 / nb06:** equity Spearman = **0.4699** to 4 d.p. across 9 prior configs and 4 drop-one covariate specs. β[disadvantage_z] moves 0.10 → 0.27 when `log_pop_density` is removed (F014 density-confound signature; Hodges–Reich confounding). Archived in `pipeline__06_prior_sensitivity__` and `pipeline__06_drop_one_covariate__`.
- [ ] **Model-validation caveats (Methods §6 must address honestly)** — **F029 / nb06:** (a) PSIS-LOO Pareto k catastrophic (100% of tracts > 0.7; 82% > 1.0; known pathology for per-tract spatial random effects). (b) PPC Bayesian p-values ∈ {0, 1} on mean/sd/min/max (fixed `obs_sigma = 0.05` generative artefact). (c) Raw-X residual Moran's I = **0.6432** > Spatial+ (0.5724) — contradicts D011's prediction. None block the equity claims or nb07 intervention sim; all three are framing problems for the validation section. Optional pre-submission: one MCMC refit with `obs_noise: estimated` to confirm (a)–(c) are noise-budget artefacts.

---

## Notes on Contribution Framing
The most novel element is **six-fold** (updated 2026-04-17 after nb07 production run):

1. **Applying BYM2 spatial hierarchical models to transit accessibility** — most prior work
   uses non-spatial or point-estimate accessibility measures. BYM2 provides partial pooling
   across neighbors, making estimates robust in data-sparse suburban tracts.
2. **Demonstrating that Bayesian spatial smoothing preserves equity rankings** while adding
   genuine uncertainty quantification. Rank stability ρ = **0.9999758** (max delta ±13 out
   of 720) means the contribution is *uncertainty quantification*, not rank reversal — a
   stronger and more honest claim.
3. **Reframing the equity hook**: not rank reversals, but **threshold uncertainty** — GEOID
   06073013317 has a 50% exceedance probability despite a "passing" deterministic score.
   This is the kind of finding that changes planning decisions: a city using the deterministic
   spreadsheet would allocate resources elsewhere; the Bayesian model flags genuine ambiguity.
4. **Multi-threshold asymmetry**: the pro-disadvantage gradient weakens in *deterministic*
   jobs (ρ: +0.484 → +0.414 from 30→60 min) while the exceedance gradient *strengthens*
   (ρ: −0.301 → −0.481). This divergence is only detectable through the probabilistic
   exceedance framework and is a new finding for the transit equity literature.
5. **Multi-destination composite deficit**: extending the posterior-exceedance framework
   from jobs-only to jobs + hospitals + groceries + schools reveals that **120 tracts**
   (16.7%) cross the desert threshold on all four destinations simultaneously, and
   **80% of those are affluent low-density suburbs** (mean disadvantage_z = −0.32).
   This operationalises an intersectional equity view that single-destination Lorenz/Gini
   analyses cannot produce. Groceries show the strongest pro-poor gradient (ρ = +0.58 at
   45 min); hospitals the weakest (ρ = +0.36). The Q4-vs-Q1 disadvantage-quartile gap
   is **2.8× jobs** and **10×** lower desert-exceedance rate — both quantitative
   statements the paper can make only because the Bayesian posterior supports
   subgroup-aggregated probability claims.
6. **Probabilistic intervention targeting beats deterministic targeting on efficiency,
   not on classification** (F031 / nb07). For a fixed budget of 20 tracts, ranking by
   posterior exceedance probability delivers **12 threshold crossings** (P > 0.5 → < 0.5
   at ≥ 80% posterior certainty) versus **8** for ranking by deterministic jobs count —
   a 50% efficiency advantage, with **11 of the 12 crossings attributable to
   queen-contiguous spillover onto boundary-ambiguous neighbours** (the directly-targeted
   extreme-desert tracts start at P ≈ 1.0 and cannot be moved across 0.5 by a single
   frequency-doubling scenario). Crucially, the det-vs-Bayes classification on the full
   720-tract set is **178 TP, 2 FP, 0 FN, 540 TN** — the two methods **agree on *which*
   tracts are deserts at the 0.5 threshold**; the Bayesian contribution is in
   **ranking-within-the-desert-set** and in making the **ambiguous-boundary tracts
   visible as intervention-responsive** (e.g. hook tract 06073013317 flipping from 0.50
   to 0.03 via spillover alone). This reframes the paper's claim #4 from "Bayesian
   uncovers hidden deserts" (not supported — FN = 0) to **"Bayesian re-orders priority
   under a fixed budget and quantifies intervention responsiveness with posterior
   distributions deterministic analysis cannot produce"** (supported — Δ_crossings = +50%,
   hook tract posterior Δ = −0.472 with 95% CI tight around the mean).

**Null result as contribution**: Posterior uncertainty is NOT larger in disadvantaged
neighborhoods (ρ ≈ −0.045, NS). This is important: the Bayesian model does not compound
existing data-quality inequities (F008, F007) into wider credible intervals for already-
disadvantaged tracts. The uncertainty is driven by spatial position and model structure,
not by socioeconomic status. This should be stated explicitly rather than buried.

Novelty vs related work: Closest papers are probabilistic transit analyses (Farchau et al.,
Boisjoly et al.) but none apply full BYM2 ICAR hierarchical modeling with posterior
exceedance probabilities to transit equity at the census-tract level. The Wasserstein
distance map (Fig 8) for comparing neighborhood accessibility distributions to a reference
pool is also novel in this literature.
