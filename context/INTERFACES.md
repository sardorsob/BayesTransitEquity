# INTERFACES.md
I/O contracts, data shapes, and file format specifications.
The bridge between the Python pipeline and the frontend is defined here.

---

## Pipeline Output: Tract-Level GeoJSON (Frontend Contract)

This is the schema that `scripts/export_frontend.py` must produce.
The frontend (`app/`) depends on this schema — changing it requires updating both.

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": { "type": "MultiPolygon", "coordinates": [...] },
      "properties": {
        "geoid": "06073000100",
        "tract_name": "Census Tract 1, San Diego County",
        "posterior_mean": 0.62,
        "posterior_sd": 0.14,
        "ci_lower_95": 0.35,
        "ci_upper_95": 0.88,
        "exceedance_prob": 0.73,
        "p_transit_desert": 0.41,
        "wasserstein_dist": 0.28,
        "entropy": 1.34,
        "pop_total": 4210,
        "pct_no_vehicle": 0.18,
        "median_income": 42000,
        "pct_poverty": 0.22
      }
    }
  ]
}
```

**Field definitions:**
- `geoid`: 11-digit census tract FIPS code (primary key)
- `posterior_mean`: Mean of accessibility posterior distribution
- `posterior_sd`: Standard deviation of posterior
- `ci_lower_95` / `ci_upper_95`: 95% credible interval bounds
- `exceedance_prob`: P(accessibility < threshold) — primary heatmap variable
- `p_transit_desert`: Posterior probability of being a transit desert
- `wasserstein_dist`: Wasserstein distance from well-served reference distribution
- `entropy`: Entropy of the accessibility posterior

---

## Pipeline Output: Posterior Samples Parquet

Path: `data/processed/posteriors/<run_id>_posterior_samples.parquet`

| Column | Type | Description |
|---|---|---|
| geoid | str | Census tract FIPS |
| sample_idx | int | MCMC sample index |
| accessibility | float | Sampled accessibility value |
| chain | int | MCMC chain index |

---

## Pipeline Output: Posterior Summary Parquet (wide, one row per tract)

Path: `data/processed/posteriors/<run_id>_posterior_summary.parquet`

Produced by `notebooks/04_bayesian_model.ipynb`. Column set may grow; core fields include:

| Column | Description |
|--------|-------------|
| GEOID | 11-digit tract FIPS |
| posterior_mean_log1p / posterior_sd_log1p | Mean and SD of **μ** on the **log1p(jobs)** scale |
| ci_lower_95_log1p / ci_upper_95_log1p | 95% credible interval for **μ** (log1p scale) |
| posterior_mean_jobs | `expm1(posterior_mean_log1p)` — point comparison to deterministic nb03 map |
| posterior_sd_jobs_delta | Delta-method SD on job scale, `exp(μ_mean) * σ_log` |
| exceedance_prob_30min / _45min / _60min | P(μ < log1p(Q25 of observed jobs at that threshold)) — **sensitivity** on desert cut; 30/60 use the same **μ** as the primary fit |
| p_transit_desert | Same as exceedance at `accessibility.travel_time_threshold_min` (e.g. `_45min`) |
| B01003_001E, poverty_rate, … | Raw ACS fields for `export_frontend.py` |

---

## Intervention Scenarios GeoJSON

Path: `data/processed/geojson/scenarios/<scenario_id>.geojson`

Same schema as the base GeoJSON but with `_scenario` suffix fields showing the
posterior shift under the intervention:
- `exceedance_prob_scenario`: Updated exceedance probability
- `exceedance_prob_delta`: Change from baseline
- `p_transit_desert_scenario`: Updated transit desert probability

---

## Config Schema

`configs/san_diego.yaml` structure:

```yaml
city: san_diego
state_fips: "06"
county_fips: "073"
bbox: [-117.40, 32.53, -116.80, 33.35]  # [min_lon, min_lat, max_lon, max_lat] — D009 expanded

gtfs_agencies:
  - id: mts
    name: San Diego MTS
    url: https://www.sdmts.com/...
  - id: nctd
    name: North County Transit District
    url: https://www.gonctd.com/...

accessibility:
  travel_time_threshold_min: 45
  departure_window_start: "07:00"
  departure_window_end: "09:00"
  opportunities: [jobs, hospitals, groceries]

model:
  sampler: mcmc  # or: vi (variational inference)
  draws: 2000
  tune: 1000
  chains: 4
  target_accept: 0.9
```
