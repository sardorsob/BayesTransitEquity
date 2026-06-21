# development_rules.md
Universal Project Architecture + Operating Rules for AI-Assisted Development (Cursor + LLMs)

This file is the **single operational contract** for how this repository is structured, how work is performed, and how results are logged so that:
- Humans can understand and reproduce everything later.
- AI agents (Cursor / LLMs) can safely extend the repo without drifting the structure.
- Analysis, production code, and presentation artifacts stay cleanly separated.
- Every claim in reports can be traced to code + data + artifacts.

If this file conflicts with any other doc, **this file wins** unless `DECISIONS.md` explicitly supersedes it.

---

## 0) Non-Negotiable Principles

1) **Reproducibility beats convenience**
   - Anything “important” must be reproducible from code, not manual steps.
   - Outputs must be tied to inputs + config + seed.

2) **Artifact-first, notebook-second**
   - Notebook outputs are not the deliverable.
   - Deliverables are exported artifacts (figures/tables/models/reports) + a written log.

3) **One source of truth for structure**
   - `structure.md` describes the repo layout and indexes produced results.

4) **Separation of concerns**
   - `notebooks/` = exploration + analysis runs
   - `src/` = reusable code
   - `artifacts/` = generated outputs
   - `site/` or `reports/` = human-facing narrative (paper-style or website)

5) **Agent-safe iteration**
   - Every meaningful change must update logs (decision / experiment / structure).
   - Avoid “silent changes” that break continuity across sessions.

---

## 1) Required Repository Layout (Domain-Neutral)

Minimum required tree (add folders only when justified):

├─ README.md
├─ development_rules.md # this file
├─ structure.md # repo map + artifact index + results log
├─ DECISIONS.md # why we chose X over Y
├─ ASSUMPTIONS.md # modeling/data assumptions + caveats
├─ CHANGELOG.md # milestones (optional but recommended)
│
├─ configs/ # yaml/json configs for experiments + pipelines
├─ context/ # project memory for humans + agents (see section 7)
│
├─ data/ # governed by tier rules below
│ ├─ raw/ # immutable inputs (often gitignored)
│ ├─ interim/ # intermediate cached transforms (gitignored)
│ ├─ processed/ # final derived datasets (selectively tracked)
│ └─ external/ # reference files, lookups, schemas
│
├─ notebooks/ # numbered, reproducible analysis notebooks
├─ src/ # reusable python code (package-style)
│ ├─ init.py
│ ├─ io/ # loaders, parsing, dataset objects
│ ├─ preprocessing/ # cleaning, transforms, feature creation
│ ├─ modeling/ # model defs, training, inference
│ ├─ evaluation/ # metrics, validation, comparisons
│ ├─ viz/ # plotting + export helpers
│ └─ utils/ # paths, seeds, logging, helpers
│
├─ scripts/ # CLI entrypoints (run.py, train.py, eval.py, build_site.py)
├─ tests/ # minimal tests for core logic
│
├─ artifacts/ # ALL generated outputs (never hand-edited)
│ ├─ figures/
│ ├─ tables/
│ ├─ models/
│ ├─ reports/
│ └─ logs/
│ ├─ runs/ # per-run metadata bundles
│ ├─ prompts/ # prompt snapshots & LLM outputs (selective)
│ └─ provenance/ # manifests/hashes for traceability
│
└─ site/ (or docs/) # static site source (MkDocs/Docusaurus/etc.) (optional)


If you add a top-level folder, you must:
- Add it to `structure.md` (purpose + what belongs there).
- Update this file if it changes workflow rules.

---

## 2) Data Governance (Works for Any Domain)

### 2.1 Data tiers (what goes where)
- `data/raw/`: immutable source data (downloaded archives, original exports, vendor dumps).
  - Usually **gitignored** due to size/licensing.
  - Never modify files in-place.
- `data/interim/`: cached transforms and temporary outputs.
  - Always gitignored.
- `data/processed/`: meaningful derived datasets you expect to reuse.
  - Track only if small enough and legally allowed; otherwise export manifests + pointers.
- `data/external/`: reference data, schemas, metadata tables, mappings.

### 2.2 “No mystery preprocessing”
Every preprocessing step must be:
- in code (`src/preprocessing/` or `scripts/`)
- configured (`configs/`)
- logged (run bundle + summarized in `structure.md`)

---

## 3) Experiment Tracking (File-Based First)

We use a **run folder per experiment** even if we later use MLflow/W&B.

### 3.1 Run ID convention
`YYYY-MM-DD__HHMM__<short_tag>__<git_shortsha>`

Example:
`2026-02-25__1130__baseline_model__a1b2c3d`

### 3.2 Every run writes a “run bundle”
Path:
`artifacts/logs/runs/<run_id>/`

Minimum files:
- `meta.json`: run_id, timestamp, user, git hash, machine info
- `config.yaml`: exact config snapshot used
- `seeds.json`: RNG seeds (if applicable)
- `inputs.json`: input dataset paths + hashes (or file sizes + modified times if hashing is too heavy)
- `outputs.json`: produced artifacts with relative paths
- `metrics.json`: key metrics (task-dependent)
- `notes.md`: short narrative summary (what/why/results/next steps)

If a run produces a “reportable” figure/table/model, it must be referenced in:
- the run’s `outputs.json`
- `structure.md` artifact index

---

## 4) Artifact Rules (STRICT)

### 4.1 Never hand-edit generated artifacts
Artifacts must be produced by code. If you need to change an artifact:
- change code
- rerun
- regenerate

### 4.2 Where artifacts go
- Figures → `artifacts/figures/`
- Tables → `artifacts/tables/`
- Models → `artifacts/models/`
- Auto reports (HTML/PDF/MD) → `artifacts/reports/`
- Run metadata → `artifacts/logs/runs/<run_id>/`

### 4.3 Naming convention (all artifacts)
`<stage>__<short_description>__<run_id>.<ext>`

Examples:
- `eda__feature_distributions__2026-02-25__1130__baseline_model__a1b2c3d.png`
- `eval__metrics_summary__2026-02-25__1130__baseline_model__a1b2c3d.csv`
- `model__final_checkpoint__2026-02-25__1130__baseline_model__a1b2c3d.pkl`

### 4.4 “Reportable” means exported
If it is referenced in a report, it must exist as an exported artifact file.

---

## 5) Notebook Rules (STRICT)

### 5.1 Notebook purpose
Notebooks are for:
- exploration
- quick iteration
- readable analysis narratives

Notebooks are NOT:
- the only place logic lives
- the only place results exist

### 5.2 Notebook structure
- Must run top-to-bottom without manual state.
- Heavy logic must move into `src/`.
- Always use relative paths via a central path helper.

### 5.3 Notebook naming
`notebooks/01_<topic>.ipynb`, `02_<topic>.ipynb`, …

### 5.4 Notebook “done” checklist (mandatory)
A notebook change is not “done” unless:
- outputs are exported to `artifacts/`
- a run bundle exists if the notebook produced results
- `structure.md` updated
- relevant assumptions/decisions updated

---

## 6) Code Standards (src/ + scripts/)

### 6.1 Style
- Prefer small functions with docstrings.
- Minimal comments; docstrings explain purpose + inputs/outputs.
- Avoid noisy prints; use lightweight logging.

### 6.2 Determinism
If randomness affects results:
- set seeds (numpy/random/torch/etc.)
- log them into `seeds.json`

### 6.3 Configuration
No hidden magic numbers:
- key hyperparameters/options go into `configs/*.yaml`
- scripts accept `--config configs/foo.yaml`

### 6.4 Paths
Never hardcode absolute paths. Use:
- repo-root resolution helper (e.g., `src/utils/paths.py`)
- config-driven data locations

---

## 7) Context System (Agent + Human Memory)

The `context/` folder is the “long-running memory” for the project.

Recommended files:
- `context/PROJECT_BRIEF.md`
  - What we’re building, why, success criteria, constraints.
- `context/GLOSSARY.md`
  - Domain terms, acronyms, dataset field meanings.
- `context/DATASETS.md`
  - Where data comes from, schemas, licensing/permissions, mappings.
- `context/INTERFACES.md`
  - APIs, I/O contracts, expected shapes, file formats.
- `context/STATUS.md`
  - Current state, what’s done, what’s next, blockers.
- `context/RISKS.md`
  - Known risks, failure modes, mitigations.

Rules:
- Agents may update these files, but must keep them concise and factual.
- Any major change in direction must be logged in `DECISIONS.md`.

---

## 8) LLM / Agent Compatibility Rules

### 8.1 Prompt & output logging (selective but structured)
When an LLM output influences the project (design, evaluation plan, codegen):
store a snapshot under:
`artifacts/logs/prompts/<date>__<topic>/`

Minimum files:
- `prompt.md`
- `response.md`
- `extracted_actions.md` (what you actually implemented)
- `status.md` (accepted/rejected/partial)

Never store secrets.

### 8.2 Decision discipline (prevents drift)
Any meaningful choice must be captured in `DECISIONS.md`:
- decision statement
- alternatives considered
- rationale
- consequences / follow-ups
- date + (optional) run_id link

### 8.3 Assumptions discipline
All assumptions go in `ASSUMPTIONS.md`:
- what you assume
- why it’s reasonable
- how it could fail
- how you’ll test sensitivity

---

## 9) Documentation System

### 9.1 structure.md (single source of truth)
`structure.md` must include:
1) Project map (what each folder/file is for)
2) Artifact index (figures/tables/models/reports with one-line meaning)
3) Results log by notebook/script (inputs → outputs → findings)
4) Current status + next steps

### 9.2 README.md (entry point)
Keep it short but complete:
- what project is
- how to install
- how to run key scripts
- where to find results

---

## 10) Website / Report Generation (Optional but Recommended)

If you maintain a site (`site/` or `docs/`):
- Site must be generated from or reference `artifacts/`.
- Do not manually copy plots; link or build.
- Prefer a build script:
  - `scripts/build_site.py`
  - reads `structure.md` (and artifact index)
  - outputs static pages

---

## 11) Reproducibility Protocol

Every “reportable result” must have:
- run_id
- config snapshot
- seeds (if relevant)
- input provenance
- output list
- minimal notes

A collaborator should be able to:
1) install environment
2) run a script/notebook
3) regenerate the same artifacts (within tolerance)

---

## 12) Minimal Testing (Keep It Lightweight)

At minimum:
- unit test for critical loader/parsing functions
- unit test for one core transformation
- unit test for one evaluation/metric function
- “smoke test” script that runs a tiny end-to-end subset

---

## 13) Security / Hygiene

- Never commit API keys.
- Never commit private datasets unless explicitly allowed.
- Maintain `.gitignore` for large files and caches:
  - `data/raw/**` (usually)
  - `data/interim/**`
  - `__pycache__/`, `.ipynb_checkpoints/`
  - optional: large artifacts if needed (but keep run logs if possible)

---

## 14) Lightweight vs Full Mode

### Lightweight mode (solo, fast)
- run bundles required only for reportable results
- minimal tests
- prompt logs only for key design steps

### Full mode (team / publishable)
- run bundles for every experiment
- richer evaluation tables + artifact index
- stronger tests and CI
- more formal decision records
- site/dashboard auto-generation

---

## 15) Definition of Done (Milestone)

A milestone is “done” only if:
- results are exported to `artifacts/` (not only notebook output)
- run bundles exist for final selected results
- `structure.md` references everything important
- assumptions and decisions are documented
- a short narrative report exists (markdown or site page)



# CURSOR_RULES.md

This repository follows a strict “artifact-first + documented-results” workflow.

The single source of truth for project structure and results is: `structure.md`.

All notebooks must:
1) save figures to `artifacts/figures/`
2) save any derived datasets (only when they are meaningful outputs) to `artifacts/data/`
3) update `structure.md` with findings + references to saved artifacts
4) avoid noisy printing and excessive comments

If any notebook is created/edited, `structure.md` MUST be updated in the same change.

--------------------------------------------------------------------
PROJECT LAYOUT (REQUIRED)
--------------------------------------------------------------------

Required folders:
- notebooks/                Jupyter notebooks only (analysis steps)
- src/                      reusable python code (functions/classes)
- artifacts/
  - figures/                all exported plots/figures (no inline-only)
  - data/                   exported CSV/Parquet of final/important derived datasets
  - models/                 serialized models (optional)
  - tables/                 exported tables as CSV or HTML (optional)
- structure.md              single source of truth for results + where things live
- README.md                 brief project intro, how to run
- environment.yml or requirements.txt

Rules:
- All paths referenced in docs must be relative paths inside the repo.
- Artifacts must be reproducible from notebooks. Do not hand-edit artifacts.
- Never save artifacts to notebook output cells only. Always export files.

--------------------------------------------------------------------
ARTIFACT RULES
--------------------------------------------------------------------

Figures:
- Save every “reportable” figure to `artifacts/figures/`
- Filename format:
  <step>__<short_description>__<YYYY-MM-DD>.png
  Example: eda__los_distribution_by_acuity__2026-01-23.png

- Use consistent size/dpi:
  - figsize chosen per plot, export with dpi=200 or dpi=300
- Every exported figure must be referenced in `structure.md` with a one-line purpose.

Derived datasets:
- Only save derived datasets that are reused later or represent a key modeling output.
- Save to `artifacts/data/` as CSV or Parquet.
- Filename format:
  <step>__<dataset_name>__<YYYY-MM-DD>.csv
  Example: preprocess__ed_arrivals_hourly__2026-01-23.csv

- Always include:
  - row count and column list in `structure.md`
  - brief description of what changed vs the input

Models:
- If a trained model is a deliverable, save to `artifacts/models/`
- Include training context in `structure.md` (data version, key hyperparameters)

--------------------------------------------------------------------
STRUCTURE.MD UPDATE RULES (MANDATORY)
--------------------------------------------------------------------

After completing a notebook (or making meaningful edits), append/update a section in `structure.md`:

For each notebook you MUST record:
- What the notebook did (1–3 sentences)
- Key outputs (figures + datasets) with RELATIVE PATHS
- Key findings (what changed, what matters operationally)
- Next steps / open questions (short, concrete)
- If applicable: assumptions or caveats

No dumping raw notebook output into `structure.md`.
Write human-readable summaries that reference artifacts.

--------------------------------------------------------------------
NOTEBOOK NAMING + ORDER
--------------------------------------------------------------------

Notebook filenames must be numbered and descriptive:

notebooks/
  01_eda.ipynb
  02_preprocess.ipynb
  03_model_arrivals.ipynb
  04_model_los.ipynb
  05_simulation_policy_eval.ipynb

Rules:
- Notebooks should run top-to-bottom without manual state.
- Use fixed random seeds where randomness affects outputs.
- Keep heavy reusable logic in `src/` and import it.

--------------------------------------------------------------------
CODE STYLE (STRICT)
--------------------------------------------------------------------

Printing:
- Do not spam print statements.
- No “big banner” prints (ALL CAPS headings, separators, etc.).
- Only print when it changes user decisions (e.g., final metric summary).
- Prefer returning objects and letting notebooks display them naturally.
- For statistical results tables, use the library’s built-in summary table
  (e.g., statsmodels summary) rather than custom verbose prints.

Comments:
- Keep comments minimal and functional.
- Avoid narrating obvious code.
- For every non-trivial function, include a short docstring:
  - what it does
  - inputs/outputs
  - key assumptions (if any)

Notebook cell structure:
- Use short cells with clear intent.
- Prefer clean variable names over excessive comments.
- Avoid deeply nested “do everything” cells; factor into functions in `src/`.

Outputs:
- If a plot or table is important, export it.
- If a dataset transformation matters, either export it or document it.

--------------------------------------------------------------------
MINIMUM “DONE” CHECKLIST FOR ANY NOTEBOOK CHANGE
--------------------------------------------------------------------

Before considering a notebook complete:
- Exported key figures to artifacts/figures/
- Exported any key derived datasets to artifacts/data/
- Added/updated the notebook section in structure.md
- Notebook runs top-to-bottom
- No noisy prints, no excessive comments
- Paths are relative and consistent

# structure.md

This file is the single source of truth for:
1) repository structure (what each file/folder is for)
2) results and artifacts produced by each notebook
3) references to exported figures and datasets
4) key findings and what they mean

All referenced paths are relative to repo root.

--------------------------------------------------------------------
PROJECT MAP
--------------------------------------------------------------------

Top-level:
- README.md
- CURSOR_RULES.md
- structure.md

Folders:
- notebooks/
  - 01_eda.ipynb: ...
  - 02_preprocess.ipynb: ...
  - 03_model_arrivals.ipynb: ...
  - 04_model_los.ipynb: ...
  - 05_simulation_policy_eval.ipynb: ...

- src/
  - <module>.py: ...

- artifacts/
  - figures/: exported figures (png)
  - data/: exported datasets (csv/parquet)
  - models/: saved model objects (optional)
  - tables/: exported tables (optional)

--------------------------------------------------------------------
ARTIFACT INDEX
--------------------------------------------------------------------

Figures (key reportable visuals):
- artifacts/figures/<file>.png — <what it shows, one line>

Datasets (key derived outputs):
- artifacts/data/<file>.csv — <what it contains, one line>

Models (if any):
- artifacts/models/<file> — <what it is, one line>

--------------------------------------------------------------------
RESULTS LOG (BY NOTEBOOK)
--------------------------------------------------------------------

## notebooks/01_eda.ipynb
Purpose:
- <1–3 sentences describing what was analyzed>

Inputs:
- <raw dataset paths / sources>

Outputs:
Figures:
- artifacts/figures/<...>.png — <purpose>
- artifacts/figures/<...>.png — <purpose>

Datasets (if saved):
- artifacts/data/<...>.csv — rows: <n>, cols: <n>; columns: <...>
  Notes: <what was created and why>

Key findings:
- <short paragraph: the main patterns that matter>
- <short paragraph: anything surprising / operationally relevant>

Assumptions / caveats:
- <short paragraph>

Next steps:
- <what the next notebook will do>

## notebooks/02_preprocess.ipynb
Purpose:
- ...

Inputs:
- ...

Outputs:
Figures:
- ...

Datasets:
- ...

Key findings:
- ...

Assumptions / caveats:
- ...

Next steps:
- ...

## notebooks/03_model_arrivals.ipynb
Purpose:
- ...

Inputs:
- ...

Outputs:
Figures:
- ...

Datasets:
- ...

Key findings:
- ...

Assumptions / caveats:
- ...

Next steps:
- ...

## notebooks/04_model_los.ipynb
Purpose:
- ...

Inputs:
- ...

Outputs:
Figures:
- ...

Datasets:
- ...

Key findings:
- ...

Assumptions / caveats:
- ...

Next steps:
- ...

## notebooks/05_simulation_policy_eval.ipynb
Purpose:
- ...

Inputs:
- ...

Outputs:
Figures:
- ...

Datasets:
- ...

Key findings:
- <policy tradeoff summary and selected operating point>

Assumptions / caveats:
- ...

Next steps:
- ...
