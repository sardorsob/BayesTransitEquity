# HANDOVER

## What This Project Does

BayesTransitEquity models transit accessibility and equity with Bayesian spatial methods and presents outputs through a Next.js map app.

## Current Direction

The project is moving from a San Diego proof-of-concept toward a California-scale uncertainty-aware transit equity framework.

The central premise is that transit equity planning should not rely only on deterministic accessibility scores. The useful policy object is a probability statement: how confident are we that a tract falls below a policy-relevant accessibility threshold, and how likely is an intervention to move that tract across the boundary?

Near-term direction:

1. Finalize California expansion scope.
2. Run statewide data inventory and EDA before modeling.
3. Build deterministic accessibility baselines.
4. Benchmark scalable probabilistic methods against MCMC validation slices.
5. Export statewide risk/ambiguity/intervention layers for a scale-aware app.
6. Write paper claims around uncertainty, threshold governance, and intervention responsiveness.

## How To Run

Current San Diego baseline:

```bash
conda env create -f environment.yml
conda activate bayestransit
python scripts/download_data.py --config configs/san_diego.yaml
python scripts/export_frontend.py
```

Frontend:

```bash
cd app
npm install
npm run dev
```

California commands are not defined yet. Do not start statewide data pulls until `context/SCOPE.md` and `context/TASKS.md` are approved.

## Validation Commands

Current known commands:

```bash
git status --short --branch
python -m pytest
cd app
npm run build
```

Notes:

- Tests currently appear scaffolded but mostly empty.
- `npm run build` is the relevant frontend smoke gate once app work begins.
- Future California tasks should add deterministic validation scripts and run-bundle checks.

## Done

- Local workflow control files created under `context/`.
- Draft California expansion scope and task plan written locally.

## Not Done

- Sardor has not yet approved the California scope as final.
- No California data inventory has been run.
- No California config exists.
- No statewide deterministic accessibility outputs exist.
- No scalable inference benchmark exists.
- No California app layer exists.

## Important Local Workflow Files

- `context/PROJECT.md`
- `context/SCOPE.md`
- `context/TASKS.md`
- `context/AGENTS.md`
- `context/HANDOVER.md`

These files are ignored by Git through `context/`.

## Important Existing Context

- `context/PROJECT_BRIEF.md`: original project premise.
- `context/STATUS.md`: detailed San Diego pipeline status.
- `context/DATASETS.md`: current data source notes.
- `context/RISKS.md`: modeling, data, engineering, and paper risks.
- `context/INTERFACES.md`: pipeline-to-frontend data contracts.
- `context/structure.md`: artifact index and repo map.
- `context/FINDINGS.md`: empirical findings log.

## Caveats

- San Diego results do not prove statewide behavior.
- Statewide data quality will vary substantially by agency and region.
- Full-state MCMC should not be assumed feasible or necessary.
- Approximate inference must be calibrated against MCMC slices before it supports paper claims.
- The public app should expose uncertainty and data-quality caveats rather than hiding them.

## Continue From Here

- Review `context/SCOPE.md`.
- Review `context/TASKS.md`, especially TASK-001 through TASK-004.
- Decide the leading deliverable priority: paper, map, reusable pipeline, or sequenced combination.
- Once approved, move TASK-001 to `in-review`, QA it, then start TASK-002.
