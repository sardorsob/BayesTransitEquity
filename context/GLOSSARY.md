# GLOSSARY.md
Domain terms, acronyms, and dataset field meanings.

---

## Transit / GTFS

| Term | Definition |
|---|---|
| GTFS | General Transit Feed Specification. Standardized format for public transit schedules. |
| GTFS-RT | GTFS-Realtime. Live vehicle positions and trip updates (vs static schedules). |
| MTS | Metropolitan Transit System. Primary transit agency for central/south San Diego. |
| NCTD | North County Transit District. Serves north San Diego County. |
| Headway | Time between successive transit vehicles on a route. Lower headway = more frequent service. |
| Stop | A fixed location where passengers board/alight. |
| Trip | A single vehicle journey along a route at a specific time. |
| Feed | A GTFS data package from a single transit agency. |

## Accessibility

| Term | Definition |
|---|---|
| Cumulative opportunity | Count of destinations (jobs, services) reachable within a time threshold. |
| Gravity-based accessibility | Distance-decay weighted sum of destinations. |
| Logsum accessibility | Utility-based measure from discrete choice theory. |
| Exceedance probability | P(accessibility < threshold). Primary output of our Bayesian model. |
| Transit desert | A neighborhood with insufficient transit service relative to its population need. |
| Realizable accessibility | Accessibility accounting for actual travel time variability (vs scheduled). |
| First/last mile | The walk (or other mode) connecting a traveler to/from a transit stop. |
| r5py | Python wrapper for Conveyal's R5 routing engine. Used to compute accessibility from GTFS. |

## Bayesian / Statistics

| Term | Definition |
|---|---|
| Posterior | P(parameter \| data). The updated belief about a quantity after seeing data. |
| Prior | P(parameter). Belief about a quantity before seeing data. |
| MCMC | Markov Chain Monte Carlo. Algorithm for sampling from a posterior distribution. |
| Credible interval | Bayesian analog of confidence interval. Contains true value with stated posterior probability. |
| R-hat | Convergence diagnostic. Values near 1.0 indicate chains have mixed. Target < 1.01. |
| ESS | Effective Sample Size. Number of independent equivalent samples. Higher is better. |
| Hierarchical model | Model with parameters at multiple levels (e.g., tract-level and city-level). |
| Partial pooling | Sharing information across groups (tracts) via a hierarchical prior. |
| CAR | Conditional Autoregressive model. Spatial random effects model for areal data. |
| BYM | Besag-York-Mollié model. CAR model with an additional unstructured random effect. |
| Spatial random effect | A latent variable capturing spatially correlated residual variation. |
| PyMC | Python library for probabilistic programming and Bayesian inference. |
| NumPyro | JAX-based probabilistic programming library. Faster than PyMC at scale. |

## Information Theory

| Term | Definition |
|---|---|
| KL divergence | Kullback-Leibler divergence. Measures how one distribution differs from a reference. Asymmetric. |
| JS divergence | Jensen-Shannon divergence. Symmetric, bounded [0,1] version of KL divergence. |
| Wasserstein distance | Earth Mover's Distance. Geometric distance between distributions. Preferred for spatial comparisons. |
| Entropy | Measure of uncertainty in a distribution. Higher = more uncertain. |
| Surprisal | -log P(x). The information content of a single observation. |

## Census / Geography

| Term | Definition |
|---|---|
| Census tract | Small, relatively permanent statistical subdivision of a county (~4,000 people). Primary unit of analysis. |
| GEOID | 11-digit FIPS code uniquely identifying a census tract (state + county + tract). |
| ACS | American Community Survey. Annual Census Bureau survey providing demographic estimates. |
| TIGER | Topologically Integrated Geographic Encoding and Referencing. Census shapefile system. |
| LODES | LEHD Origin-Destination Employment Statistics. Job location data by census block. |
