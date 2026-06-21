# SCOPE

## Current Intent

Expand BayesTransitEquity from a San Diego-focused Bayesian transit equity pipeline into a California-scale workflow.

The purpose is not merely to make a bigger map. The purpose is to build a decision-support framework for uncertainty-aware transit equity planning, where accessibility estimates become policy-relevant probability statements rather than brittle binary tract labels.

## Research / Policy Premise

Transit equity programs often rely on deterministic accessibility indicators: a tract is scored, ranked, and then classified relative to a threshold. That workflow is politically convenient but mathematically thin. It hides uncertainty and makes eligibility or priority labels look more certain than they are.

BayesTransitEquity reframes the object of analysis:

```text
accessibility score < threshold
```

becomes:

```text
P(accessibility below threshold | data, spatial context, model assumptions)
```

The statewide version should answer:

1. Which California tracts are confidently underserved?
2. Which tracts are confidently adequate?
3. Which tracts are policy-boundary cases where deterministic labels are fragile?
4. Which tracts or neighborhoods are most intervention-responsive?
5. Where do data quality, feed freshness, or model assumptions weaken the claim?
6. Which approximate inference methods are credible enough for statewide screening?
7. Where is full MCMC scientifically necessary?

## Long-Run Product

A California-scale transit equity pipeline and map that can support:

- academic paper writing
- policy-facing visualization
- regional transportation planning review
- grant or service-priority sensitivity analysis
- reproducible statewide data/model audits
- future national expansion

## Goals

- Build a California data inventory across GTFS, ACS, TIGER tracts, LODES jobs, OSM street network, and OSM/other opportunity locations.
- Run a statewide EDA/data quality phase before modeling.
- Produce a deterministic statewide accessibility baseline at tract level.
- Evaluate scalable probabilistic modeling alternatives: conjugate or semi-conjugate approximations, empirical Bayes, Laplace/INLA-like approximations, variational inference, and MCMC validation.
- Use MCMC as a gold-standard calibration benchmark on selected regions, not necessarily as the first statewide production engine.
- Quantify transit-desert risk, threshold ambiguity, uncertainty width, spatial borrowing, and intervention responsiveness.
- Upgrade the map for state-scale exploration: state overview, region/county drilldown, tract detail, uncertainty layers, and data-quality caveats.
- Preserve a traceable artifact chain from raw data to final map/report.
- Produce a paper-ready argument about uncertainty-aware threshold governance in transit equity.

## Non-Goals

- Do not claim that Bayesian methods always overturn deterministic maps. San Diego already suggests the stronger claim is about ambiguity, priority ordering, and intervention responsiveness near thresholds.
- Do not run full California MCMC before data quality, deterministic accessibility, and smaller validation regions are understood.
- Do not build live inference into the Vercel app. All inference should remain precomputed.
- Do not hand-edit generated artifacts. Change code/config, rerun, regenerate.
- Do not introduce private or proprietary data unless explicitly approved.
- Do not make policy prescriptions that exceed what the model supports.

## Recommended Strategy

### Stage 1: Statewide Data Audit / EDA

Before modeling, build a California data inventory:

- all candidate GTFS schedule feeds and their validity windows
- agency-to-county and agency-to-region coverage
- overlapping feeds and cross-county service
- stop/trip/route/stop_time scale
- service-date compatibility across agencies
- TIGER/ACS tract count and demographic completeness
- LODES job availability and block-to-tract aggregation plan
- OSM PBF and routing-network feasibility
- opportunity datasets for jobs, groceries, hospitals, schools, and optional future categories

### Stage 2: Multi-Region Pilot

Pick representative validation regions:

- San Diego as known baseline
- Los Angeles / Orange County for high complexity
- Bay Area for multi-agency regional integration
- Sacramento for state-capital metro
- Central Valley region for equity and rural/urban contrast
- rural/northern California region for sparse-service behavior

The goal is to learn failure modes before statewide compute.

### Stage 3: Deterministic Statewide Accessibility

Produce tract-level deterministic accessibility first. This creates:

- baseline map
- data completeness checks
- model input table
- region-level distributions
- computational scale estimates

### Stage 4: Scalable Probabilistic Layer

Compare candidate inference families:

- empirical Bayes / shrinkage model for fast statewide tract risk
- conjugate or semi-conjugate model where assumptions are acceptable
- Laplace or INLA-like approximation for latent Gaussian spatial models
- variational inference for full probabilistic approximation
- MCMC on selected regions for calibration

The estimand matters more than the sampler. The statewide model should preserve policy-relevant uncertainty, not blindly replicate the San Diego PyMC implementation.

### Stage 5: Approximation QA

Approximate methods must be compared against MCMC validation slices:

- posterior mean error
- exceedance probability error
- uncertainty width calibration
- threshold classification disagreement
- ranking disagreement near the policy boundary
- intervention crossing disagreement

### Stage 6: Intervention Model v2

Extend San Diego nb07 into a more defensible statewide intervention module:

- separate targeted effects from spillover effects
- run sensitivity over budget, intervention strength, and spillover weights
- distinguish physical rerouting experiments from parametric policy scenarios
- report population-weighted reach and threshold crossings
- avoid claiming county/state equity-gradient change from tiny interventions

### Stage 7: Visualization and Paper

The map should make uncertainty legible:

- confident desert
- confident adequate
- ambiguous boundary case
- high uncertainty / low data confidence
- intervention-responsive
- data-quality caution

The paper should argue that uncertainty-aware accessibility is a better governance language for threshold-based equity planning.

## Constraints

- Keep workflow control files under `context/`.
- Do not push local workflow state unless explicitly requested.
- Avoid co-author trailers in commits.
- Commit locally only unless Sardor asks for a push.
- Use staged Builder/QA passes even when Codex and Sardor both act as both roles.
- Preserve public-data-only assumptions unless a new data source is explicitly approved.
- Keep app inference precomputed; Vercel should serve static/derived artifacts, not run models.
- Prefer config-driven geography so San Diego -> California -> national expansion avoids rewrites.
- Treat generated artifacts as reproducible outputs, not hand-edited deliverables.

## Open Questions

- What is the lead deliverable: paper credibility, public interactive map, reusable pipeline, or all three with one primary driver?
- Should the statewide tract geography use current TIGER year or stay aligned with the existing 2023/2024 ACS pipeline?
- Which California regions should be the MCMC validation slices?
- What accessibility opportunities matter most beyond jobs: hospitals, groceries, schools, colleges, parks, climate shelters, healthcare, government services?
- What policy threshold should be primary statewide: 30, 45, 60 minutes, or a region-relative percentile threshold?
- Should the model use one statewide spatial field, regional fields, county random effects, or a multilevel state/region/county structure?
- How should rural tracts be handled when transit is structurally sparse rather than merely underfunded?
- What compute budget is acceptable for MCMC validation runs?
- Should intervention v2 include physically rerouted GTFS/r5py scenarios, or remain parametric for the first California paper?

## Acceptance Definition

- The California expansion scope names a clear policy/research question.
- Data sources and licenses are documented before modeling.
- Statewide deterministic accessibility is reproducible from configs and scripts.
- Approximate probabilistic inference is calibrated against MCMC validation slices.
- The app exposes uncertainty and data quality honestly.
- Paper claims are framed around uncertainty, threshold ambiguity, and intervention responsiveness, not unsupported claims of discovering wholly hidden deserts.
- Every reportable result has code, config, artifact, and QA notes.
