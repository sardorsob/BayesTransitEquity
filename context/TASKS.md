# TASKS

## Build Progress

| Status | Count |
| --- | ---: |
| pending | 12 |
| in-progress | 1 |
| in-review | 0 |
| needs-fix | 0 |
| blocked | 0 |
| done | 0 |
| obsolete | 0 |

## Status Machine

Allowed statuses:

- pending
- in-progress
- in-review
- needs-fix
- blocked
- done
- obsolete

Legal transitions:

- pending -> in-progress
- in-progress -> in-review
- in-progress -> blocked
- in-review -> done
- in-review -> needs-fix
- in-review -> blocked
- needs-fix -> in-progress
- blocked -> pending
- blocked -> in-progress
- pending -> obsolete

## TASK-001

- Phase: scope
- Title: Finalize California expansion scope and research framing
- Depends on: none
- Assigned agent: Builder + QA: Sardor and Codex
- Contract refs: `context/SCOPE.md`, `context/PROJECT.md`, `context/PROJECT_BRIEF.md`, `context/PAPER.md`
- Data refs: none
- Scientific refs: San Diego findings F020, F027, F028, F029, F030, F031
- User value / decision value: Establish the statewide expansion as a policy-relevant uncertainty framework before investing in data pulls or long compute.
- Functional notes:
  - Clarify whether the leading deliverable is paper credibility, public map, reusable pipeline, or a sequenced combination.
  - Frame the project around threshold uncertainty, ambiguity, and intervention responsiveness.
  - State explicitly that the project is not "Bayesian finds totally different deserts" unless evidence supports it.
- Statistical notes:
  - Define the target estimands before choosing inference method.
  - Candidate estimands: posterior mean accessibility, exceedance probability, uncertainty width, boundary ambiguity, intervention crossing probability.
- Edge cases:
  - If the policy threshold is unclear, keep multiple thresholds and avoid a single headline.
  - If a method cannot preserve uncertainty calibration, it should be screening-only.
- Files to create/modify:
  - `context/SCOPE.md`
  - `context/PROJECT.md`
  - `context/TASKS.md`
- Artifacts to produce:
  - Draft California scope
  - Draft task list
- Acceptance criteria:
  - [ ] Sardor approves the research/policy premise.
  - [ ] Scope has goals, non-goals, constraints, open questions, and acceptance definition.
  - [ ] Task list is staged and executable.
- Verification commands:
  - Manual review only; context files are ignored local workflow state.
- Manual QA:
  - Sardor reads scope and flags missing/incorrect strategic assumptions.
- QA notes:
  - pending
- Attempts: 1
- Max attempts: 3
- Attempt log:
  - 2026-06-20: Drafted initial California expansion scope and task breakdown.
- Status: in-progress

## TASK-002

- Phase: data-audit
- Title: Build California dataset inventory and data card
- Depends on: TASK-001
- Assigned agent: Builder
- Contract refs: `context/SCOPE.md`, `context/DATASETS.md`
- Data refs: GTFS catalog, ACS, TIGER, LODES, OSM, opportunity datasets
- Scientific refs: Sardor data science workflow Gate 1 Data Gate
- User value / decision value: Know what statewide data exists, what is stale or missing, and what can be legally/reproducibly used before modeling.
- Functional notes:
  - Inventory statewide transit, demographics, geography, jobs, streets, and opportunities.
  - Document source, license, update cadence, schema, geographic level, and known caveats.
  - Preserve San Diego source notes while adding California sections.
- Statistical notes:
  - Data quality flags must become model/app covariates or caveats where relevant.
  - Feed validity windows are not clerical; they affect accessibility estimates.
- Edge cases:
  - Some agencies may publish stale, missing, or malformed GTFS feeds.
  - Some regions may have cross-county service that breaks county-only analysis.
- Files to create/modify:
  - `context/DATASETS.md`
  - `context/SCOPE.md`
  - `configs/california.yaml` (later implementation)
- Artifacts to produce:
  - California data inventory table
  - Feed freshness summary
- Acceptance criteria:
  - [ ] Each data source has owner/source, license, path, schema summary, and caveat.
  - [ ] Transit feeds are mapped to likely county/region coverage.
  - [ ] Unknowns are explicitly marked, not guessed.
- Verification commands:
  - Define once California data inventory scripts exist.
- Manual QA:
  - Review whether the data inventory is complete enough to start EDA.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-003

- Phase: data-audit
- Title: Design California config and region partition strategy
- Depends on: TASK-001, TASK-002
- Assigned agent: Builder
- Contract refs: `configs/san_diego.yaml`, `configs/defaults.yaml`, `context/INTERFACES.md`
- Data refs: California TIGER tracts, GTFS coverage, OSM PBF
- Scientific refs: none
- User value / decision value: Make geography config-driven so California and future national expansion do not require rewrites.
- Functional notes:
  - Decide whether California is one config with region partitions or multiple region configs under one state umbrella.
  - Include tract filters, agency filters, departure dates, OSM PBF paths, and chunking parameters.
  - Define canonical region IDs for pilot slices.
- Statistical notes:
  - Region partitioning affects spatial adjacency, random effects, and cross-region comparability.
- Edge cases:
  - Large rural tracts may distort adjacency and accessibility distributions.
  - Cross-county transit corridors should not be cut incorrectly.
- Files to create/modify:
  - `configs/california.yaml`
  - `configs/regions/*.yaml` or equivalent
  - `context/DECISIONS.md`
- Artifacts to produce:
  - Config design decision
- Acceptance criteria:
  - [ ] Config can represent San Diego, pilot regions, and statewide California.
  - [ ] No hardcoded geography is needed in scripts/notebooks.
- Verification commands:
  - Define once California config validation exists.
- Manual QA:
  - Review region partition list before data pulls.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-004

- Phase: eda
- Title: Run California GTFS and spatial coverage EDA
- Depends on: TASK-002, TASK-003
- Assigned agent: Builder
- Contract refs: `context/DATASETS.md`, `context/RISKS.md`, `context/structure.md`
- Data refs: GTFS feeds, TIGER tracts, OSM boundaries
- Scientific refs: San Diego EDA notebooks 01-07
- User value / decision value: Identify feed quality, service coverage, and spatial alignment issues before expensive accessibility computation.
- Functional notes:
  - Reuse San Diego EDA structure, but scale outputs by agency, county, and region.
  - Summarize stop coverage, route coverage, service calendars, headways, feed validity, and cross-county overlap.
- Statistical notes:
  - EDA should test whether San Diego density-confound patterns generalize statewide.
- Edge cases:
  - Very large stop_times files may need chunked parsing.
  - Multiple agencies can share stops or duplicate service.
- Files to create/modify:
  - EDA notebooks or scripts
  - `context/FINDINGS.md`
  - `context/structure.md`
- Artifacts to produce:
  - GTFS feed summary table
  - Coverage maps
  - Feed freshness warnings
  - Service density/confound diagnostics
- Acceptance criteria:
  - [ ] California EDA identifies usable feeds and high-risk feeds.
  - [ ] Regional/pilot selection is evidence-backed.
  - [ ] Findings are logged with caveats.
- Verification commands:
  - Define once California EDA scripts/notebooks exist.
- Manual QA:
  - Review whether statewide modeling should proceed or data gaps block it.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-005

- Phase: deterministic-accessibility
- Title: Prototype deterministic accessibility on representative California regions
- Depends on: TASK-003, TASK-004
- Assigned agent: Builder
- Contract refs: `context/INTERFACES.md`, `context/RISKS.md`
- Data refs: pilot GTFS bundles, OSM PBF clips, TIGER/ACS/LODES
- Scientific refs: San Diego notebook 03
- User value / decision value: Estimate routing scale and validate region-level computation before statewide runs.
- Functional notes:
  - Start with San Diego, LA/OC, Bay Area, Sacramento, Central Valley, and one rural/northern region.
  - Record runtime, memory, OD size, failures, and output schema.
- Statistical notes:
  - Deterministic accessibility becomes the response/input for probabilistic modeling.
  - Keep threshold sensitivity at 30/45/60 minutes.
- Edge cases:
  - r5py may fail on malformed merged GTFS.
  - Some regions may require multiple departure dates due to feed validity.
- Files to create/modify:
  - routing scripts
  - config files
  - run bundle scripts
- Artifacts to produce:
  - Pilot accessibility bundles
  - Runtime/memory logs
  - Failure notes
- Acceptance criteria:
  - [ ] At least three representative regions produce valid accessibility bundles.
  - [ ] Failures are classified as data, routing, memory, or config problems.
- Verification commands:
  - Define once pilot routing scripts exist.
- Manual QA:
  - Inspect maps/tables for obviously impossible accessibility patterns.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-006

- Phase: deterministic-accessibility
- Title: Produce statewide deterministic accessibility baseline
- Depends on: TASK-005
- Assigned agent: Builder
- Contract refs: `context/INTERFACES.md`
- Data refs: statewide GTFS, OSM, ACS, LODES
- Scientific refs: San Diego notebook 03
- User value / decision value: Create the statewide baseline map and model input table.
- Functional notes:
  - Run California by partition if needed, then stitch tract-level outputs.
  - Export deterministic jobs and opportunity access for all supported tracts.
- Statistical notes:
  - Preserve thresholds and region identifiers for downstream hierarchical modeling.
- Edge cases:
  - Missing service regions should be represented honestly, not silently dropped.
  - Boundary tracts may need careful routing partition treatment.
- Files to create/modify:
  - pipeline scripts
  - export scripts
  - `context/structure.md`
- Artifacts to produce:
  - statewide tract accessibility bundle
  - deterministic GeoJSON or tile-ready layer
  - run bundle
- Acceptance criteria:
  - [ ] Every included tract has a clear status: computed, no service, missing data, or excluded with reason.
  - [ ] Outputs satisfy documented schema.
- Verification commands:
  - Define once statewide routing pipeline exists.
- Manual QA:
  - Statewide map sanity check by region.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-007

- Phase: modeling
- Title: Define statewide probabilistic estimands and candidate model family
- Depends on: TASK-001, TASK-006
- Assigned agent: Builder + QA: Sardor and Codex
- Contract refs: `context/SCOPE.md`, `context/RISKS.md`
- Data refs: statewide accessibility baseline
- Scientific refs: San Diego BYM2 model, Bayesian workflow checklist
- User value / decision value: Choose model structure based on policy estimands and scale rather than sampler preference.
- Functional notes:
  - Define estimands: posterior mean, exceedance probability, ambiguity zone, uncertainty width, intervention crossing probability.
  - Compare candidate model classes before implementation.
- Statistical notes:
  - Consider empirical Bayes, conjugate/semi-conjugate approximations, Laplace/INLA-like latent Gaussian methods, VI, and MCMC validation.
  - Treat VI variance underestimation as a known QA target.
- Edge cases:
  - One statewide spatial field may be too rigid; region/county random effects may be needed.
  - Rural tracts may require separate modeling assumptions.
- Files to create/modify:
  - `context/MODELING_NOTES.md` or existing equivalent
  - `context/DECISIONS.md`
  - model design docs
- Artifacts to produce:
  - modeling design memo
- Acceptance criteria:
  - [ ] Estimands are explicit.
  - [ ] Candidate model families are compared with tradeoffs.
  - [ ] Validation plan against MCMC is defined.
- Verification commands:
  - Manual review only.
- Manual QA:
  - Sardor approves model direction.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-008

- Phase: modeling
- Title: Benchmark approximate inference against MCMC validation slices
- Depends on: TASK-005, TASK-007
- Assigned agent: Builder
- Contract refs: `context/RISKS.md`, `context/FINDINGS.md`
- Data refs: pilot accessibility bundles and posterior outputs
- Scientific refs: San Diego MCMC diagnostics, Bayesian workflow checklist
- User value / decision value: Determine which fast inference method is credible for statewide screening.
- Functional notes:
  - Fit candidate approximate methods on selected regions.
  - Fit or reuse MCMC benchmarks where feasible.
  - Compare results around policy thresholds.
- Statistical notes:
  - Primary comparison should include exceedance probability calibration, not only posterior mean error.
  - VI underdispersion should be measured directly.
- Edge cases:
  - MCMC may fail or be too slow in LA/Bay Area; use smaller slices if necessary.
  - Approximate methods may rank well but miscalibrate threshold probabilities.
- Files to create/modify:
  - model scripts
  - evaluation scripts
  - experiment registry
- Artifacts to produce:
  - approximation-vs-MCMC comparison tables
  - calibration plots
  - runtime/memory benchmark
- Acceptance criteria:
  - [ ] At least two approximate methods are compared to MCMC or a credible benchmark.
  - [ ] Chosen statewide method has documented limitations.
- Verification commands:
  - Define once benchmark scripts exist.
- Manual QA:
  - Scientific review of whether approximation is honest enough for paper/map claims.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-009

- Phase: modeling
- Title: Build statewide probabilistic transit-risk layer
- Depends on: TASK-006, TASK-008
- Assigned agent: Builder
- Contract refs: `context/INTERFACES.md`
- Data refs: statewide accessibility bundle
- Scientific refs: selected modeling design from TASK-008
- User value / decision value: Produce statewide uncertainty-aware tract outputs for map and paper.
- Functional notes:
  - Export posterior/risk summaries with tract, county, region, and data-quality fields.
  - Keep full posterior samples only where needed; use summaries for app payload.
- Statistical notes:
  - Include uncertainty calibration caveats from TASK-008.
  - Distinguish screening estimates from gold-standard MCMC estimates.
- Edge cases:
  - Some tracts may be excluded or marked low-confidence.
  - Statewide adjacency may require sparse matrix handling and memory safeguards.
- Files to create/modify:
  - model pipeline
  - export pipeline
  - data contracts
- Artifacts to produce:
  - statewide risk summary table
  - statewide frontend layer
  - run bundle
- Acceptance criteria:
  - [ ] Outputs include posterior mean/risk/ambiguity/uncertainty fields.
  - [ ] Data-quality flags travel into exported layers.
- Verification commands:
  - Define once statewide probabilistic export exists.
- Manual QA:
  - Inspect statewide risk maps for implausible regional artifacts.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-010

- Phase: interventions
- Title: Design California intervention simulation v2
- Depends on: TASK-007, TASK-009
- Assigned agent: Builder
- Contract refs: San Diego nb07 outputs, `context/RISKS.md`
- Data refs: statewide risk layer, GTFS service features
- Scientific refs: San Diego F031
- User value / decision value: Estimate which investments are likely to move tracts across policy-relevant thresholds.
- Functional notes:
  - Separate targeted improvements from spillover effects.
  - Run sensitivity over budget, intervention strength, and spillover weight.
  - Consider route/frequency scenario inputs but keep parametric mode as first version.
- Statistical notes:
  - Report threshold crossings and population-weighted reach, not only changes in statewide correlation.
  - Avoid claiming physical rerouting effects unless r5py reroute scenario is actually run.
- Edge cases:
  - Spillover may dominate results as in San Diego.
  - Extreme-desert targets may not cross thresholds even after intervention.
- Files to create/modify:
  - intervention scripts/notebooks
  - frontend scenario export
  - `context/FINDINGS.md`
- Artifacts to produce:
  - intervention sensitivity table
  - scenario layers
  - hook/boundary tract examples
- Acceptance criteria:
  - [ ] Sensitivity over spillover and budget is included.
  - [ ] Claims distinguish parametric simulation from routing-engineering scenario.
- Verification commands:
  - Define once intervention v2 scripts exist.
- Manual QA:
  - Review whether intervention claims are politically and scientifically defensible.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-011

- Phase: frontend
- Title: Redesign map/app architecture for California-scale layers
- Depends on: TASK-006, TASK-009
- Assigned agent: Builder
- Contract refs: `context/INTERFACES.md`, app data contracts
- Data refs: statewide deterministic and probabilistic layers
- Scientific refs: visualization app lane checklist
- User value / decision value: Let users inspect statewide patterns without hiding tract-level uncertainty and caveats.
- Functional notes:
  - State overview -> region/county -> tract drilldown.
  - Layers: deterministic access, exceedance probability, ambiguity, uncertainty, intervention responsiveness, data quality.
  - Use vector tiles/PMTiles/split bundles as needed.
- Statistical notes:
  - Visual labels must distinguish confidence from point estimates.
  - Avoid color scales that imply certainty where model/data confidence is weak.
- Edge cases:
  - GeoJSON may be too large for direct static serving.
  - Mobile interaction needs a simpler reading path.
- Files to create/modify:
  - app routes/components
  - frontend data layer
  - export scripts
- Artifacts to produce:
  - frontend data bundles
  - screenshot QA
- Acceptance criteria:
  - [ ] App can load state-scale data without freezing.
  - [ ] Uncertainty and data-quality caveats are visible in the main user flow.
- Verification commands:
  - `npm run build`
  - browser/screenshot smoke tests
- Manual QA:
  - Inspect desktop and mobile views.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-012

- Phase: paper
- Title: Draft paper argument for uncertainty-aware threshold governance
- Depends on: TASK-001, TASK-008, TASK-010
- Assigned agent: Builder
- Contract refs: `paper/`, `context/PAPER.md`, `context/FINDINGS.md`
- Data refs: final analysis artifacts
- Scientific refs: transit accessibility, Bayesian workflow, equity planning literature
- User value / decision value: Convert the technical pipeline into a useful academic and policy contribution.
- Functional notes:
  - Frame around uncertainty, threshold politics, ambiguity, and intervention responsiveness.
  - Use San Diego as proof-of-concept and California as scaling/validation if available.
- Statistical notes:
  - Do not overclaim rank reversals or hidden deserts.
  - Include approximation limits and MCMC validation results.
- Edge cases:
  - If statewide modeling is incomplete, write as methods/platform + pilot paper.
- Files to create/modify:
  - `paper/main.md`
  - `paper/sections/*.md`
  - `context/PAPER.md`
- Artifacts to produce:
  - paper draft sections
  - figure/table references
- Acceptance criteria:
  - [ ] Claims trace to artifacts.
  - [ ] Limitations are explicit.
  - [ ] Policy relevance is clear without model determinism.
- Verification commands:
  - Define once paper build/lint process exists.
- Manual QA:
  - Read for scientific honesty and political clarity.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## TASK-013

- Phase: qa-handover
- Title: Maintain QA gates, run bundles, and handover state
- Depends on: all active tasks
- Assigned agent: QA
- Contract refs: `context/HANDOVER.md`, `context/PROJECT.md`, `context/structure.md`
- Data refs: all produced artifacts
- Scientific refs: Sardor workflow kit gates
- User value / decision value: Keep the project continuable without relying on chat memory.
- Functional notes:
  - Update context after each done task.
  - Keep run bundles for reportable outputs.
  - Record blockers, verification commands, and caveats.
- Statistical notes:
  - QA must check whether claims match the strength of evidence.
- Edge cases:
  - Long compute jobs may finish outside a chat session; logs must be durable.
- Files to create/modify:
  - `context/HANDOVER.md`
  - `context/PROJECT.md`
  - `context/structure.md`
  - artifact logs
- Artifacts to produce:
  - updated handover
  - run bundle index
- Acceptance criteria:
  - [ ] Every completed task has evidence.
  - [ ] Handover explains how to continue.
  - [ ] No secrets or private data are committed.
- Verification commands:
  - Define as each task adds concrete validation commands.
- Manual QA:
  - Sardor confirms project state is understandable.
- QA notes:
  - pending
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending

## Task Template

```text
## TASK-000

- Phase:
- Title:
- Depends on:
- Assigned agent:
- Contract refs:
- Data refs:
- Scientific refs:
- User value / decision value:
- Functional notes:
- Statistical notes:
- Edge cases:
- Files to create/modify:
- Artifacts to produce:
- Acceptance criteria:
- Verification commands:
- Manual QA:
- QA notes:
- Attempts: 0
- Max attempts: 3
- Attempt log:
  - none
- Status: pending
```
