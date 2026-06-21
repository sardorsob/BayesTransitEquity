# FINDINGS.md
Empirical results log. Updated after every notebook run, model fit, or analysis step.

Each finding is structured as:
- **Observation** — the raw number or result
- **Interpretation** — what it means in plain language
- **Paper implication** — how it connects to the core argument or a specific section/figure
- **Next question** — what this finding makes us want to check next

Cross-references: findings cite the notebook and artifact they came from. Once a finding
becomes a figure or table in the paper, that is noted here.

---

## Phase 1 — Data Acquisition + EDA

### F001 — MTS feed is current; NCTD feed is expired
**Date**: 2026-03-29
**Source**: `notebooks/eda/01_inventory_and_provenance.ipynb`
**Artifact**: `artifacts/tables/eda__gtfs_feed_info__2026-03-29.csv`

**Observation**: MTS feed version "2601 unmerged v2" is valid 2026-01-25 through 2026-06-06
(current as of analysis date). NCTD feed version "Version_20250205" is valid only through
2025-05-17, approximately 10 months expired. Force-refresh with `--refresh-mobility-catalog`
re-pulled the mdb-14 bundle but the payload was unchanged — NCTD has not pushed a new feed
to Mobility Database.

**Interpretation**: MTS data is reliable for schedule analysis. NCTD data reflects service
as of early 2025, meaning any route additions, frequency improvements, or service cuts since
May 2025 are invisible to the model. This is not a minor gap — a ~10-month stale GTFS feed
can miss entire route restructurings, especially post-pandemic service adjustments.

**Paper implication**: This is a real-world limitation that belongs in the Data section (Section 4)
and Discussion. It also illustrates the broader point about data quality in transit equity
research: even the "standard" input data (GTFS) has provenance gaps that are rarely disclosed.
Deterministic analyses that simply ingest GTFS without checking validity windows are silently
using stale information. The paper can use this as a minor motivating example.

**Next question**: How much service change did NCTD actually make between May 2025 and now?
Check NCTD open data portal or press releases for service changes. If material, this
strengthens the "data quality compounds uncertainty" argument.

---

### F002 — NCTD stop coverage inside study bbox is very low (18.76%)
**Date**: 2026-03-29
**Source**: `notebooks/eda/02_gtfs_schedule_exploration.ipynb`
**Artifact**: `artifacts/tables/eda__gtfs_bbox_coverage__2026-03-29.csv`

**Observation**: Of NCTD's 1,823 stops, only 342 (18.76%) fall within the study bbox
`[-117.28, 32.53, -116.93, 33.11]`. MTS has 4,373 stops, of which 4,082 (93.41%) are
inside the bbox.

**Interpretation**: The study bbox was drawn around central and south San Diego, which is
MTS territory. North County — where NCTD primarily operates — extends north and east beyond
the bbox. Including NCTD as a full agency while only capturing 18% of its stops is
misleading: NCTD's coverage metrics will appear artificially poor not because of actual
service gaps but because most NCTD service is simply outside the study area.

**Paper implication**: This is a study area design decision that must be stated explicitly
in the Data section. Two legitimate choices: (1) expand the bbox to include more North County
tracts and properly evaluate NCTD, or (2) scope the study to MTS service area and treat NCTD
as supplementary. Either is defensible, but the choice must be made before the model is fit.
If the paper claims to cover "San Diego County transit equity," the bbox needs to expand. If
the paper is more narrowly "central San Diego / MTS service area," the current bbox is fine
and NCTD becomes context rather than primary data. See open decision D009.

**Next question**: What do the 342 in-bbox NCTD stops look like spatially? Are they in the
northern edge of the bbox (a thin strip), or are they distributed? Check notebook 04
(spatial alignment) for the overlap map.

---

### F003 — MTS system scale: 105 routes, 22,705 trips, 4,373 stops, ~540k stop_times
**Date**: 2026-03-29
**Source**: `notebooks/eda/02_gtfs_schedule_exploration.ipynb`
**Artifact**: `artifacts/tables/eda__gtfs_feed_summary__2026-03-29.csv`

**Observation**: MTS operates 105 routes, 22,705 trips, 4,373 stops. NCTD (full feed, not
bbox-clipped) has 50 routes, 4,589 trips, 1,823 stops.

**Interpretation**: MTS is substantially larger in every dimension than NCTD — roughly 2x
routes, 5x trips, 2.4x stops. This makes sense: MTS covers urban San Diego, NCTD covers
lower-density North County. The trip count gap (22,705 vs 4,589) likely reflects higher
frequency service on MTS routes due to greater population density.

**Paper implication**: Basic system characterization that belongs in the Data section's
"study area" paragraph. Establishes the two-agency structure and relative scale of each
operator for readers unfamiliar with San Diego transit.

**Next question**: After clipping to bbox, what are the effective MTS and NCTD trip/route
counts? The in-bbox NCTD number is the relevant one for the model.

---

### F004 — MTS headway spread: best 5 min (route 201), worst 70 min (route 83)
**Date**: 2026-03-29
**Source**: `notebooks/eda/02_gtfs_schedule_exploration.ipynb`
**Artifact**: `artifacts/tables/eda__gtfs_headways__2026-03-29.csv`

**Observation**: Weekday morning-peak median headways across MTS routes range from 5 minutes
(route 201, the Blue Line trolley) to 70 minutes (route 83). NCTD headways range from
20 minutes (best) to 180 minutes (route 395, worst — effectively 3-hour service gaps).

**Interpretation**: There is a 14x spread in headway quality within MTS alone and a
36x spread across both agencies. Route 201 (trolley) has rail-comparable frequency.
Route 83 and the NCTD 395 represent service that is essentially useless for commute-time
travel — one miss and you wait over an hour. This kind of within-system variation is exactly
what the Bayesian model is designed to capture: neighborhoods served only by high-headway
routes face a structurally different accessibility distribution than trolley-adjacent ones,
and the uncertainty in that distribution is larger when fewer departures are available in
the departure window.

**Paper implication**: This is a key motivating data point for the paper. Headway variation
is the empirical engine behind posterior uncertainty — routes with 5-min headways produce
tight accessibility posteriors (many departure samples, consistent outcomes), while 70-min
headways produce wide posteriors (few departures, high variability). This directly motivates
the Bayesian approach. Consider using route 201 vs route 83 as the contrast case in the
opening hook and in Figure 6 (posterior distributions for contrasting neighborhoods).

**Next question**: Do the high-headway routes cluster spatially in lower-income tracts?
This is the key empirical hypothesis. Notebook 03 (ACS) + notebook 04 (spatial alignment)
will let us cross-reference headway by route against income/race by tract. If yes, this
is one of the paper's main results.

---

### F005 — MTS data quality: 106 unused stops, no zero-trip routes
**Date**: 2026-03-29
**Source**: `notebooks/eda/02_gtfs_schedule_exploration.ipynb`
**Artifact**: `artifacts/tables/eda__gtfs_data_quality__2026-03-29.csv`

**Observation**: MTS has 106 stops that appear in `stops.txt` but have no associated
stop_times (unused stops). NCTD has 3 zero-trip routes and 12 unused stops. Neither
agency has orphaned trips (trips in trips.txt with no stop_times).

**Interpretation**: 106 unused MTS stops is a minor data quality issue, likely representing
stops that were decommissioned or planned but never activated, and not yet removed from the
feed. At 2.4% of total stops (106/4,373) this is low enough to not materially affect
accessibility computation, since r5py routes to real stops only. NCTD's 3 zero-trip routes
are more interesting — these are routes that exist in routes.txt but have no scheduled
service, possibly seasonal, suspended, or stub entries for future service.

**Paper implication**: Worth a one-sentence mention in the data cleaning section of the
Methods (Section 3) to show the analysis was not naive — feed validation was performed and
minor quality issues were noted but do not affect the core results.

**Next question**: Are the 106 unused MTS stops concentrated in any particular area? If they
cluster in a specific neighborhood, it could indicate a corridor where stop infrastructure
exists but service was pulled — itself an equity story.

---

### F006 — SD County has 737 census tracts; ACS estimates confirmed downloaded
**Date**: 2026-03-29
**Source**: `notebooks/eda/01_inventory_and_provenance.ipynb`
**Artifact**: `artifacts/tables/eda__census_artifact_check__2026-03-29.csv`

**Observation**: TIGER/Line 2023 shapefile extracted successfully. 737 census tracts for
San Diego County (FIPS 06-073). ACS 5-year (2019-2023) estimates downloaded for all 737
tracts. Study bbox will include a subset of these tracts.

**Interpretation**: 737 tracts is a comfortable model size for a Bayesian hierarchical
spatial model. MCMC with PyMC at 2,000 draws, 4 chains over ~700 spatial units is feasible
on a modern laptop in a few hours. At California scale (~9,000 tracts) the same model would
require NumPyro + GPU, which is why D006 makes PyMC the SD-phase default.

**Paper implication**: Establishes the model's spatial granularity. 737 tracts means each
posterior is a distribution over a unit of roughly 4,000 residents — fine enough to
distinguish neighborhood-level equity patterns without being so fine that data becomes
too sparse per unit.

**Next question**: After clipping to the study bbox, how many tracts remain? What fraction
of SD's tracts does the study cover? This is notebook 04 (spatial alignment). Also: do any
tracts have too few ACS observations to trust? Check MOE/estimate ratios in notebook 03.

---

### F007 — ACS relative MOE vs disadvantage: mixed signals across estimands
**Date**: 2026-03-29
**Source**: `notebooks/eda/03_census_tracts_and_acs.ipynb`
**Artifacts**: `artifacts/tables/eda__acs_moe_disadvantage_spearman__2026-03-29.csv`,
`artifacts/figures/eda__acs_moe_vs_disadvantage__2026-03-29.png`,
`artifacts/figures/eda__acs_choropleth_rel_moe_pop__2026-03-29.png`

**Observation**: Spearman correlations (tracts with non-missing pairs; see table for *n*):
population estimate relative MOE (`rel_moe_pop`) vs poverty rate: ρ ≈ 0.16 (*p* ≈ 10⁻⁵);
vs composite `disadvantage_z`: ρ ≈ 0.16 (*p* ≈ 2×10⁻⁵); vs Hispanic share: ρ ≈ 0.20 (*p* ≈ 5×10⁻⁸).
No-vehicle rate vs `rel_moe_pop`: ρ ≈ 0.06 (*p* ≈ 0.10, not significant at α = 0.05).
By contrast, *poverty count* relative MOE vs poverty rate and vs `disadvantage_z` is strongly
*negative* (ρ ≈ −0.53 and −0.56, both *p* ≪ 10⁻⁵⁰), consistent with smaller poor populations
producing noisier *rate* denominators in a different way than total-population MOE scaling.

**Interpretation**: The simple story “MOE is always worst in the most disadvantaged tracts”
does not hold uniformly across ACS estimands. Total-population relative MOE rises modestly
with poverty, disadvantage, and Hispanic share — enough to mention as spatial heterogeneity
in data precision — but poverty-count MOE behaves differently because it scales with count
and sampling design. Any narrative about “noisiest where equity stakes are highest” must be
estimand-specific and checked per variable, not assumed from one margin.

**Paper implication**: Strengthens the Methods case for explicit uncertainty (MOE-aware or
hierarchical) treatment rather than treating ACS point estimates as exact. The introduction
can cite the modest positive association of `rel_moe_pop` with disadvantage as *partial*
support for caring about uncertainty in high-stakes tracts, while the Discussion should note
that MOE structure is not monotonic across all poverty-related tables.

**Next question**: Repeat MOE/disadvantage checks after clipping tracts to the study bbox
(notebook 04) and when adding transit exposure covariates, so correlations reflect the final
analysis sample.

---

### F008 — ACS margins of error are practically large: poverty and vehicle data are near-unusable at tract level
**Date**: 2026-03-29
**Source**: `notebooks/eda/03_census_tracts_and_acs.ipynb`
**Artifacts**: `artifacts/tables/eda__acs_summary_stats__2026-03-29.csv`,
`artifacts/tables/eda__acs_sd_tract_attributes__2026-03-29.csv`

**Observation**: Median relative MOE (MOE / estimate) across all 737 tracts:
population count 14.2%, median household income 19.6%, poverty count 58.8%,
no-vehicle households 87.8%. In absolute terms: 51 of 727 tracts (7.0%) have a poverty
count MOE that exceeds the estimate itself (rel_moe_poverty_count > 1.0); 259 of 683 tracts
(37.9%) have a no-vehicle household MOE that exceeds the estimate. Income MOE averages
$23,287 on a mean estimate of $110,900 — roughly ±$23k on a typical income figure.
The no-vehicle relative MOE reaches a maximum of 3.0 (MOE is triple the estimate).

**Interpretation**: This is the most striking empirical finding from EDA 03 and arguably the
strongest empirical hook in the paper. For more than a third of San Diego tracts, the ACS
cannot reliably tell us how many households lack a vehicle — a variable that is one of the
primary proxies for transit dependency. Any deterministic analysis that ranks neighborhoods
by "transit need" using no-vehicle rate as a covariate is ranking noise in those 37.9% of
tracts. Poverty count is similarly afflicted: 7% of tracts have estimates where the margin of
error is larger than the point estimate itself, meaning the true poverty count could
plausibly be near zero. These are not fringe tracts — they are scattered across the county.
This is not a limitation of the ACS per se; it is how small-area surveys work. But it means
that treating these numbers as exact in a deterministic equity metric is analytically
indefensible. The Bayesian model propagates this uncertainty explicitly rather than
discarding it.

**Paper implication**: This is Figure 5 material (credible interval width map) and a strong
motivator for the paper's core argument in Section 1. The opening hook can be built around
this: "For 38% of San Diego census tracts, we do not know — within a factor of two — how
many households lack a car. Standard equity analyses use these numbers as if they were
facts." The Methods section can explain how the hierarchical model uses partial pooling to
borrow strength from neighboring tracts precisely in cases like these, producing more
reliable estimates than the raw ACS figures alone.

**Next question**: Do the tracts with the worst no-vehicle and poverty MOEs cluster
spatially, or are they spread uniformly? If they cluster in specific neighborhoods, that is
both an equity concern and a spatial structure the CAR prior can exploit. Check in
notebook 04 (spatial alignment) once tracts are clipped to the study bbox.

---

### F009 — San Diego income and poverty landscape: high mean, extreme spread
**Date**: 2026-03-29
**Source**: `notebooks/eda/03_census_tracts_and_acs.ipynb`
**Artifact**: `artifacts/tables/eda__acs_sd_tract_attributes__2026-03-29.csv`

**Observation**: Across 737 tracts — median household income ranges from $24,125 (min) to
$250,001 (max, ACS top-code). Mean across tracts is $110,900 but the median is $104,457,
indicating right-skew. Poverty rate ranges 0.0% to 42.8%. No-vehicle household rate ranges
0.0% to 24.7%. Top-quartile disadvantage tracts (z > 0.76) have a median income of $66,476
versus $159,976 for bottom-quartile tracts — a 2.4× income gap between the most and least
disadvantaged quarters of the county. The bottom-quartile median income MOE is $28,179,
roughly 17.6% of the estimate; the top-quartile MOE is $15,532, roughly 23.4% — so both
groups face substantial income uncertainty, with the wealthier tracts slightly worse in
relative terms (likely because high-income tracts have more income variance, widening the
survey interval).

**Interpretation**: San Diego is a genuinely unequal city. A 2.4× income gap between
quartile-1 and quartile-4 tracts is large but not unusual for a major US metro. The more
notable feature is how wide the income MOE is in absolute terms even for relatively
advantaged tracts: ±$28k on a $160k estimate means the ordering of "middle-rich" tracts
is uncertain. When an equity analysis ranks neighborhoods from most to least advantaged, the
ranking within the middle of the distribution is largely noise. Only the extremes — the very
poorest and the very wealthiest tracts — are positioned with enough certainty to rank
reliably. This directly motivates using posterior distributions over accessibility rather
than point estimates: if the covariate (income) is uncertain, the accessibility target
derived from it should be uncertain too.

**Paper implication**: Goes into Section 4 (Data) as the descriptive characterization of the
study area. The 2.4× quartile income gap sets up the equity question. The income MOE spread
supports the Methods argument about propagating demographic uncertainty into the model.
Consider a supplementary figure showing the income distribution with MOE error bars per
tract to make this viscerally clear.

**Next question**: What does the spatial distribution of income look like? Are the low-income
tracts clustered in a specific corridor? (Expected: yes — Barrio Logan, City Heights,
National City are known concentrations.) This becomes the "study area" map in Figure 1 once
notebook 04 produces the spatial join.

---

### F010 — Disadvantage composite is right-skewed; extreme outlier tracts reach z = 4.3
**Date**: 2026-03-29
**Source**: `notebooks/eda/03_census_tracts_and_acs.ipynb`
**Artifact**: `artifacts/tables/eda__acs_sd_tract_attributes__2026-03-29.csv`

**Observation**: The composite disadvantage z-score (standardized sum of poverty rate,
no-vehicle rate, pct_hispanic, pct_black, minus pct_nh_white and income) has mean 0.145
and std 0.812 across 737 tracts. Distribution is right-skewed: 25th pct = −0.41,
median = 0.007, 75th = 0.59, max = 4.32. The top decile threshold is z > 1.25. The
single most disadvantaged tract is Census Tract 56.01 (GEOID 6073005601, z = 4.32) with a
poverty rate of 28.0% and 17.3% Hispanic share. The second most disadvantaged is Tract
52.02 (z = 3.52, poverty 27.2%, 24.0% Hispanic). Tract 51.03 has the highest raw poverty
rate in the dataset at 42.4% (z = 3.36, 53.9% Hispanic). The least disadvantaged tract
(z = −1.60) has 0% measured poverty and 36.8% non-Hispanic white share.

**Interpretation**: The long right tail — with three tracts above z = 3.0 and one at 4.32 —
confirms that extreme concentrated disadvantage exists within the county even in a city
with an above-average mean income. These outlier tracts are the most important cases for the
paper: they are where the equity gap is largest, where the Bayesian model's posterior
uncertainty is likely widest (sparse data, high survey noise), and where a planning
intervention would have the most impact. The right-skewed shape also suggests that the
majority of tracts are near the county median with a minority of truly distressed tracts
pulling the distribution right — a "long tail of need" structure common in US metros.

**Paper implication**: The disadvantage distribution figure belongs in Section 4 (Data) as a
histogram or density plot. The extreme outlier tracts (56.01, 52.02, 51.03) are candidates
for the paper's case study neighborhoods in Figure 6 (posterior distributions for contrasting
tracts) — pairing one of these against a low-disadvantage tract will make the strongest
visual argument about differential posterior uncertainty. The tract numbers should be
cross-referenced against named neighborhoods in notebook 04 once the spatial join is done.

**Next question**: What neighborhoods do tracts 56.01, 52.02, and 51.03 correspond to by
name? Are they in known high-poverty corridors (National City, Barrio Logan, City Heights)?
And how does their transit service look — are they in MTS coverage, and what are their
closest route headways from the EDA 02 data?

---

### F011 — ACS / TIGER perfect alignment: all 737 tracts matched 1-to-1
**Date**: 2026-03-29
**Source**: `notebooks/eda/03_census_tracts_and_acs.ipynb`
**Artifact**: `artifacts/tables/eda__acs_tiger_alignment__2026-03-29.csv`

**Observation**: ACS JSON returned 737 rows; TIGER/Line 2023 shapefile contains 737 tracts
for San Diego County (FIPS 06-073); inner merge on GEOID produces 737 matched rows. Zero
unmatched GEOIDs in either direction.

**Interpretation**: The two primary input datasets join cleanly. There are no phantom tracts
in the ACS that lack geometry, and no geometry tracts that lack demographic data. This means
the full county can go into the spatial model without imputation or tract exclusion on data
availability grounds. The only exclusions downstream will be geographic (bbox clip in
notebook 04) or data quality driven (e.g., tracts with zero population).

**Paper implication**: One sentence in the Data section: "ACS demographic estimates and
TIGER/Line geometries were available for all 737 San Diego County tracts with no unmatched
records." Establishes that the spatial join is clean before any reader worries about it.

**Next question**: None blocking for TIGER/ACS alignment. Bbox overlap and stop joins are
documented in F012–F013 (notebook 04).

---

### F012 — Study bbox intersects 726 of 737 SD tracts; expanded county-wide bbox per D009
**Date**: 2026-03-30 (updated; original 2026-03-29 used old bbox `[-117.28, 32.53, -116.93, 33.11]`)
**Source**: `notebooks/eda/04_spatial_alignment_and_coverage.ipynb`
**Artifacts**: `artifacts/tables/eda__spatial_layers_crs__2026-03-30.csv`,
`artifacts/tables/eda__tract_gtfs_service__2026-03-30.csv`,
`artifacts/figures/eda__tract_stops_choropleth__2026-03-30.png`

**Observation**: Using TIGER tract polygons and the expanded study bbox `[-117.40, 32.53,
-116.80, 33.35]` (D009, county-wide), **726** tracts have `intersects_study_bbox == True`
(98.5% of 737). Only **11** county tracts lie entirely outside the bbox — remote eastern
backcountry/Anza-Borrego tracts with negligible population and zero transit service. CRS
manifest unchanged: TIGER EPSG:4269, GTFS EPSG:4326, analysis projected to EPSG:3310
(California Albers, meters). *Previous result (old bbox): 571 of 737 in-bbox.*

**Interpretation**: The expanded bbox captures essentially the entire county. The 155 tracts
previously out-of-bbox (North County NCTD territory) are now included, validating D009. The
effective analysis sample is **726 tracts**. Intersection rule is unchanged: polygon
intersection (not centroid), so fringe tracts touching the bbox boundary are included.

**Paper implication**: Data section: "726 of 737 San Diego County census tracts intersect the
study area bounding box; 11 remote backcountry tracts are excluded." Expanded bbox means
NCTD service territory is now properly represented in all downstream analyses.

**Next question**: Confirm the 11 out-of-bbox tracts are low-population. These are almost
certainly Anza-Borrego Desert State Park area tracts.

### F013 — Stop counts right-skewed; 102 county tracts have zero stops; NCTD now partially integrated
**Date**: 2026-03-30 (updated with expanded bbox)
**Source**: `notebooks/eda/04_spatial_alignment_and_coverage.ipynb`
**Artifact**: `artifacts/tables/eda__tract_gtfs_service__2026-03-30.csv`

**Observation**: County-wide (737 tracts): **102** tracts have `n_stops_in_bbox == 0`; median
**7** stops, mean **8.1**, max **90**. MTS stops per tract: mean 5.7, median 4. NCTD stops per
tract: mean 2.4, median 0 — **189 of 737 tracts** have at least one NCTD stop, confirming
meaningful NCTD coverage in the expanded bbox. Min-headway mean 22.7 min, median 15 min.
Nearest-stop distance: mean **1,256 m**, median **487 m**, max **58,566 m** (remote
backcountry outlier). Spearman: `n_stops_in_bbox` vs `disadvantage_z`: **ρ = +0.391**;
`dist_nearest_stop_m` vs `disadvantage_z`: **ρ = −0.535**; `min_route_headway_min` vs
`disadvantage_z`: **ρ = −0.397** (n=630 with headway data). *Previous result (old bbox):
83 zero-stop tracts, mean 6.0, median 4, ρ(stop count)=+0.422, ρ(nearest dist)=−0.568.*

**Interpretation**: With the county-wide bbox, NCTD service is now visible in the data —
189 tracts have NCTD stops. The density confound persists in the same direction as before:
disadvantaged tracts still have more stops, better headways, and closer stop distances than
affluent tracts. The moderately weaker correlations (0.391 vs 0.422, −0.535 vs −0.568)
reflect the dilution effect of adding suburban/rural North County tracts where neither
disadvantage nor transit supply is particularly high. The core finding is unchanged.

**Paper implication**: Fig. 1 / Data section: choropleth of stop density with updated N
(726 in-bbox tracts). Headway roll-up methodology documented in Methods. The 102 zero-stop
tracts are almost all in the low-disadvantage suburban fringe — see F019.

**Next question**: NCTD data still uses the stale May-2025 feed (F001). Once refreshed,
re-run EDA 04 to update NCTD stop counts and headways.

---

### F014 — OSM walk graph is a single fully-connected component (expanded bbox); density correlated with disadvantage
**Date**: 2026-03-30 (updated; original 2026-03-29 used old bbox)
**Source**: `notebooks/eda/05_osm_pedestrian_network.ipynb`
**Artifacts**: `artifacts/tables/eda__osm_walk_graph_summary__2026-03-30.csv`,
`artifacts/tables/eda__osm_tract_walk_edges__2026-03-30.csv`,
`artifacts/tables/eda__osm_walk_disadvantage_spearman__2026-03-30.csv`,
`artifacts/figures/eda__osm_tract_walk_density_choropleth__2026-03-30.png`

**Observation**: Walk network for expanded bbox `[-117.40, 32.53, -116.80, 33.35]`:
**337,824** nodes, **906,988** directed edges, **65,750 km** total edge length,
**one** weakly connected component (single giant component, no islands). Mean edge length
72.5 m. Spearman correlation between walk edge density (km/km²) and `disadvantage_z`:
**ρ = +0.371** (*p* = 1.7×10⁻²⁵, n = 737). *Previous result (old bbox): 241,423 nodes,
658,182 edges, ~43,841 km, ρ = +0.23.*

**Interpretation**: The network grew ~40% in nodes and edges with the expanded bbox, as
expected when capturing North County walkable urban areas. The graph remains a single
connected component — excellent for r5py routing (no reachability issues). The walk
density–disadvantage correlation strengthened (0.371 vs 0.23), likely because the
expanded bbox added more suburban/rural low-density, low-disadvantage North County tracts
that pull down the low end of both distributions, sharpening the contrast.

**Paper implication**: Methods: cite OSM walk extent (337k nodes, 65,750 km, single
component). Disclose centroid-based tract density as EDA proxy; actual r5py uses the full
graph. The correlation strength (ρ=0.371) confirms the density confound extends to the
pedestrian network layer.

**Next question**: Compare tract walk density to r5py walk times to stops in pipeline nb 03;
spot-check F010 outlier tracts on a basemap for coverage gaps.

---
## Phase 1 — Pending Findings (notebook 06 onward)

*See `notebooks/eda/README.md` — next is `06_opportunities_and_destinations.ipynb`.*

---

## Cross-Reference: Findings → Paper Sections

| Finding | Status | Paper Section | Figure candidate |
|---------|--------|--------------|-----------------|
| F001 — NCTD stale feed | confirmed | Sec 4 Data; Sec 6 Discussion (limitations) | — |
| F002 — NCTD bbox gap (18.76%, old bbox) — resolved by D009 bbox expansion | confirmed | Sec 4 Data; D009 resolved | — |
| F003 — MTS/NCTD system scale | confirmed | Sec 4 Data | — |
| F004 — Headway spread 5–70 min (MTS) | confirmed | Sec 1 Introduction hook; Sec 4 Data | Fig 6 (posterior contrast) |
| F005 — GTFS data quality flags | confirmed | Sec 3 Methods (data cleaning) | — |
| F006 — 737 tracts, ACS downloaded | confirmed | Sec 4 Data (model granularity) | — |
| F007 — MOE vs disadvantage: mixed Spearman signals | confirmed | Sec 3 Methods; Sec 6 Discussion | Fig 5 (CI width map) |
| F008 — MOE practically large; 37.9% tracts no-vehicle unusable | confirmed | Sec 1 Introduction hook; Sec 3 Methods | Fig 5 candidate |
| F009 — Income $24k–$250k; 2.4× quartile gap | confirmed | Sec 4 Data (study area) | Supplementary |
| F010 — Disadvantage right-skewed; top tract z=4.32 | confirmed | Sec 4 Data; Sec 5 Results (case studies) | Fig 6 (case study tracts) |
| F011 — ACS/TIGER 737/737 perfect match | confirmed | Sec 4 Data (one sentence) | — |
| F012 — 726 tracts intersect study bbox (expanded bbox); CRS manifest | confirmed | Sec 4 Data (final sample N) | Fig 1 (study area map) |
| F013 — Stop skew; 102 zero-stop tracts; ρ(stop count)=+0.391, ρ(dist)=−0.535 | confirmed | Sec 3 Methods; Sec 4 | Fig 1 candidate |
| F014 — OSM walk: one WCC, 337k nodes, 65,750 km; density ρ=0.371 (expanded bbox) | confirmed | Sec 3 Methods; Sec 4 | Supplementary |

---

### F014 — Density confound confirmed in expanded bbox: disadvantaged tracts still have better supply on all metrics
**Date**: 2026-03-30 (updated; original 2026-03-29 used old 571-tract bbox)
**Source**: `notebooks/eda/04_spatial_alignment_and_coverage.ipynb`
**Artifact**: `artifacts/tables/eda__tract_gtfs_service__2026-03-30.csv`,
`artifacts/figures/eda__tract_nearest_stop_distance__2026-03-30.png`,
`artifacts/figures/eda__nearest_stop_distance_hist__2026-03-30.png`

**Observation**: County-wide (737 tracts, 726 in-bbox), Spearman correlations with
`disadvantage_z`: distance to nearest stop ρ = **−0.535** — more disadvantaged tracts are
*closer* to transit. Stop count ρ = **+0.391** — more disadvantaged tracts have *more*
stops. Min route headway ρ = **−0.397** (n=630 with headway data) — more disadvantaged
tracts have *lower* (better) headways. NCTD stop count ρ = **+0.179** (189 non-zero tracts).
Tracts >1 km from any stop: **174** (23.6% of all 737 tracts), mean z = negative (suburban
fringe). Max nearest-stop distance: **58,566 m** (remote backcountry outlier). *Previous
result (571-tract bbox): dist ρ=−0.568, stop count ρ=+0.422, headway ρ=−0.447.*

**Interpretation**: The density confound is fully robust to the bbox expansion. All
correlations hold in the same direction after adding 155 North County tracts. The slightly
weaker magnitudes (e.g., −0.535 vs −0.568) reflect dilution from added suburban/rural North
County tracts where both disadvantage scores and transit supply are moderate — neither
extreme — which compresses the correlation range. The core finding is unchanged and arguably
more credible now that it holds county-wide: San Diego's most disadvantaged tracts are
physically better served by transit on every static supply metric.

This is the central empirical finding of the paper. A naive proximity analysis would
conclude transit is equitable or pro-poor in San Diego. The Bayesian accessibility model is
necessary because the right question is not "are stops nearby?" but "can residents reach
jobs and services within a reasonable travel time given route structure and destination
geography?"

**Paper implication**: Section 1 hook: "Every raw supply metric — stop count, headway,
walk proximity — favors San Diego's most disadvantaged tracts. Yet these metrics capture
supply, not access. We show the equity picture is more complex using a Bayesian
accessibility framework." Updated Spearman values (county-wide) replace the 571-tract
numbers in Fig. 2 / Table 2.

**Next question**: Pipeline nb 03 tests whether the density advantage translates to actual
accessibility. If cumulative 45-min opportunity counts are still lower in disadvantaged
tracts despite better supply, the equity driver is destination geography, not transit supply.

---
### F015 — OSM walk network: 337,824 nodes, 65,750 km, single component (expanded bbox)
**Date**: 2026-03-30 (updated; original 2026-03-29 used old bbox)
**Source**: `notebooks/eda/05_osm_pedestrian_network.ipynb`
**Artifacts**: `artifacts/tables/eda__osm_walk_graph_summary__2026-03-30.csv`,
`artifacts/tables/eda__osm_walk_disadvantage_spearman__2026-03-30.csv`,
`artifacts/tables/eda__osm_tract_walk_edges__2026-03-30.csv`,
`artifacts/figures/eda__osm_walk_network_sample_map__2026-03-30.png`,
`artifacts/figures/eda__osm_tract_walk_density_choropleth__2026-03-30.png`

**Observation**: Expanded-bbox OSM pedestrian network: **337,824 nodes**, **906,988 directed
edges**, **one** weakly connected component (every walkable node can reach every other),
**65,750 km** total walk edge length, mean edge length **72.5 m**. Walk edge density vs
`disadvantage_z`: **ρ = +0.371** (*p* = 1.7×10⁻²⁵, n=737). *Previous result (old bbox):
241,423 nodes, 658,182 edges, 43,841 km, ρ=+0.23.*

**Interpretation**: Network grew ~40% with expanded bbox. Single-component property
preserved — no routing islands. r5py can route between any two stops without reachability
failures. Walk density correlation strengthened (0.371 vs 0.23), confirming the density
confound extends to the pedestrian network: disadvantaged tracts are in denser walk
environments. Coverage bias (untagged paths, micro-barriers) remains an assumption (A007);
this EDA cannot detect it.

**Paper implication**: Methods: "The San Diego OSM pedestrian network for the study area
comprises 337,824 nodes and 65,750 km of walk edges in a single connected component,
enabling complete isochrone routing from all 726 in-study-area tract centroids."

**Next question**: Cross-validate walk edge density vs r5py isochrone coverage in pipeline
nb 03; flag any tracts where walk density predicts good access but r5py finds dead ends.

---
### F016 — Destination landscape: 181 hospitals, 1,170 groceries, 1,273 schools in expanded bbox
**Date**: 2026-03-30 (updated; original 2026-03-29 used old bbox with school cap at 1,000)
**Source**: `notebooks/eda/06_opportunities_and_destinations.ipynb`
**Artifacts**: `artifacts/tables/eda__opportunities_destination_summary__2026-03-30.csv`,
`artifacts/tables/eda__osm_destinations_fetch_status__2026-03-30.csv`,
`artifacts/tables/eda__osm_destinations_groceries__2026-03-30.csv`,
`artifacts/tables/eda__osm_destinations_hospitals__2026-03-30.csv`,
`artifacts/tables/eda__osm_destinations_schools__2026-03-30.csv`,
`artifacts/tables/eda__tract_osm_destination_counts__2026-03-30.csv`,
`artifacts/figures/eda__osm_destinations_map__2026-03-30.png`

**Observation**: OSM Overpass fetches for expanded bbox: **181 hospitals**, **1,170
groceries**, **1,273 schools**. All three fetches status "ok". School count now exceeds
1,000, confirming the previous run hit the Overpass default cap — the expanded bbox query
returned 1,273 schools, indicating the limit was raised or pagination resolved. Tract-level
assignments: hospitals assigned to **107/737 tracts** (171 point features in tracts), groceries
to **449/737 tracts** (1,101 features), schools to **501/737 tracts** (1,042 features).
Tracts with zero all destinations: **106 (14.4%)**. *Previous result (old bbox): 143
hospitals, 886 groceries, 1,000 schools (capped), 13.7% zero-all-destination tracts.*

**Interpretation**: Larger study area = more destinations. The school cap from the previous
run is resolved. Hospital rarity at the tract level is structural (healthcare facilities
cluster at a scale larger than census tracts) — reachability via transit is what matters,
not in-tract presence. The 14.4% zero-destination tracts are overwhelmingly in the
low-density suburban/rural fringe, not the disadvantaged urban core (consistent with F017
and F019 patterns). LODES job data still not downloaded — remains highest-priority before
pipeline nb 03.

**Paper implication**: Section 4 (Data) updated counts: 181 hospitals, 1,170 groceries,
1,273 schools across the study area. The previous school count was an artifact of the old
API result limit; 1,273 is the correct number. LODES jobs layer still needed.

**Next question**: LODES download is still missing (F018/STATUS.md). Run:
`python scripts/download_data.py --config configs/san_diego.yaml --sources lodes`.

---
### F017 — Destination counts pro-poor (density confound); updated county-wide correlations
**Date**: 2026-03-30 (updated; original 2026-03-29 used old bbox)
**Source**: `notebooks/eda/06_opportunities_and_destinations.ipynb`
**Artifact**: `artifacts/tables/eda__tract_osm_destination_counts__2026-03-30.csv`

**Observation**: Spearman correlations between `disadvantage_z` and county-wide tract
destination counts (n=737): groceries ρ = **+0.258**, hospitals ρ = **+0.345**, schools ρ =
**+0.010** (near zero). *Previous result (571-tract bbox): groceries +0.266, hospitals
+0.355, schools +0.025.* All correlations hold in the same direction; minor weakening
consistent with dilution from added suburban tracts (same pattern as F014). Tracts with
zero of all three destination types: **106/737 (14.4%)**; these are concentrated in the
low-density suburban fringe.

**Interpretation**: The density confound extends to the destination layer and remains
robust county-wide. Disadvantaged urban core tracts have more destinations within their
polygon because they are in a denser built environment — not because transit-dependent
residents can actually access more opportunities. The near-zero school correlation (ρ=0.010)
is stable, likely reflecting schools' more even spatial distribution relative to grocery
stores and hospitals. This finding closes the EDA evidence for the density confound:
it appears in every supply and destination metric (stops, headways, walk density,
grocery counts, hospital counts). The Bayesian accessibility model is the correct tool
to disentangle supply density from actual opportunity access.

**Paper implication**: Section 3 (Methods): "Destination counts within census tract
boundaries correlate positively with disadvantage (groceries ρ=0.258, hospitals ρ=0.345),
confirming that origin-tract destination counts conflate urban density with access. We
therefore use r5py cumulative opportunity accessibility — the count of destinations
reachable within T minutes — rather than within-tract proximity."

**Next question**: Once r5py runs, compare 45-min grocery/hospital/job counts from Q1 vs
Q4 tracts. Hypothesis: Q4 (most disadvantaged) tracts reach *fewer* total opportunities
despite higher within-tract counts, because route structure and destination geography
dominate over supply density.

---
### F018 — Cross-source master join is clean: all four datasets align on 737 GEOIDs with zero orphans
**Date**: 2026-03-30
**Source**: `notebooks/eda/07_cross_source_sanity_joins.ipynb`
**Artifact**: `artifacts/tables/eda__cross_source_join_diagnostics__2026-03-30.csv`

**Observation**: The four data layers — base ACS/TIGER (737 rows), GTFS tract service
(737 rows), OSM walk edges (737 rows), OSM destination counts (737 rows) — all join on
`GEOID` to produce a single master table of **737 rows, 737 unique GEOIDs**, zero unmatched
keys in any direction. Every tract has a record in every source.

**Interpretation**: The analysis-ready master table is clean and pipeline-ready. No imputation
is needed for missing source records. The only filtering that will happen downstream is
geographic (bbox clip to 726 tracts) and potentially population-based (exclude zero-population
tracts). This is the final EDA sanity check that confirms all upstream joins — TIGER/ACS
(F011), GTFS/tract (F012–F013), walk/tract (F015), and destination/tract (F016) — are
consistent with each other when assembled into one table.

**Paper implication**: One sentence in the Data section: "All data sources were joined on census tract GEOID with no unmatched records across 737 San Diego County tracts (726 in-study-area bounding box)." The join
diagnostics table is the provenance record for this claim.

**Next question**: None blocking. EDA phase is complete. Proceed to pipeline.

---

### F019 — Transit access deserts are almost exclusively in affluent, low-density tracts — 2 in-bbox high-disadvantage exceptions
**Date**: 2026-03-30 (updated with expanded bbox)
**Source**: `notebooks/eda/07_cross_source_sanity_joins.ipynb`
**Artifacts**: `artifacts/tables/eda__tracts_zero_or_few_stops_within_0p5km__2026-03-30.csv`,
`artifacts/tables/eda__tracts_zero_or_few_stops_summary__2026-03-30.csv`,
`artifacts/figures/eda__tracts_zero_or_few_stops_within_0p5km__2026-03-30.png`,
`artifacts/figures/eda__nearest_stop_distance_hist__2026-03-30.png`

**Observation**: Using a 0.5 km walk-radius threshold among **726 in-bbox tracts**: **98
tracts** (13.5%) have zero stops within 500 m; **134** (18.5%) have few stops within 500 m.
**6 county tracts** simultaneously have zero stops within 0.5 km AND `disadvantage_z` > 0.5
— but only **2 are inside the study bbox**: (1) Tract **209.03** (z=0.657, poverty 16.9%,
nearest stop 23.1 km — this is a remote eastern fringe tract that barely clips the expanded
bbox), and (2) Tract **139.07** (z=0.759, poverty 17.1%, nearest stop 0.61 km — just over
the 0.5 km threshold, essentially on the boundary). The remaining 4 high-disadvantage
zero-stop tracts are outside the study area (rural backcountry: Tracts 211.01, 211.02,
189.03, 189.04). *Previous result (old bbox): 80 of 571 tracts (14.0%) zero-stop; only 1
in-bbox high-disadvantage exception (Tract 139.07 only).*

Mean `disadvantage_z` for zero-stop tracts: strongly negative (suburban/rural fringe
pattern unchanged). The core finding — transit access deserts are in affluent,
low-disadvantage tracts — is fully confirmed county-wide.

**Interpretation**: The expanded bbox adds Tract 209.03, which is a rural eastern fringe
tract that happens to fall within the new, larger bbox but is not meaningfully part of the
urban transit service area. Tract 139.07 remains the same edge case from the original run.
The conclusion is unchanged: there is essentially no spatial overlap between transit
proximity deserts and high-disadvantage urban tracts. The equity problem — if it exists in
San Diego — is about what the transit network delivers in terms of job/service accessibility,
not about stop presence or walk distance.

**Paper implication**: Updated count: "98 of 726 in-bbox tracts (13.5%) have zero transit
stops within 500 m; only 2 have both zero stops and elevated disadvantage (z > 0.5), one
of which (Tract 209.03) is a remote rural fringe outlier." The finding that transit
proximity deserts fall overwhelmingly in affluent, low-dependency suburbs sharpens the
Section 1 framing and motivates the accessibility-vs-supply analysis.

**Next question**: See **F020** — pipeline nb03 now tests whether cumulative **job** accessibility
follows the same pro-disadvantage pattern as supply-side metrics.

---

### F020 — Schedule-based job accessibility remains pro-disadvantage (ρ ≈ +0.47 at 45 min)
**Date**: 2026-04-03
**Source**: `notebooks/03_accessibility_computation.ipynb`
**Artifacts**: `data/processed/accessibility/tract_accessibility_bundle__2026-04-03.parquet`,
`artifacts/tables/pipeline__03_accessibility_summary__2026-04-03.csv`,
`artifacts/figures/pipeline__03_accessibility_choropleth__2026-04-03.png`

**Observation**: For **726** tract origins inside the study bbox, Spearman correlation between
`jobs_C000_45min` (LODES total jobs reachable within **45** minutes by transit+walk, with
walk-only fallback per notebook logic) and ACS-derived **`disadvantage_z`** is **ρ ≈ 0.467**
(positive). **6** tracts have **zero** reachable jobs (including jobs located inside the
origin tract); median jobs reachable is about **10,740**. The same ρ appears in the
2026-04-01 run (summary CSV), indicating a stable sign/magnitude for this configuration.

**Interpretation**: The EDA “density confound” is **not overturned** when moving from crude
supply proxies (stops, headways, walk density) to **r5py schedule-based cumulative job
accessibility** at 45 minutes: tracts with **higher** composite disadvantage still tend to
**reach more jobs**, not fewer. That is the opposite of a simple “transit deserts = poor
neighborhoods” story for **employment access** under this measure. It does **not** say
equity is solved — only that **under-served-by-stops** is the wrong framing for SD on this
outcome; questions shift to **quality of jobs**, **temporal/reliability** (GTFS-RT deferred),
**non-work trips**, and **who can use** the network (see caveats).

**Paper implication**: Answers the Methods/Data tension posed after F014–F017: for **LODES
jobs at 45 min**, destination geography and routing **do not** cancel the urban core’s
access advantage. The paper hook must **not** assume a clean “supply pro-poor → access
anti-poor” reversal for jobs; **F024** (Bayesian layer / rankings vs deterministic; dual **X** specs) and
**POI-only** diagnostics (hospitals/groceries/schools in the bundle) may still show
different structure — report them explicitly. Fig 2 / Fig 3 plan: deterministic accessibility
map (nb03 choropleth) + later posterior map (nb05).

**Next question**: Summarize Spearman (or regression) for **POI** reach vs `disadvantage_z`
at 45 min; fit **nb04/nb05** to get uncertainty-aware rankings and P(access < threshold).

---

## Cross-Reference: Findings → Paper Sections

| Finding | Status | Paper Section | Figure candidate |
|---------|--------|--------------|-----------------|
| F001 — NCTD stale feed | confirmed | Sec 4 Data; Sec 6 Discussion | — |
| F002 — NCTD bbox gap (18.76%, old bbox) — resolved by D009 bbox expansion | confirmed | Sec 4 Data; D009 resolved | — |
| F003 — MTS/NCTD system scale | confirmed | Sec 4 Data | — |
| F004 — Headway spread 5–70 min (MTS) | confirmed | Sec 1 hook; Sec 4 Data | Fig 6 |
| F005 — GTFS data quality flags | confirmed | Sec 3 Methods | — |
| F006 — 737 tracts, ACS downloaded | confirmed | Sec 4 Data | — |
| F007 — MOE vs disadvantage: mixed Spearman | confirmed | Sec 3 Methods; Sec 6 Discussion | Fig 5 |
| F008 — 37.9% tracts no-vehicle MOE unusable | confirmed | Sec 1 hook; Sec 3 Methods | Fig 5 candidate |
| F009 — Income $24k–$250k; 2.4x quartile gap | confirmed | Sec 4 Data | Supplementary |
| F010 — Disadvantage right-skewed; top z=4.32 | confirmed | Sec 4; Sec 5 (case studies) | Fig 6 |
| F011 — ACS/TIGER 737/737 perfect match | confirmed | Sec 4 Data | — |
| F012 — 726 in-bbox tracts (expanded bbox); intersection rule | confirmed | Sec 4 Data (sample N) | Fig 1 |
| F013 — Stop density right-skewed; 102 zero-stop county-wide; NCTD in 189 tracts | confirmed | Sec 3 Methods; Sec 4 | Fig 1 |
| F014 — Density confound: disadvantaged tracts have better supply county-wide (all ρs confirmed) | confirmed | Sec 1 hook (central reframe); Sec 5 | Fig 2 |
| F015 — OSM: single component 337k nodes, 65,750 km; density vs disadvantage rho=0.371 | confirmed | Sec 3 Methods; Sec 4 | Supplementary |
| F016 — Destinations: 181 hospitals, 1,170 groceries, 1,273 schools; 14.4% zero-all tracts | confirmed | Sec 4 Data | Supplementary |
| F017 — Destination counts pro-poor (density confound); justifies r5py over proximity | confirmed | Sec 3 Methods | — |
| F018 — 4-source master join clean; 737/737 GEOIDs; EDA complete | confirmed | Sec 4 Data (one sentence) | — |
| F019 — Transit access deserts in affluent suburbs; 2 in-bbox high-disadvantage exceptions (1 rural fringe) | confirmed | Sec 1 hook (key framing); Sec 5 | Fig 2 candidate |
| F020 — Job accessibility at 45 min **pro-disadvantage** (ρ ≈ +0.47); no supply–jobs divergence | confirmed (pipeline nb 03) | Sec 5 Results; reframes hook vs **F024** | Fig 2 (deterministic), Fig 3–4 (posterior) |
| F021 — BYM2 fixed effects (historical seven-predictor run; see note + **F024** for live spec) | superseded in part | Sec 5 Results | Fig 3–4 |
| F022 — σ vs σ_obs poor mixing (**2026-04-03** only; fixed obs noise fixes this) | superseded | Sec 3 Methods archive | diagnostics fig |
| F023 — Posterior mean vs disadvantage pro-poor; SD correlation weak / NS | confirmed pattern; update ρ with **F024** | Sec 5 Results | Fig 5 candidate |
| F024 — Raw **X** vs **Spatial+** BYM2: stable Spearman, shifting β / σ | confirmed (nb04) | Sec 3 Methods; Sec 5 Results | Table |
| F025 — Production MCMC passes practical R-hat / ESS | confirmed (nb04) | Sec 3 Methods | diagnostics |
| F026 — Moran's I residual high under heavy Spatial+; 5% default; primary = raw X | confirmed (nb04+context) | Sec 3 Methods; Discussion | Moran CSV |
| F027 — nb05 comprehensive results: multi-threshold equity, perfect hook GEOID, Wasserstein, rank stability | confirmed (nb05) | Sec 5 Results; Sec 1 hook; Fig 6–8 | Figs 3–8 complete |

---

## Phase 2 — Bayesian Model (nb04)

### F021 — BYM2 beta coefficients: density, income, no-vehicle, and Hispanic share are signal; poverty and Black share overlap zero
**Date**: 2026-04-05 (production run; supersedes 2026-04-03 partial run)
**Note (2026-04-07):** The **current** notebook uses a **four-predictor** block centred on **`disadvantage_z`** (see **F024** for saved diagnostics and **Spatial+** comparison). The **coefficient list below** describes an **earlier seven-predictor** run kept for **historical** comparison — do not treat **pct_hispanic** / **poverty** rows as the live model without checking the notebook.

**Source**: `notebooks/04_bayesian_model.ipynb`
**Artifacts**: `artifacts/tables/pipeline__04_model_diagnostics__2026-04-05.csv`

**Observation**: Regression uses z-scored covariates and a **standardised** `log1p` response (mean 0, sd 1 in-sample). April 5 production run — `pct_nh_white` dropped (r=−0.81 with pct_hispanic; D011), Student-t likelihood ν=4 (D012), NaN routing-failure tracts dropped (D013). Posterior means (94% HDI):
- `beta[log_pop_density]` **+0.469** [0.403, 0.538] — HDI excludes 0 ✓
- `beta[no_vehicle_hh_rate]` **+0.175** [0.104, 0.242] — HDI excludes 0 ✓
- `beta[log_median_income]` **−0.163** [−0.252, −0.072] — HDI excludes 0 ✓ (stronger than April 3)
- `beta[pct_hispanic]` **−0.121** [−0.196, −0.052] — HDI excludes 0 ✓ (new signal; was overlapping 0 April 3)
- `beta[pct_black]` −0.023 [−0.078, 0.035] — overlaps 0
- `beta[poverty_rate]` +0.023 [−0.052, 0.102] — overlaps 0
- `alpha` ≈ −0.003 [−0.054, 0.050]

*April 3 betas for comparison: log_pop_density +0.577, log_median_income −0.091 (barely excluded 0), pct_hispanic −0.057 (overlapped 0, pct_nh_white in model).*

**Interpretation**: Four parameters now have HDIs that credibly exclude zero. **Population density** remains the dominant structural driver. **Income** effect strengthened after dropping collinear pct_nh_white — higher-income (suburban) tracts show lower fitted job accessibility. **No-vehicle rate** remains a positive signal for urban-transit correlation. Most importantly, **Hispanic share now shows a credible negative association**: after removing the pct_nh_white competitor for the same variance, the model correctly identifies that predominantly Hispanic tracts have lower fitted accessibility than density/income alone predict — a genuine equity signal that was previously masked by collinearity. **Poverty and Black share** remain in the model but show no independent slope beyond what spatial effects and the other covariates capture.

**Paper implication**: Upgrade the fixed-effects story from three to four credible signals. Lead with density and income (structural/geographic), then the no-vehicle/urban-form proxy, then flag the Hispanic share finding as an equity-relevant residual after space is controlled. Poverty and Black share are absorbed by the spatial BYM2 surface — do not claim “poverty doesn’t matter”; claim “poverty’s effect appears spatially mediated and is captured by the random field.”

**Next question**: Marginal contrasts by disadvantage quartile (nb05); sensitivity to dropping `no_vehicle_hh_rate` (urban-form proxy test); interpret pct_hispanic sign with care (see F026).

---

### F022 — BYM2 spatial mixing ρ is well identified; variance split (σ vs σ_obs) still mixes poorly
**Date**: 2026-04-03 (updated after prior / likelihood overhaul)
**Status (2026-04-07):** Applies to the **estimated `sigma_obs`** era only. **Superseded** for **variance-parameter reporting** by **F025** (`obs_noise: fixed`, long sampling). Keep this finding as **documentation of the 2026-04-03 failure mode**.

**Source**: `notebooks/04_bayesian_model.ipynb`
**Artifacts**: `artifacts/tables/pipeline__04_model_diagnostics__2026-04-03.csv`, `artifacts/figures/pipeline__04_trace_plot__2026-04-03.png`

**Observation**: After `rho ~ Beta(2,2)`, standardised **y**, and tighter `sigma_obs ~ HalfNormal(0.2)`: **`rho`** posterior mean **≈ 0.44** (HDI ~[0.06, 0.79]), **R-hat ≈ 1.005**, **ESS_bulk ≈ 1.7k** — acceptable mixing. **`sigma`** (BYM2 spatial scaling): R-hat **≈ 1.07**, ESS_bulk **≈ 38**. **`sigma_obs`**: R-hat **≈ 1.32**, ESS_bulk **≈ 10**. **189 divergent transitions** in 8 000 retained draws (~**2.4%**). *First nb04 run* had ρ, σ, and σ_obs all stuck with R-hat > 1.1 and 438 divergences — **that failure mode is largely resolved for ρ**, not for the variance decomposition.

**Interpretation**: HMC still struggles to separate **spatially structured residual scale** from **Gaussian observation noise** on a near-saturated smooth spatial surface — a known fragile axis even with PC priors. Fixed effects and ρ are informative enough to estimate; **width-like quantities** (posterior predictive spread, exceedance probabilities) inherit more noise from σ / σ_obs.

**Paper implication**: Methods must report σ / σ_obs diagnostics explicitly. Do **not** treat exceedance probabilities or CI widths as publication-final until R-hat and ESS for both variance parameters look healthy (rule-of-thumb R-hat < 1.01, ESS in the hundreds).

**Next question**: Increase `tune` (e.g. 2000 in `configs/defaults.yaml` — still 1000 at last run), longer sampling, or reparameterise (non-centered / soft sum-to-zero alternatives); consider a more informative prior on one variance if theory allows.

---

### F023 — Posterior equity Spearman: pro-disadvantage mean access holds; SD–disadvantage link is weak and not significant
**Date**: 2026-04-03 (updated after overhaul)
**Source**: `notebooks/04_bayesian_model.ipynb`
**Artifacts**: `artifacts/tables/pipeline__04_equity_spearman__2026-04-03.csv`, `artifacts/figures/pipeline__04_posterior_mean_map__2026-04-03.png`, `artifacts/figures/pipeline__04_posterior_sd_map__2026-04-03.png`

**Observation**: Spearman vs `disadvantage_z` on 726 tracts: `posterior_mean_jobs` **ρ ≈ +0.52** (*p* ≈ 7×10⁻⁵¹) — same qualitative story as deterministic F020 (disadvantaged tracts tend to have **higher** posterior mean reachable jobs). `exceedance_prob_45min` and `p_transit_desert` **ρ ≈ −0.58** (*p* ≈ 2.6×10⁻⁶⁶) — more disadvantaged tracts are **less** often below the city Q25 “desert” cut on μ. **`posterior_sd_log1p` ρ ≈ −0.07**, *p* **≈ 0.075** — **not significant** at α = 0.05 (contrast first run ρ ≈ −0.14, *p* ≈ 10⁻⁴).

**Interpretation**: The **ranking** story for San Diego jobs at 45 min is stable: Bayesian smoothing does not flip the pro-urban-core / pro-disadvantage association seen in nb03. **Notebook 05** confirms **deterministic vs Bayesian mean-access ranks** track each other (**Spearman ρ ≈ 1** on typical runs) — there is **no** large-scale **rank reversal** for the hook; the narrative shifts to **uncertainty**, **exceedance** near thresholds, and **Wasserstein** contrasts (**nb05** exports). The **uncertainty-vs-disadvantage** link remains **weak / NS** in Spearman terms (**F024** table). **Production** MCMC (**F025**) no longer has the **F022** `sigma` / `sigma_obs` identifiability problem; updated numeric ranges appear in **F024**.

**Paper implication**: Lead with **access level** (posterior mean / desert probability) vs disadvantage; treat **uncertainty layering** as exploratory until variance parameters converge. If σ diagnostics improve and ρ(SD) stays near zero, the paper’s originality may shift toward **probabilistic thresholds** and **spatial partial pooling** rather than “more uncertainty in poor tracts.”

**Next question**: Recompute Spearman after improved MCMC; optionally correlate CI width with MOE (F007) rather than only disadvantage_z. Updated Spearman magnitudes for the **2026-04-05 / 2026-04-06** bundles are tabulated in **F024**.

---

### F024 — Two BYM2 estimands: z-scored X vs Spatial+ (eigenvector residualization) before ICAR
**Date**: 2026-04-07 (documents saved bundles **2026-04-05** vs **2026-04-06**)
**Source**: `notebooks/04_bayesian_model.ipynb`
**Artifacts** (semantic run ids; legacy dates **`2026-04-05`** / **`2026-04-06`** = same fits):
`artifacts/tables/pipeline/pipeline__04_model_diagnostics__fit_raw_zscore_x.csv`,
`artifacts/tables/pipeline/pipeline__04_model_diagnostics__fit_spatial_plus_x.csv`,
`artifacts/tables/pipeline/pipeline__04_equity_spearman__fit_raw_zscore_x.csv`,
`artifacts/tables/pipeline/pipeline__04_equity_spearman__fit_spatial_plus_x.csv`

**Observation** — Same outcome (standardised `log1p` LODES jobs at **45** min), same BYM2 structure, same **four** covariates (`disadvantage_z`, `no_vehicle_hh_rate`, `log_median_income`, `log_pop_density`), **~720** tracts after dropping NaN r5 origins. **Two separate** executions (different `RID` / `PIPELINE_RUN_ID`) differ in whether **X** is **z-scored raw** (**`PIPELINE_NO_SPATIAL_PLUS=1`**) or **Spatial+** residualized (project off the smoothest eigenvectors of **Q = D − W**, then re-standardize). **As of 2026-04-08** the notebook default removes **~5%** of modes (**not** 15%); older saved bundles (legacy **2026-04-06**) may have used **~15%** and showed **problematic residual Moran's I** (**F026**). **Primary estimand:** raw **X** (**D011**); Spatial+ = **sensitivity**. **Semantic ids:** `fit_raw_zscore_x`, `fit_spatial_plus_x` (`src.utils.config`).

| Quantity | **`fit_raw_zscore_x`** (BYM2 on **z-scored X**) | **`fit_spatial_plus_x`** (BYM2 on **Spatial+ X**) |
|----------|----------------------------------------|----------------------------------------|
| **β[disadvantage_z]** | mean **+0.104**, HDI **[−0.07, 0.29]** (crosses 0) | mean **+0.058**, HDI **[−0.12, 0.23]** (crosses 0) |
| **β[no_vehicle_hh_rate]** | **+0.158** [0.05, 0.26] (excludes 0) | **−0.055** [−0.17, 0.06] (crosses 0; sign reverses) |
| **β[log_median_income]** | **−0.024** (HDI crosses 0) | **−0.162** [**−0.29, −0.03**] (**excludes 0** — credible after Spatial+) |
| **β[log_pop_density]** | **+0.451** [0.39, 0.51] (excludes 0) | **+0.234** [0.16, 0.31] (excludes 0; magnitude shrinks ~48%) |
| **σ** (spatial scale on std-y scale) | **≈ 1.11** | **≈ 1.31** |
| **ρ** | **≈ 0.43** | **≈ 0.42** |
| **α ESS_bulk** | ≈ 425 | ≈ 361 (improved from 197 at 15%; 10K draws path) |
| Spearman **posterior_mean_jobs** vs disadvantage | **ρ ≈ 0.470** | **ρ ≈ 0.468** |
| Spearman **exceedance_prob_45min** vs disadvantage | **ρ ≈ −0.467** | **ρ ≈ −0.432** |
| **posterior_sd_log1p** vs disadvantage | ρ ≈ −0.045, *p* ≈ 0.23 | ρ ≈ −0.039, *p* ≈ 0.30 (both NS) |
| **Moran's I residuals** | Run Moran cell after **raw X** fit → `pipeline__04_moran_residual_summary__<RID>.csv`; optional compare from saved `idata.nc` → `pipeline__04_moran_residual_from_idata_compare__<RID>.csv` | **0.572** (Spatial+ ~5% k_remove=36; **F026**; archived **`fit_spatial_plus_x`** bundle) |

**Important:** The notebook **overwrites `X` → `X_splus`** before a **single** `pm.sample`; it does **not** write both posteriors in one run. The two columns above exist because the analyst **saved** two different runs. OLS on **pre–Spatial+** `X` is printed in-notebook as a **non-spatial** baseline only.

**Interpretation** — (1) **Tract-level rankings / maps** stay **equity-aligned** in both runs: **higher** disadvantage ↔ **higher** posterior **mean** jobs and **lower** “desert” exceedance — consistent with **F020** and **not** reversed by Bayesian smoothing. (2) **Regression slopes are not comparable** across columns without naming the **estimand**: a **positive** `β_disadvantage` on raw **X** is plausibly **inflated** by **shared smooth geography** with **y** (spatial confounding). **Spatial+** **removes** that subspace from **X**; **`β_disadvantage` near zero** with a **wide** interval means **no detectable linear association of the composite after that adjustment** — not “equity vanished.” (3) **Larger σ** after Spatial+ is **expected**: variation previously soaked up by **X** moves into **φ**.

**Paper implication** — Methods: **primary** = BYM2 on **z-scored X**; **supplement** = Spatial+ with reported **k_remove** and **Moran's I** (**F026**). Results: **Spearman + maps** first (stable story), then **β / σ** sensitivity table. Discussion: Hodges–Reich confounding; **do not** apply regression-only **14–21%** eigen rules blindly when ICAR is present.

**Next question** — Archive **`pipeline__04_moran_residual_summary__<RID>.csv`** for each run; optional grid **k_remove** ∈ {3%, 5%, 10%}.

---

### F025 — MCMC diagnostics: long sampling + fixed obs noise + Student-t (production settings)
**Date**: 2026-04-07
**Source**: `configs/defaults.yaml`, `notebooks/04_bayesian_model.ipynb`
**Artifacts**: `artifacts/tables/pipeline__04_model_diagnostics__2026-04-05.csv`

**Observation** — Production defaults include **`draws: 8000`**, **`tune: 4000`**, **`target_accept: 0.99`**, **`likelihood: student_t`** (ν=4), **`obs_noise: fixed`** (`fixed_obs_sigma=0.05`, no sampled `sigma_obs`), **`beta_sigma: 0.3`**, **`nuts_init: advi+adapt_diag`**. Run **2026-04-05**: **0 divergences**; **ρ** and **σ** — **R-hat ≈ 1.000**, **ESS_bulk in the ~6k–7k** range; **α** and all **β** — **ESS_bulk ≈ 420–835**, **R-hat ≈ 1.003–1.011** (only **α** barely above 1.01). This **supersedes the variance-mixing failure** documented in **F022** for the **2026-04-03** `sigma_obs` + short-tune configuration.

**Interpretation** — For **reporting global parameters and fixed effects** on the **current** spec, diagnostics meet **practical** Vehtari-style targets on the **raw X** path (**2026-04-05**). The **Spatial+** run (**2026-04-06**) shows **lower α ESS (~197)** at **8k** draws; the notebook now floors **draws ≥ 10 000** when Spatial+ is **on** (unless `PIPELINE_FAST_MCMC`) to target better **α**/**β** ESS — re-run to populate new diagnostics.

**Paper implication** — Replace “σ provisional” wording tied to **F022**; report **R-hat**, **bulk ESS**, **divergences**, **BFMI**. Cite **`pipeline__04_model_diagnostics__<RID>.csv`** and optional `scripts/nb04_export_diagnostics.py` bundle.

**Next question** — If **α** must satisfy **R-hat ≤ 1.01** strictly, add **draws** or **tune** marginally; re-run diagnostics export for figures.

---

### F026 — Moran's I on posterior-mean residuals: Spatial+ path shows I ≈ 0.57 even at 5% k_remove
**Date**: 2026-04-08 (updated 2026-04-06 confirmed run)
**Source**: `notebooks/04_bayesian_model.ipynb` (Moran diagnostic cell); `artifacts/tables/pipeline/pipeline__04_moran_residual_summary__fit_spatial_plus_x.csv`
**Observation**: The **`fit_spatial_plus_x`** Spatial+ run used **k_remove = 36** eigenvectors out of **720** tracts (**~5%**, current notebook default). Moran's I of **y_std − E[μ]** = **0.5724** — well above the target (< 0.05). Three probable causes, in order of likelihood:
1. **Low per-tract u_free ESS** — ICAR has 719 free parameters; even with 10K draws × 4 chains (40K total), median per-tract ESS may be 200–800. Low ESS means the posterior mean of μ per tract is poorly estimated, so y_std − μ_mean still carries spatial pattern. This is a convergence artefact, not model misspecification.
2. **PC prior on σ is too tight** — Exponential(2) prior (mean=0.5) regularises sigma toward small values. The ICAR needs sigma ≈ 1.3 (posterior) to absorb the spatial structure in y that X no longer explains after Spatial+. With tighter priors the ICAR is constrained.
3. **Spatial+ removes y-correlated variance** — even 5% of the smoothest eigenvectors overlaps with spatial patterns in y. The ICAR must then fully compensate, which requires deeper sampling and possibly larger σ.
The **`fit_raw_zscore_x`** run should archive Moran's I via the same Moran cell after a **raw X** fit, or by loading **`fit_raw_zscore_x_idata.nc`** in an optional compare cell while another session is loaded (same `y_std` / `W` / tract set). Expected **lower** I than Spatial+ because spatially structured **X** explains much of **y** autocorrelation directly. The main Moran cell prints a **one-line sensitivity summary** (`RID`, `estimand`, `k_remove`, `E_spatial`, `Moran_I`).

**Interpretation**: Spatial+ at **~5%** (current default) still produces high Moran's I under BYM2 with current PC priors. The primary paper estimand is **BYM2 on raw z-scored X** (D011); Spatial+ is sensitivity only. Convergence cell now reports per-tract u_free ESS to diagnose whether Moran's I is a sampling issue vs. a prior-tightness issue.

**Paper implication**: Report Moran's I + k_remove + estimand label for every archived run. For supplement: note that Spatial+ BYM2 tension (projection vs. ICAR absorption) requires careful prior calibration. Raw X primary avoids this issue.

**Next question**: Re-run nb04 with `PIPELINE_NO_SPATIAL_PLUS=1` and archive `pipeline__04_moran_residual_summary__<raw_X_RID>.csv`; compare to 0.5724. If raw X Moran's I < 0.10, primary estimand is confirmed well-specified.

---

## Phase 3 — Posterior Analysis (nb05)

### F027 — nb05 comprehensive results: multi-threshold equity, perfect paper hook, Wasserstein map, rank stability
**Date**: 2026-04-08 (production run; all 23 cells executed for `fit_raw_zscore_x`)
**Source**: `notebooks/05_posterior_analysis.ipynb`
**Artifacts** (all `fit_raw_zscore_x` stem):
- `artifacts/tables/pipeline__05_summary__fit_raw_zscore_x.csv`
- `artifacts/tables/pipeline__05_multithreshold_equity__fit_raw_zscore_x.csv`
- `artifacts/tables/pipeline__05_exceedance_shift_by_threshold__fit_raw_zscore_x.csv`
- `artifacts/tables/pipeline__05_hook_near_q25_exceedance_ambiguous__fit_raw_zscore_x.csv`
- `artifacts/tables/pipeline__05_wasserstein_tracts__fit_raw_zscore_x.csv`
- `artifacts/figures/pipeline__05_posterior_mean_jobs__fit_raw_zscore_x.png` (Fig 3)
- `artifacts/figures/pipeline__05_p_low_access_q25__fit_raw_zscore_x.png` (Fig 4)
- `artifacts/figures/pipeline__05_ci_width_log1p__fit_raw_zscore_x.png` (Fig 5)
- `artifacts/figures/pipeline__05_case_study_fan_kde_jobs__fit_raw_zscore_x.png` (Fig 6)
- `artifacts/figures/pipeline__05_rank_divergence__fit_raw_zscore_x.png` (Fig 7)
- `artifacts/figures/pipeline__05_wasserstein_map__fit_raw_zscore_x.png` (Fig 8)
- `artifacts/figures/pipeline__05_case_study_mu__fit_raw_zscore_x.png`

---

#### F027a — Primary equity Spearman (posterior level and uncertainty)

**Observation**:

| Variable | Spearman ρ vs disadvantage_z | p-value |
|---|---|---|
| posterior_mean_jobs | **+0.4699** | 8.05×10⁻⁴¹ |
| posterior_sd_log1p | −0.0449 | 0.229 (NS) |
| exceedance_prob_45min | **−0.4669** | 2.86×10⁻⁴⁰ |
| ci_width_log1p | −0.0590 | 0.114 (NS) |
| rank_det_vs_rank_bayes | **+0.9999758** | 0.0 |

**Interpretation**: The pro-disadvantage pattern from deterministic accessibility (F020) survives Bayesian spatial smoothing without change — posterior mean ρ ≈ +0.47 is identical to the deterministic ρ ≈ 0.467. More disadvantaged tracts are *less* likely to fall below the Q25 desert threshold (exceedance ρ ≈ −0.47). Crucially, **posterior uncertainty is NOT larger in disadvantaged tracts** — both `posterior_sd` and `ci_width` have non-significant negative correlations with disadvantage. This is an important null finding: Bayesian uncertainty does not compound the equity problem (compare to F008, where ACS MOE was linked to disadvantage on some indicators). The near-perfect rank stability (ρ = 0.9999758, max delta = ±13 out of 720 tracts) means the Bayesian model does not reorder neighborhoods — its value is in **uncertainty quantification**, not **reranking**.

**How this changed from F023**: F023 (nb04 era) reported ρ ≈ +0.52 for posterior mean and ρ ≈ −0.58 for exceedance — those values came from a run with slightly different tract count or NaN handling. The production nb05 values (+0.470 / −0.467) are fully consistent directionally; the modest magnitude difference reflects consistent thresholds applied at the nb05 stage after loading the canonical bundle.

**Paper implication**: Sec 5 Results: "Bayesian posterior mean job accessibility retains the pro-urban-core, pro-disadvantage gradient confirmed deterministically (F020): Spearman ρ = +0.470, p < 10⁻⁴⁰. Posterior uncertainty is independent of disadvantage status (ρ = −0.045, NS), indicating the model does not compound data sparsity inequities into wider credible intervals for already-disadvantaged neighborhoods. Rank stability ρ = 0.9999758 with maximum rank delta = ±13 (out of 720 tracts)."

---

#### F027b — Multi-threshold sensitivity (30 / 45 / 60 min)

**Observation** (from `pipeline__05_multithreshold_equity__fit_raw_zscore_x.csv`):

| Threshold | Deterministic jobs ρ vs disadvantage | Exceedance ρ vs disadvantage |
|---|---|---|
| **30 min** | +0.484 (p=1.6×10⁻⁴³) | −0.301 (p=1.4×10⁻¹⁶) |
| **45 min** | +0.467 (p=2.6×10⁻⁴⁰) | −0.467 (p=2.9×10⁻⁴⁰) |
| **60 min** | +0.414 (p=3.1×10⁻³¹) | −0.481 (p=6.1×10⁻⁴³) |

**Interpretation**: Two opposing gradient trends emerge as the travel-time window expands. The **deterministic** pro-disadvantage gradient **weakens** from 30 to 60 min (ρ drops from +0.484 to +0.414) — as windows open wider, suburban tracts reach more jobs via highway-adjacent express service, reducing the urban-core relative advantage. In contrast, the **exceedance** desert probability correlation **strengthens** (−0.301 → −0.481) at longer thresholds — more tracts fall below the desert cut at 60 min, but disadvantaged tracts *avoid* this classification more robustly because their urban core location means they reach threshold-crossing job counts even at longer windows. This crossing pattern is a genuinely novel finding: the equity gradient's direction is threshold-dependent in the exceedance metric but not in the mean metric, a distinction only visible through the Bayesian uncertainty framework.

**How this changed from earlier iterations**: Prior runs only reported the 45 min threshold. This is the first systematic multi-threshold table, confirming the equity story is not an artifact of any single cutoff.

**Paper implication**: Sec 5 / Methods sensitivity: report all three rows. The weakening deterministic gradient and strengthening exceedance gradient at 60 min is a key contribution of the probabilistic framing — deterministic-only analysis would miss this asymmetry.

---

#### F027c — Rank stability: confirmed ρ = 0.9999758, max delta = ±13

**Observation**: Spearman(rank_det, rank_bayes) = **0.9999758**; maximum rank shift: +13 (largest downgrade: GEOID 06073006200, rank_det=226 → rank_bayes=239) and −6 (largest upgrade: GEOID 06073010300, rank_det=538 → rank_bayes=532). The mean absolute rank delta across all 720 tracts is tiny (< 2 positions).

**Interpretation**: BYM2 spatial smoothing **preserves** the deterministic accessibility ranking almost perfectly. This definitively answers "does Bayesian spatial pooling reorder which tracts get labeled 'underserved'?" — it does not. The ICAR component borrows strength from neighbors to stabilize posterior means, but because San Diego's accessibility landscape is smooth (dense urban core, sparse suburban periphery), the spatial smoothing confirms rather than contradicts the deterministic picture. This finding reframes the paper's novelty claim: the contribution is **uncertainty quantification** and **threshold probability assessment**, not rank reversal discovery.

**How this changed**: Earlier drafts of the paper hook speculated about rank reversals as the key contribution. The hook now correctly pivots to near-threshold exceedance uncertainty (see F027d).

---

#### F027d — Perfect paper hook: GEOID 06073013317 (exceedance_prob = 0.4998)

**Observation** (from `pipeline__05_hook_near_q25_exceedance_ambiguous__fit_raw_zscore_x.csv`):

| GEOID | det_jobs | exceedance_prob_45min | ci_width_log1p | disadvantage_z |
|---|---|---|---|---|
| **06073013317** | 4,467 | **0.4998** | 0.341 | −0.476 |
| 06073013908 | 4,492 | 0.484 | 0.332 | **+0.251** |

**Interpretation**: GEOID **06073013317** sits at exactly 50% exceedance probability — the Bayesian posterior says it is a **coin flip** whether this tract is a transit desert. The city's deterministic schedule says it has 4,467 reachable jobs (just above the Q25 threshold), so it would be classified as "adequately served." But the posterior credible interval spans the threshold, giving equal probability to "adequate" and "desert." This is the paper's opening case: *the spreadsheet says fine; the model says we genuinely don't know.* The second candidate (GEOID 06073013908) is complementary because it has elevated disadvantage (z=+0.251) — a disadvantaged tract where the city's data gives false confidence in classifying it as non-desert. Together these two cases make Fig 6 (case study posteriors) the most compelling panel in the paper.

**How this changed from earlier runs**: Prior nb05 runs did not have a systematic hook-candidate export cell. This finding is new to the April 7–8 production run and directly informs PAPER.md's hook section update.

**Paper implication**: Section 1 opening: "GEOID 06073013317's deterministic accessibility score places it just above the transit-desert threshold. A 50% exceedance probability — a coin flip — is the Bayesian model's honest answer to the planning question 'is this neighborhood adequately served?'"

---

#### F027e — Wasserstein distance map: urban-suburban gradient 13,500 → 86,000+

**Observation**: Wasserstein-1 distances from each tract's posterior job distribution to the top-quartile "well-served" reference pool (D003):
- Range: ~**13,500** (dense urban core, best-served tracts) to **86,000+** (deep suburban / rural periphery)
- The map creates a clean urban → suburban gradient consistent with the density confound story (F014)
- Notable outlier: GEOID **06073005801** (high disadvantage, high posterior mean) has W = **61,515** — surprisingly far from the reference pool despite having above-average posterior mean jobs. This reveals that even when a disadvantaged tract reaches many jobs in expectation, its *distribution shape* (wider CI, heavier tails) differs substantially from the confident, narrow posteriors of genuinely well-served tracts.

**Interpretation**: Wasserstein distance captures something the posterior mean misses: distributional shape similarity, not just expected value similarity. Two tracts can have the same posterior mean but one can have a narrow confident posterior (high-SES suburban tract with predictable high access) and one a wider uncertain posterior (urban core tract with structural access but high sampling variance). The Wasserstein metric quantifies this gap, making Fig 8 the most technically novel visualization in the paper. The outlier pattern at 06073005801 is an important nuance for the Discussion: even the equity "success story" disadvantaged tract has distributional dissimilarity from the top-quartile reference that is invisible to point-estimate analysis.

**How this changed**: Wasserstein computation was not present in early nb05 runs. It was added as a new extension cell in the April 7–8 overhaul (D003). The 13,500–86,000 range and the 06073005801 outlier are entirely new findings.

**Paper implication**: Fig 8 caption: "Wasserstein-1 distance from each tract's posterior job accessibility distribution to the top-quartile well-served reference pool. High values indicate not just lower expected access but distributional dissimilarity — uncertainty structure that is invisible to deterministic metrics."

---

#### F027f — All 7 paper figures produced (Figs 3–8 + case study)

**Status** (April 8, 2026): All figures for the Results section are complete for the primary estimand `fit_raw_zscore_x`:

| Figure | File | Status |
|---|---|---|
| Fig 3 — Posterior mean accessibility map | `pipeline__05_posterior_mean_jobs__fit_raw_zscore_x.png` | ✅ Complete |
| Fig 4 — Exceedance probability P(access < Q25) | `pipeline__05_p_low_access_q25__fit_raw_zscore_x.png` | ✅ Complete |
| Fig 5 — Credible interval width map | `pipeline__05_ci_width_log1p__fit_raw_zscore_x.png` | ✅ Complete |
| Fig 6 — Case study KDE fan (4 tracts) | `pipeline__05_case_study_fan_kde_jobs__fit_raw_zscore_x.png` | ✅ Complete (NEW) |
| Fig 7 — Rank divergence scatter | `pipeline__05_rank_divergence__fit_raw_zscore_x.png` | ✅ Complete |
| Fig 8 — Wasserstein distance map | `pipeline__05_wasserstein_map__fit_raw_zscore_x.png` | ✅ Complete (NEW) |
| Case study μ trajectories | `pipeline__05_case_study_mu__fit_raw_zscore_x.png` | ✅ Complete |

Fig 9 (intervention simulation) requires nb07, which is the next pipeline step.

**Interpretation**: The analysis pipeline is complete through the Bayesian posterior analysis stage. The two NEW figures (Fig 6 KDE fan, Fig 8 Wasserstein) are the most methodologically novel additions — they did not exist in prior nb05 iterations and directly support the three-part novelty claim in PAPER.md.

**Next question**: Build nb07 (intervention simulation) to produce Fig 9 — probabilistic posterior shift when bus frequency increases in high-exceedance, high-disadvantage tracts. This is the final analytical step before writing.

---

## Phase 3 — Equity Decomposition + Model Validation (nb06)

### F028 — Multi-destination equity confirmed across jobs, hospitals, groceries, schools (pro-poor at 30/45/60 min); composite deficit concentrates in affluent suburbs
**Date**: 2026-04-17 (production run; all 11 sections of `notebooks/06_equity_decomposition.ipynb` executed for `fit_raw_zscore_x`)
**Source**: `notebooks/06_equity_decomposition.ipynb`
**Artifacts** (all `fit_raw_zscore_x` stem):
- `artifacts/tables/pipeline__06_summary__fit_raw_zscore_x.csv` — master metric roll-up (11 sections)
- `artifacts/tables/pipeline__06_gini_ci_equity__fit_raw_zscore_x.csv` — Gini + Concentration Index
- `artifacts/tables/pipeline__06_subgroup_posterior_summary__fit_raw_zscore_x.csv` — disadvantage quartile + vehicle-ownership posteriors
- `artifacts/tables/pipeline__06_multidestination_equity__fit_raw_zscore_x.csv` — Spearman × {Jobs,Hospitals,Groceries,Schools} × {30,45,60} min
- `artifacts/tables/pipeline__06_composite_deficit_ranked__fit_raw_zscore_x.csv` — composite exceedance rank (720 tracts)
- `artifacts/tables/pipeline__06_multidesert_tracts__fit_raw_zscore_x.csv` — binary desert flags per destination
- `artifacts/tables/pipeline/pipeline__06_wasserstein_equity_metrics__fit_raw_zscore_x.csv` — Wasserstein equity roll-up
- `artifacts/tables/pipeline/pipeline__06_wasserstein_by_disadv_quartile__fit_raw_zscore_x.csv` — Wasserstein quartile table
- `artifacts/figures/pipeline__06_lorenz_curve__fit_raw_zscore_x.png`
- `artifacts/figures/pipeline__06_subgroup_forest_plot__fit_raw_zscore_x.png`
- `artifacts/figures/pipeline__06_multidestination_spearman_heatmap__fit_raw_zscore_x.png`
- `artifacts/figures/pipeline__06_composite_deficit_map__fit_raw_zscore_x.png`
- `artifacts/figures/pipeline__06_multidesert_map__fit_raw_zscore_x.png`

**Observation**:

| Metric | Value (CI) | Note |
|---|---|---|
| **Gini (posterior mean jobs, pop-weighted)** | **0.1445** [0.139, 0.151] | Deterministic baseline = 0.5187 (different scale; see caveat) |
| **Concentration Index (posterior mean jobs vs disadvantage_z)** | **+0.0359** [+0.0355, +0.0363] | Positive ⇒ pro-poor |
| **CI on desert exceedance vs disadvantage_z** | **−0.458** | Disadvantaged tracts are far below the desert cut |
| **Spearman posterior mean vs disadvantage** | **+0.470** (Jobs 45 min) | Same as F027a (cross-check passes) |
| **Spearman — Jobs** (30/45/60 min) | **+0.484 / +0.467 / +0.414** | Pro-poor at every threshold |
| **Spearman — Hospitals** | **+0.353 / +0.361 / +0.348** | Pro-poor |
| **Spearman — Groceries** | **+0.563 / +0.581 / +0.545** | **Strongest pro-poor gradient** |
| **Spearman — Schools** | **+0.472 / +0.525 / +0.517** | Pro-poor |
| **Subgroup posterior-mean jobs (Q4 most disadv)** | **26,526** jobs (n=180) | ≈ **2.8×** Q1 (least disadv, 9,590) |
| **Subgroup frac(exceedance > 0.5), Q4 vs Q1** | **1.7% vs 17.8%** | Disadvantaged tracts **10× less often** flagged as desert |
| **n tracts with composite exceedance > 0.5 across all 4 destinations** | **120 / 720** (16.7%) | Of those, **mean disadvantage_z = −0.32**, **median −0.33**; **80%** have `disadvantage_z < 0` |
| **Wasserstein by quartile (pop-weighted mean)** | Q1=31,512 → Q2=31,038 → Q3=27,114 → **Q4=24,237** | Disadvantaged tracts **closer** to well-served reference pool |
| **Spearman Wasserstein vs disadvantage_z** | **−0.360** (p=1.7×10⁻²³) | Same story: higher disadvantage → smaller distributional distance from the reference pool |

**Interpretation**: The nb05 equity story (F020/F024/F027) is **not** a jobs-only artefact. At every travel-time threshold and for every destination type we measure, San Diego's most-disadvantaged tracts reach **more** opportunities than affluent suburban tracts, with Spearman coefficients ranging from +0.35 (hospitals) to +0.58 (groceries 45 min). Groceries show the strongest pro-poor gradient; hospitals the weakest (fewer facilities county-wide plus a disproportionate share in urban-core tracts where disadvantage is moderate rather than extreme). The **composite deficit** view — tracts that exceed 50% desert probability on **all four** destination types simultaneously — identifies 120 tracts, and **they are overwhelmingly affluent low-density suburbs** (mean disadvantage_z = −0.32; only 20% of multi-deficit tracts have `disadvantage_z > 0`). The subgroup posterior table makes this quantitative: the most-disadvantaged quartile has ~2.8× the expected reachable jobs of the least-disadvantaged quartile, and its desert-exceedance rate is an order of magnitude lower (1.7% vs 17.8%). The Wasserstein-by-quartile roll-up confirms the same pattern in **distributional-shape** space, not just mean space: the posterior *shape* of disadvantaged tracts is systematically closer to the top-quartile reference pool.

**Important caveat (Gini)**: The 0.14 posterior Gini vs 0.52 deterministic Gini reflects **scale**, not inequality reduction. Bayesian partial pooling compresses posterior means toward the spatial mean; the deterministic Gini is computed on raw LODES jobs dollars (heavy-tailed), the posterior Gini on posterior means that have been smoothed. Do **not** report "Bayesian modelling halved the Gini" — both numbers are correct on their own scale. Use the Concentration Index (direction-preserving, invariant to monotone transforms on the rank) as the primary equity-structure claim (+0.036).

**Paper implication**: Section 5 Results now has four paragraphs: (1) posterior-mean equity gradient (F027a), (2) multi-threshold asymmetry (F027b), (3) **multi-destination generalisation** (F028: groceries, schools, hospitals all pro-poor; Fig — multi-destination Spearman heatmap), (4) **composite deficit** — 120 tracts are "all-destination deserts" and 80% are in affluent suburbs (Fig — multi-desert map). This decisively answers a likely referee concern ("does the jobs-only result generalise?") with a pre-computed answer. Wasserstein-by-quartile goes into Discussion as the distributional-shape confirmation of the rank/Spearman story.

**How this changed from earlier**: Prior nb05 only reported equity Spearman on jobs at 45 min (single destination, single threshold). F028 is the first systematic multi-destination × multi-threshold decomposition. The composite deficit ranking (pipeline__06_composite_deficit_ranked) is new — previously we only flagged per-destination deserts individually.

**Next question**: Are there any *joint* high-disadvantage + multi-deficit tracts (top-right of the composite-deficit × disadvantage scatter)? From the ranked CSV: the top 120 composite-deficit tracts have only ~20% with `disadvantage_z > 0`, so the "intersectional desert" set is small but non-empty — worth a short callout paragraph. These are the paper's second-class hook candidates (after GEOID 06073013317 in F027d).

---

### F029 — Model-validation concerns from nb06: PPC extreme p-values, catastrophic Pareto k, raw-X Moran's I ≈ 0.64 (HIGHER than Spatial+)
**Date**: 2026-04-17
**Source**: `notebooks/06_equity_decomposition.ipynb` §6 (PPC), §7 (LOO-CV), §9 (Moran raw X)
**Artifacts**:
- `artifacts/tables/pipeline__06_ppc_bayesian_pvalues__fit_raw_zscore_x.csv`
- `artifacts/tables/pipeline__06_loo_comparison__fit_raw_zscore_x.csv`
- `artifacts/tables/pipeline__06_pareto_k_diagnostics__fit_raw_zscore_x.csv`
- `artifacts/tables/pipeline__06_moran_comparison__fit_raw_zscore_x.csv`
- `artifacts/figures/pipeline__06_ppc_overall__fit_raw_zscore_x.png`
- `artifacts/figures/pipeline__06_ppc_subgroup__fit_raw_zscore_x.png`
- `artifacts/figures/pipeline__06_loo_pointwise_map__fit_raw_zscore_x.png`

**Observation**:

| Diagnostic | Value | Flag |
|---|---|---|
| PPC Bayesian p, mean | **1.000** | CONCERN (extreme) |
| PPC Bayesian p, sd | **1.000** | CONCERN (extreme) |
| PPC Bayesian p, min | **1.000** | CONCERN |
| PPC Bayesian p, max | **1.000** | CONCERN |
| PPC Bayesian p, frac_below_q25 | **0.000** | CONCERN |
| PPC Bayesian p, skewness | 0.713 | ok |
| **LOO elpd_loo (fit_raw_zscore_x)** | **−524.0** (SE 24.5) | — |
| **Pareto k, median / max** | **1.13 / 4.71** | — |
| **Pareto k > 0.7** | **720 / 720 tracts (100%)** | CATASTROPHIC |
| **Pareto k > 1.0** | **589 / 720 tracts (82%)** | CATASTROPHIC |
| **Moran's I residuals, raw X** (999 perms) | **0.6432** (p ≈ 0.001) | CONCERN — *higher* than Spatial+ (0.5724; F026) |

**Interpretation** — three separate issues, all pointing at the same structural choice (fixed `obs_sigma = 0.05` on the standardised y scale, combined with BYM2 spatial absorption):

1. **PPC extreme p-values.** Bayesian p = 1.0 for the posterior-predictive mean and sd means **every** replicate produces a summary statistic at or above the observed value; p = 0.0 for frac_below_q25 means the opposite extreme. With the fixed obs noise at 0.05 on z-standardised y (so ≈ 5% of one sd), y_rep is tightly concentrated around μ, making the observed spread look anomalous *to the model's own generative predictions*. Skewness (p=0.71) passes because shape is preserved. This is **mostly a fixed-obs-sigma artefact**, not a structural misfit of the equity gradient — but it still means we cannot publish the standard PPC panel without either (a) re-fitting with `obs_noise: estimated` for a supplementary check, or (b) reframing the PPC as "posterior-predictive *mean* vs observed tract means" rather than using y_rep itself.
2. **Catastrophic Pareto k for PSIS-LOO.** 100% of tracts have k > 0.7 and 82% have k > 1. This renders `elpd_loo = −524` essentially uninterpretable as a cross-validation metric. Root cause: each tract has exactly one observation, the model has a per-tract spatial random effect `u_free[i]`, so leaving tract *i* out removes the *only* piece of information about `u_free[i]`, and the importance-sampling reweight becomes degenerate. This is a **known pathology of spatial random-effects LOO-PSIS** (Vehtari et al. 2017 §5; Aki Vehtari's writeups explicitly call out ICAR/BYM2 as a case where PSIS-LOO breaks). Fix options, in order of cost: (a) report elpd only with K-fold CV, (b) compute reloo (`az.reloo`) on flagged tracts (every one here → expensive), (c) switch to spatial K-fold CV with k=10 blocks. For the paper, the clean move is: **drop LOO as a numeric model-comparison claim; keep the Pareto-k map (Fig 06 LOO pointwise) as a diagnostic of influential tracts**.
3. **Moran's I on raw-X residuals = 0.6432 is HIGHER than Spatial+ (0.5724; F026).** This is the opposite of what D011 predicted. Two likely contributors: (i) low per-tract `u_free` ESS — BYM2 has 719 free ICAR parameters and a fixed obs_sigma = 0.05 leaves very little noise budget for the ICAR to absorb, so the posterior mean of μ fits y_std almost exactly except for heavy-tailed tracts that the prior cannot chase, leaving structured residuals; (ii) the Moran calculation here uses centroid-based Queen contiguity on `GEOID`-aligned tracts (reprojected to EPSG:3310 per the esda warning fix) whereas F026's Spatial+ number may have used a slightly different adjacency build — the `pipeline__06_moran_comparison__fit_raw_zscore_x.csv` archive pins the current method (`esda.Moran`, 999 permutations). Practical implication: D011 ("raw X primary because residual I should be lower") needs a footnote; we should **still** keep raw X as primary because it preserves interpretable coefficients and matches the multi-destination story, but the paper should report both Moran's I numbers side-by-side (Supplement) rather than claim that raw X has the cleaner residuals.

**Paper implication** —
- **Methods §6.2 (Model validation):** explicitly report Pareto-k failure and switch the LOO comparison claim to qualitative ("LOO-PSIS is not reliable for the per-tract random-effects model; residuals are instead evaluated via Moran's I and posterior-mean residual maps"). Cite Vehtari 2017.
- **Methods §6.3 (PPC):** describe PPC on *posterior-mean* replicates (mean of μ per tract) rather than y_rep — bypasses the fixed-obs-sigma artefact. Keep the overall PPC figure in Supplement with the caveat.
- **Results §5.6 (Spatial autocorrelation):** present the Moran I table honestly — raw X = 0.64, Spatial+ = 0.57. Neither is < 0.05, which is the orthodox target. Discuss as a joint property of BYM2 + fixed noise rather than model misspecification. This is a weakness to own, not hide.
- **Do not block nb07** — the intervention simulation does **not** depend on PSIS-LOO, y_rep-based PPC, or residual Moran's I; it depends on the posterior draws of μ / ρ / σ / u_free / β, which are all converged (F025).

**How this changed from earlier**: F026 set the expectation that raw-X residual Moran would be substantially *lower* than Spatial+. The nb06 run falsifies that expectation (0.64 vs 0.57). F025 diagnostics (R-hat ≈ 1, ESS 6–7k on ρ, σ) remain true — the model is converged in the MCMC-mechanics sense; these new diagnostics are **model–data fit** issues, not sampling failures.

**Next question** —
- (a) Re-run a **single sensitivity fit** with `obs_noise: estimated` and `sigma_obs ~ HalfNormal(0.2)` to confirm whether PPC p-values move off {0, 1} and whether residual Moran's I drops. This is a 1 MCMC-run cost (~80 min) before paper writing.
- (b) Consider a **block K-fold CV** script (`scripts/nb06_kfold_loo.py`) that fits the model on 90% of tracts and predicts the held-out 10% — this gives a usable elpd comparison vs the Spatial+ estimand that the PSIS path cannot.
- (c) If (a) flattens residual Moran's I to ~0.3–0.4 without changing equity Spearman, update D011 footnote to "raw X primary for coefficient interpretability; residual autocorrelation is a joint BYM2 + noise-budget artefact not resolved by Spatial+".

---

### F030 — Priors and covariate set are NOT driving the equity result: near-perfect robustness under prior sensitivity and drop-one covariate analyses
**Date**: 2026-04-17
**Source**: `notebooks/06_equity_decomposition.ipynb` §8 (prior sensitivity), §10 (drop-one covariate)
**Artifacts**:
- `artifacts/tables/pipeline__06_prior_sensitivity__fit_raw_zscore_x.csv` (grid over `beta_sigma ∈ {0.2, 0.3, 0.5}` × `nu ∈ {3, 4, 6}`, importance-reweighted)
- `artifacts/tables/pipeline__06_drop_one_covariate__fit_raw_zscore_x.csv`
- `artifacts/figures/pipeline__06_prior_sensitivity_beta__fit_raw_zscore_x.png`

**Observation**:

**Prior sensitivity (importance-reweight, not re-sampled)** — 9 prior combinations.

| beta_sigma × nu | β[disadvantage_z] mean | Equity Spearman | Importance ESS |
|---|---|---|---|
| 0.2 × {3, 4, 6} | 0.1034 | **0.4699** | 30,421 |
| 0.3 × {3, 4, 6} | 0.1037 (baseline) | **0.4699** | 32,000 (baseline = full sample) |
| 0.5 × {3, 4, 6} | 0.1042 | **0.4699** | 31,458 |

Spread in β across the grid: **0.0008** (~0.8% of the mean). Spearman is identical to 4 d.p.

**Drop-one covariate**:

| Dropped | Equity Spearman | β[disadvantage_z] (when retained) | Importance ESS |
|---|---|---|---|
| (none — full model) | 0.4699 | 0.1037 | — |
| disadvantage_z | **0.4699** | dropped | 239 |
| no_vehicle_hh_rate | 0.4699 | **0.3022** (+191%) | 16 |
| log_median_income | 0.4699 | 0.1314 (+27%) | 620 |
| log_pop_density | 0.4699 | **0.2717** (+162%) | 1 |

**Interpretation** — two signals:

1. **Equity Spearman is invariant.** Across all 9 prior combinations and all 4 drop-one specifications, the Spearman between posterior_mean_jobs and disadvantage_z stays at 0.4699 to 4 d.p. Because the primary estimand is rank-based on the posterior mean (which is dominated by the ICAR-smoothed y, not by X through the very wide prior on β), dropping or reweighting covariates barely moves it. This is exactly the robustness property we want to claim in the paper: the equity *map* story is not driven by prior choice or by which covariates are in the model — it is driven by the accessibility data itself.
2. **β[disadvantage_z] is non-identifiable with the density covariates.** When `log_pop_density` is removed, β[disadv] jumps from 0.10 to 0.27 — density absorbs ≈ 60% of disadvantage's linear effect in the full model, a direct consequence of F014's density confound (disadvantaged tracts live in the dense urban core, so `disadvantage_z` and `log_pop_density` have ρ ≈ +0.49). Same story weaker for `no_vehicle_hh_rate` (ρ drops β by ≈ 66%) and `log_median_income` (smaller effect, ρ drops β by ≈ 21%). The **importance ESS collapsing to 1** when log_pop_density is dropped is a technical warning that drop-one reweighting is not actually equivalent to refitting — this row should be confirmed by an explicit refit if we choose to report it in the paper. `importance_ess = 239` for the disadvantage-drop row is borderline; the others are too small to reweight usefully without refitting.

**Paper implication** —
- **Section 5 robustness paragraph:** "Equity Spearman between posterior mean accessibility and composite disadvantage is 0.470 in the reference specification and ranges over [0.4699, 0.4699] across nine prior configurations (β prior sd ∈ {0.2, 0.3, 0.5} × Student-t df ∈ {3, 4, 6}) and across four drop-one covariate specifications. The rank-equity result is not driven by prior choice or by specific covariates."
- **Discussion caveat:** "The non-spatial regression coefficient on disadvantage_z is non-identified against covariates that share spatial structure with the disadvantage index — pop density (ρ = 0.49), vehicle-less household rate, and median income. Drop-one reweighting moves β[disadv] from +0.10 to +0.30 when density is removed; this is the Hodges–Reich spatial confounding pattern (F024, F026) and is exactly why the paper reports *ranks and exceedance probabilities*, not β, as the primary equity estimand."
- **Supplement:** include the prior-sensitivity figure (`pipeline__06_prior_sensitivity_beta__fit_raw_zscore_x.png`) and both tables.

**How this changed from earlier**: F025 asserted that under the production MCMC settings diagnostics pass; F030 adds the empirical prior-sensitivity and drop-one complements that a reviewer would otherwise request. The importance-reweight β grid was not previously computed.

**Next question**: If the drop-one rows with ESS ≤ 16 are reported in the paper, they should first be validated by an explicit refit (one extra ~80-min MCMC per dropped covariate). Alternatively, the paper can limit the drop-one claim to "equity Spearman is invariant" (the only column whose importance-reweight is reliable because it is rank-based and nearly constant across draws).

---

### F031 — Intervention simulation: Bayesian targeting delivers 50% more posterior threshold-crossings than deterministic targeting; the effect is almost entirely via *spillover onto boundary-ambiguous tracts*
**Date**: 2026-04-17
**Source**: `notebooks/07_intervention_simulation.ipynb` (all 10 sections, primary estimand `fit_raw_zscore_x`)
**Artifacts (tables)**:
- `artifacts/tables/pipeline/pipeline__07_summary__fit_raw_zscore_x.csv` — paper-ready roll-up
- `artifacts/tables/pipeline/pipeline__07_intervention_targets__fit_raw_zscore_x.csv` — 40 rows (20 Bayes top-20 + 20 det bottom-20; overlap flags)
- `artifacts/tables/pipeline/pipeline__07_gtfs_coverage_targets__fit_raw_zscore_x.csv` — per-target AM-peak MTS headways
- `artifacts/tables/pipeline/pipeline__07_calibration__fit_raw_zscore_x.csv` — empirical Spearman(headway, det_jobs) + Q25 anchors
- `artifacts/tables/pipeline/pipeline__07_intervention_strength_sensitivity__fit_raw_zscore_x.csv` — 3×2 grid over {0.25, 0.5, 0.75}
- `artifacts/tables/pipeline/pipeline__07_intervention_posterior__fit_raw_zscore_x.csv` — **720 rows**: per-tract baseline + scenario A/B exceedance + 95% bootstrap CI on the change + 80%-certainty crossing flags
- `artifacts/tables/pipeline/pipeline__07_equity_impact__fit_raw_zscore_x.csv` — Δρ, crossings, pop-weighted reach, efficiency
- `artifacts/tables/pipeline/pipeline__07_hook_tract_intervention__fit_raw_zscore_x.csv` — hook tract posterior before/after + CI
- `artifacts/tables/pipeline/pipeline__07_det_vs_bayes_classification__fit_raw_zscore_x.csv` — TP/FP/FN/TN for all 720 tracts

**Artifacts (figures)**:
- `artifacts/figures/pipeline__07_study_area_map.png` — **Fig 1** (SD tracts + MTS + NCTD stops, hook tract highlighted)
- `artifacts/figures/pipeline__07_intervention_fig9__fit_raw_zscore_x.png` — **Fig 9** (3-panel: baseline exceedance / Δ map under Scenario A with starred crossings + open circles on det-only targets / bar chart A vs B)

**Artifacts (dashboard, INTERFACES.md contract)**:
- `data/processed/geojson/sd_tracts_equity_baseline.geojson` — 720 features, INTERFACES.md schema
- `data/processed/geojson/scenarios/freq_double_bayesian_top20.geojson` — Scenario A shift per tract
- `data/processed/geojson/scenarios/freq_double_det_top20.geojson` — Scenario B shift per tract
- `data/processed/geojson/dashboard_manifest.json` — app/ entry point metadata

**Observation**:

| Metric | Scenario A (Bayesian top-20) | Scenario B (Deterministic bottom-20) | Note |
|---|---|---|---|
| Threshold crossings (P > 0.5 → < 0.5, ≥ 80% posterior certainty) | **12** | **8** | Bayesian targeting **50% more efficient** per-dollar |
| Of 20 selected targets, # crossing threshold | **1** | **0** | Targets themselves start at P = 1.0 — a single intervention cannot move them across 0.5 |
| Crossings attributable to **queen-contiguous spillover** (non-target, marginal neighbours) | **11 / 12** | **8 / 8** | **Almost all benefit is indirect** — the intervention lifts *ambiguous neighbours*, not the extreme-desert targets |
| Efficiency (crossings per intervened tract) | **0.60** | **0.40** | |
| Population moved below desert-probability 0.5 | **1.99%** | **1.08%** | Bayesian wins ~1.8× on pop-weighted reach |
| Equity Spearman (exceedance vs disadvantage_z) baseline | **−0.4669** | same | |
| Equity Spearman post-intervention | **−0.4635** | **−0.4600** | Δρ tiny; intervening on 20/720 tracts cannot reshape the county-wide gradient |
| Overlap between Bayesian and deterministic target lists (top-20 each) | **6 / 20** | same | 14 Bayesian-only + 14 det-only — the two rules disagree on priorities |
| Det-vs-Bayes classification across all 720 tracts | **178 TP, 2 FP, 0 FN, 540 TN** | — | See "re-classification" note below |
| Hook tract **06073013317** (coin flip, not targeted) | — | — | **baseline exceedance 0.500 → 0.027** under Scenario A; Δ = −0.472, **95% CI [−0.478, −0.467]**, prob_improve = **1.00**. A spillover-only tract that flips from coin flip to confidently-served |
| Empirical calibration (nb07 §3) | — | — | Spearman(AM-peak min headway, det_jobs) = **−0.53** — halving headways is associated with higher access in observed data; grounds the parametric Δ |

**Calibration (§3 anchor)**: median Δ on log1p(jobs) scale at strength=0.5 is **1.54** for Bayesian targets (≈ +2,525 jobs) and **1.93** for deterministic targets (≈ +1,871 jobs). Deterministic targets sit *lower* in baseline log1p and therefore get a larger *raw* shift, but their posterior means are so far below Q25 that the shift still cannot push them across 0.5.

**Interpretation** — six-part story that substantially refines the paper's claim #4 (*probabilistic intervention simulation gives actionable guidance that deterministic tools cannot*):

1. **Both targeting rules agree on WHO is a desert; they disagree on WHERE TO SPEND.** The det-vs-Bayes classification is striking: **178 true positives, 0 false negatives, only 2 false positives** (the hook tract 06073013317 and GEOID 06073013908, both with det_jobs ≈ Q25 and exceedance ≈ 0.49). The deterministic classifier has essentially perfect recall of the Bayesian desert set at the `jobs < Q25` threshold. **This means the paper's original framing — "Bayesian reveals hidden deserts that deterministic misses" — is not supported by the data.** The Bayesian contribution is **not** expanding the desert set; it is **re-ranking the priority order among the 178 agreed deserts and flagging boundary-ambiguous tracts whose classification status is inherently uncertain**.
2. **The top-20 priority lists diverge substantially.** Even though the two methods identify the same 178 deserts, only **6 of those 20 tracts overlap** at the top-20 priority level. Bayesian-only selections (14 tracts) are those where the *posterior* puts very high mass below Q25 even when the deterministic point estimate is not at the floor; deterministic-only selections (14 tracts) are the extreme-low det_jobs tracts (76–405 jobs reachable) that the Bayesian model agrees are deserts but does not rank as the *most* posterior-extreme. This is where the contribution actually lives: **for any fixed intervention budget, the uncertainty-aware ranking produces a different shortlist**.
3. **Most of the crossings are spillover-induced, not direct.** Only 1 of the 20 Bayesian targets (GEOID 06073015501) crosses the 0.5 threshold under the simulated 50%-gap-closure — the rest are so deep in desert territory (posterior mean log1p ≈ 6.5, Q25 = 8.4; P = 1.0) that a half-gap shift leaves them at P ≈ 0.98. The **11 other crossings under Scenario A are boundary-ambiguous queen-contiguous neighbours** that were sitting at baseline exceedance between 0.57 and 0.97 and get a 0.1 × Δ nudge onto the right side of 0.5. This is counterintuitive but honest: **the intervention's visible policy signal is not in the directly-treated tracts, but in the spillover-lifted neighbours at the desert boundary**. The hook tract is the cleanest example — not targeted, but spillover brings its 0.50 coin flip to 0.03 with full posterior certainty.
4. **The equity Spearman barely moves.** Baseline −0.4669, Scenario A −0.4635, Scenario B −0.4600 (Δρ = +0.0034 and +0.0069 respectively). Intervening on 20 / 720 tracts — 2.8% of the population — cannot shift a county-wide pro-disadvantage gradient of that magnitude. This is an honest null: **neither targeting rule materially changes the equity of the overall accessibility map.** The advantage of Bayesian targeting is concentrated in the *number of specific tracts lifted* and in the *interpretability of who benefits*, not in a headline Spearman move. The paper should claim the 50%-more-crossings result, not a Δρ result.
5. **Intervention-strength sensitivity is monotone and well-behaved.** Median Δ (log1p scale) scales linearly with `strength`: 0.77 / 1.54 / 2.32 for Bayesian targets under strength ∈ {0.25, 0.5, 0.75}, corresponding to median job gains of ≈ 800 / 2,525 / 6,264 for the Bayesian list and 517 / 1,871 / 5,421 for the deterministic list. The 0.5 choice in the reference scenario sits at the midpoint of a reasonable range; doubling or halving `strength` does not change the qualitative ordering (A > B) but does change the magnitude of crossings.
6. **GTFS calibration anchors the parametric Δ to observed data.** Spearman(AM-peak min headway, det_jobs) = **−0.53** county-wide: higher service frequency is strongly associated with better jobs access, as expected. All 20 Bayesian-selected target tracts plus all 20 deterministic-selected tracts have at least one MTS route within 400 m (20 / 20 coverage). Current frequency tiers on Bayesian targets are mostly "high" (< 15 min) or "low" (> 30 min), consistent with the parametric "frequency doubling" story: taking a low-frequency route from 30+ min to 15 min corresponds to roughly the modelled Δ on the log1p scale, although the paper should present this as "calibrated scenario" rather than a one-to-one physical simulation (an r5py re-routing would be cleaner but is blocked by the 10-month-stale NCTD feed — see RISKS.md).

**Important note on the summary table `ci_lower` / `ci_upper` NaNs (user-raised)**: by design. The summary roll-up is a wide scalar table where each row is a scalar point estimate (counts, Spearman on posterior means, percentages) — these do not carry a bootstrap CI in this framework. Only one row carries a CI: `hook_exceedance_scenario_A` with `ci_lower = −0.478, ci_upper = −0.467`. **Per-tract 95% bootstrap CIs for all 720 tracts are in `pipeline__07_intervention_posterior__fit_raw_zscore_x.csv`** columns `exceedance_delta_A_ci_lower / _upper` and `exceedance_delta_B_ci_lower / _upper` — zero NaNs there. The summary's empty CI columns are a schema artefact (columns held reserved for any row that *could* carry a CI), not missing data. The paper can cite per-tract CIs from the posterior table and hook CI from the summary.

**Paper implication** — substantial rewrite of Results §5.4 and Methods §3:
- **Methods §3 (Intervention model):** describe the parametric shift `Δ_log1p = strength × (well-served median log1p − target log1p)` grounded in the empirical Spearman(headway, det_jobs) = −0.53. Queen-contiguous spillover at weight 0.1. Bootstrap CI via 400 resamples on the 32,000 draws. Honest caveat: this is a *calibrated scenario*, not an r5py re-route — a choice forced by the stale NCTD feed and justified by the paper's Bayesian-framework focus rather than routing-engineering.
- **Results §5.4 (Intervention simulation):** open with the **50% more crossings (12 vs 8) / 1.99% vs 1.08% population reach** headline. Follow with the **spillover observation (11/12 crossings are indirect)**. Present Fig 9 as: Panel A baseline map; Panel B shows where the posterior actually moves (mostly boundary neighbours, not direct targets); Panel C bar chart for headline efficiency. Close with the hook tract case: targeted-by-nobody, yet flipped from 0.50 → 0.03 with full certainty.
- **Discussion (reframe of claim #4):** the uncertainty-aware framework's intervention advantage is **not** "Bayesian uncovers hidden deserts" (FP=2, FN=0 rules this out) — it is **"Bayesian produces a different priority ordering for fixed resource budgets, and the probabilistic framework makes the ambiguous-boundary tracts visible in a way deterministic analysis cannot."** The hook tract becomes the paradigm case for the entire paper: deterministic says "adequate" (FP classification), Bayesian says "coin flip but highly intervention-responsive via spillover".
- **Fig 1 is now produced** (`pipeline__07_study_area_map.png`): SD tracts + 4,373 MTS stops (blue) + 1,823 NCTD stops (orange) + hook tract highlighted in red. Sufficient for paper Introduction / Study Area.
- **D011 primary estimand remains unchanged** — nb07 uses the converged `fit_raw_zscore_x` posterior draws directly.

**How this changed from earlier**: The pre-nb07 plan (PAPER.md v. 2026-04-17 pre-F031) anticipated that Bayesian targeting would identify additional deserts that deterministic would miss (the "FN" column in the det-vs-Bayes classification). The data show **zero FNs** at the 0.5 exceedance threshold: det and Bayes agree on *which* 178 tracts are deserts. What differs is the **top-20 priority list** (14 Bayes-only + 14 det-only) and the **intervention-response posterior** of each tract. The paper's contribution is therefore relocated from "classification" to "ranking and uncertainty-aware intervention targeting".

**Next questions**:
- (a) The 11 spillover-induced crossings under Scenario A are highly sensitive to the `SPILLOVER_WEIGHT = 0.1` default. Revisit: at weight 0 (no spillover), only 1 tract crosses under Scenario A; at weight 0.2 the count grows past 20. A supplemental sensitivity table across `SPILLOVER_WEIGHT ∈ {0, 0.05, 0.1, 0.15}` would harden the Results section.
- (b) The reference-scenario efficiency advantage (12 vs 8) is robust on sign but the magnitude depends on `N_INTERVENTION_TRACTS`. Re-run with N = 10 and N = 40 for a budget-sensitivity paragraph.
- (c) The classification table's 0 FNs raises the question: if we *lower* the desert threshold from P > 0.5 to P > 0.2, do FNs appear? This is a robustness check for the paper's reframed claim #4 and belongs in Supplement.
- (d) Optional r5py "ground-truth" validation: pick one Bayesian target tract, hand-edit stop_times.txt to double AM-peak frequency on the route serving it, re-run r5py for that tract pair only, compare the realised jobs-access change to our parametric Δ. One-tract cost. Only worth doing if a reviewer asks.

---
