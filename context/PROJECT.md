# PROJECT

## Phase

- Current: 0-Planning / California-scale expansion scope
- Last updated: 2026-06-20
- Active task: TASK-001
- Blockers: Scope is still draft until Sardor approves the first California expansion plan.

## Project Summary

- Name: BayesTransitEquity
- Owner: Sardor Sobirov
- Audience: transit planners, regional/state transportation agencies, equity analysts, academic reviewers, and public-interest researchers.
- Decision or question supported: how transit accessibility equity decisions change when deterministic tract scores are replaced with uncertainty-aware risk, ambiguity, and intervention-responsiveness estimates.
- Smallest useful deliverable: a reproducible California pilot that compares deterministic accessibility, probabilistic transit-desert risk, and intervention priority for multiple representative regions.

## Core Premise

BayesTransitEquity is not just a map of transit deserts. The project argues that transit equity planning often treats noisy accessibility estimates as exact, then uses those point estimates for threshold-based policy decisions. A tract is not simply served or underserved. It has an accessibility distribution shaped by GTFS schedules, street networks, tract geometry, demographic need, opportunity locations, data quality, and model assumptions.

The long-run project is to build an uncertainty-aware planning framework:

- deterministic accessibility answers: "How much access does this tract appear to have?"
- probabilistic accessibility answers: "How confident are we that this tract is below a policy threshold?"
- ambiguity metrics answer: "Which tracts sit close enough to the boundary that a binary label is politically fragile?"
- intervention simulations answer: "Which investments are most likely to move people across meaningful thresholds?"
- statewide scaling answers: "Can this become a governance tool rather than a one-county case study?"

The paper contribution should be framed around threshold politics, uncertainty, and public decision-making, not around a claim that Bayesian methods always find completely different deserts.

## Status Table

| Area | Status | Notes |
| --- | --- | --- |
| Problem framing | draft | San Diego proof-of-concept exists; California policy scope needs approval. |
| Data card | pending | Statewide GTFS, ACS, TIGER, LODES, OSM inventory not yet audited. |
| EDA / data audit | pending | Must precede statewide modeling. |
| Deterministic baseline | pending | Needed before probabilistic layer. |
| Modeling strategy | draft | Recommended: approximate statewide inference plus MCMC validation slices. |
| Evaluation / QA | pending | Must include approximation-vs-MCMC calibration. |
| Visualization app | draft | Needs scale-aware state/region/county/tract flow. |
| Handover | pending | Will be updated after each completed phase. |

## Workstream Table

| Workstream | Task IDs | Status | Evidence |
| --- | --- | --- | --- |
| Scope and research framing | TASK-001 | in-progress | This planning document and `context/SCOPE.md`. |
| Data inventory and EDA | TASK-002, TASK-003, TASK-004 | pending | None yet for California. |
| Statewide deterministic pipeline | TASK-005, TASK-006 | pending | San Diego pipeline is the reference. |
| Scalable probabilistic modeling | TASK-007, TASK-008, TASK-009 | pending | San Diego BYM2 MCMC is the benchmark. |
| Intervention simulation | TASK-010 | pending | San Diego nb07 is the reference. |
| Frontend / map expansion | TASK-011 | pending | Redesign branch may be reviewed as a candidate base. |
| QA, documentation, handover | TASK-012, TASK-013 | pending | Local workflow files exist. |

## Decisions Log

| Date | Decision | Rationale | Follow-up |
| --- | --- | --- | --- |
| 2026-06-20 | Treat California expansion as staged platform + research program, not a direct full-state MCMC rerun. | Full-state MCMC first is likely too slow and too fragile; deterministic and approximate layers should guide where gold-standard MCMC is needed. | Draft scope and tasks; approve before implementation. |
| 2026-06-20 | Keep workflow control files under `context/` and ignored by Git. | Keeps repo root clean and avoids pushing local agent state. | Continue updating local `context/*` during planning. |
| 2026-06-20 | Use no AI co-author trailers. | Sardor wants commits to appear as single-author local commits. | Commit locally only unless asked to push. |

## Last Session

- Date: 2026-06-20
- AI tools used: Codex, GitHub connector, Sardor agentic workflow repo reference, brainstorming skill.
- Shipped: Local workflow shell under `context/`; initial California expansion planning underway.
- Verified: `context/` ignored by Git; no workflow files will push unless explicitly force-added.
- Blockers: Need Sardor approval on draft scope before implementation tasks begin.
- Next: Review `context/SCOPE.md` and `context/TASKS.md`; revise into approved plan.
