# PROJECT_BRIEF.md

## What We Are Building

A Bayesian spatial analysis pipeline and interactive web application for measuring transit
accessibility equity in San Diego, CA — designed from day one to scale to California and
eventually the full United States.

## The Problem

Standard transit equity tools produce deterministic point estimates (single accessibility scores
per neighborhood). This hides uncertainty that is especially large in data-sparse areas, and
may lead to systematically wrong conclusions about which neighborhoods are most underserved.

## What We Produce

1. **Posterior distributions** over accessibility per census tract — not scores, but full
   probability distributions with uncertainty quantified.
2. **P(transit desert | data)** — posterior probability of being underserved, accounting for
   demographic need, service supply, and data sparsity.
3. **Intervention posteriors** — how distributions shift under hypothetical route/frequency changes,
   giving planners probabilistic what-if analysis.
4. **Interactive Vercel map** — users explore neighborhoods, see posterior distributions,
   compare areas via Wasserstein distance, and simulate interventions.
5. **Academic paper** — arguing that uncertainty-aware analysis changes equity conclusions.

## Success Criteria

- [ ] End-to-end pipeline running for San Diego (GTFS → posteriors → GeoJSON → map)
- [ ] Bayesian model with proper convergence diagnostics (R-hat < 1.01, adequate ESS)
- [ ] Interactive map deployed on Vercel with at least: heatmap layer, posterior panel,
      credible interval layer, basic intervention slider
- [ ] At least one compelling empirical result: neighborhoods where Bayesian and deterministic
      methods give opposite equity conclusions
- [ ] Paper draft with methods + results sections complete

## Constraints

- All compute happens offline (precomputed posteriors). No live inference on Vercel.
- All data sources must be publicly available (GTFS, Census, OpenStreetMap).
- Code must be config-driven so SD → CA → National requires no rewrites.
- Paper figures must be traceable to run bundles.
