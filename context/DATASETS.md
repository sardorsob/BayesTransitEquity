# DATASETS.md
All data sources, schemas, licensing, and download notes.
Update this file whenever a new dataset is added or a feed is refreshed.

---

## Transit Data

**Automated GTFS URLs:** `scripts/download_data.py` reads `mobility_db_id` from the city config, downloads the Mobility Database catalog to `data/interim/mobility_database/feeds_v2.csv` (refreshed when older than `mobility_catalog.max_age_days` in `configs/defaults.yaml`), and uses **`urls.latest`** (`https://files.mobilitydatabase.org/mdb-*/latest.zip`) for the actual download. Agency `download_url` in YAML is only a fallback. Optional flag: `--refresh-mobility-catalog`.

### MTS (San Diego Metropolitan Transit System)
- **Type**: GTFS Schedule
- **Direct download URL**: `http://www.sdmts.com/google_transit_files/google_transit.zip`
- **Info page**: https://www.sdmts.com/business-center/app-developers
- **Mobility Database ID**: mdb-13
- **License**: Public / Open
- **Coverage**: Bus + Trolley in central/south SD
- **Raw path**: `data/raw/gtfs/mts/` (zip + `extracted/` subfolder)
- **Download script**: `python scripts/download_data.py --sources gtfs`
- **Notes**: Feed updates periodically — no versioned URL. Run script to refresh. Check feed_info.txt for validity window.
- **Last downloaded**: —
- **Feed version**: —
- **Feed valid**: — → —

### NCTD (North County Transit District)
- **Type**: GTFS Schedule
- **Direct download URL**: `http://www.gonctd.com/google_transit.zip`
- **Info page**: https://www.gonctd.com/about/open-data/
- **Mobility Database ID**: mdb-14
- **License**: Public / Open
- **Coverage**: North County SD (Coaster, Sprinter, Breeze buses)
- **Raw path**: `data/raw/gtfs/nctd/` (zip + `extracted/` subfolder)
- **Download script**: `python scripts/download_data.py --sources gtfs`
- **Last downloaded**: —
- **Feed version**: —
- **Feed valid**: — → —

### GTFS-Realtime (Optional / Phase 2)
- **Source**: MTS real-time feed (requires API key)
- **Purpose**: Reliability-adjusted accessibility (scheduled vs realizable)
- **Status**: Deferred to Phase 2. Schedule-only for initial model.

---

## Demographics + Geography

### Census Tract Shapefiles (TIGER/Line)
- **Source**: US Census Bureau TIGER/Line
- **Direct download URL**: `https://www2.census.gov/geo/tiger/TIGER2023/TRACT/tl_2023_06_tract.zip`
- **Year**: 2023 (matches ACS year in config)
- **Raw path**: `data/raw/census/tl_2023_06_tract.zip` + `tl_2023_06_tract/` (extracted)
- **Download script**: `python scripts/download_data.py --sources census`
- **Key fields**: GEOID (11-digit FIPS = state+county+tract), geometry
- **Note**: State-level file (all CA tracts). Filter to county_fips=073 during EDA.
- **Last downloaded**: —

### ACS 5-Year Estimates (Demographics)
- **Source**: Census Bureau API — ACS 2023 5-year estimates
- **API URL**: `https://api.census.gov/data/2023/acs/acs5?get=<vars>&for=tract:*&in=state:06%20county:073`
- **Raw path**: `data/raw/census/acs5_2023_sd_county.json`
- **Download script**: `python scripts/download_data.py --sources census`
- **Variables (E = estimate, M = margin of error)**:
  - `B01003_001` — Total population
  - `B08201_002` — No-vehicle households
  - `B19013_001` — Median household income
  - `B17001_002` — Below poverty level
  - `B18101_001` — Total (disability base)
  - `B03002_001/003/012/004` — Race/ethnicity (total, White NH, Hispanic, Black)
- **Last downloaded**: —
- **N tracts (SD county)**: —

---

## Opportunity / Destination Data

### Jobs (LODES / LEHD)
- **Source**: US Census LEHD Origin-Destination Employment Statistics — LODES8
- **Direct download URL**: `https://lehd.ces.census.gov/data/lodes/LODES8/ca/wac/ca_wac_S000_JT00_2021.csv.gz`
- **Year**: 2021 (most recent stable LODES8 release)
- **Download script**: `python scripts/download_data.py --sources lodes`
- **Purpose**: Job counts by census block → aggregate to tract for cumulative opportunity metric
- **Raw path**: `data/raw/external/lodes/ca_wac_2021.csv.gz`
- **Note**: State-wide CA WAC (gzip). The published `.csv.gz` is much smaller on disk than the expanded CSV; filter to SD county (FIPS 073) when aggregating to tracts.
- **Last downloaded**: 2026-03-30 (manifest: `artifacts/logs/provenance/data_manifest_san_diego_2026-03-30_1827.json`)

### Essential Services (OpenStreetMap)
- **Source**: OSM via osmnx or Overpass API
- **Categories**: Hospitals, urgent care, grocery stores, schools
- **Raw path**: `data/raw/external/pois/`

---

## Walk Network

### OpenStreetMap Pedestrian Network (EDA / osmnx)
- **Source**: osmnx (OSM via Overpass API)
- **Purpose**: EDA walk graph; **not** used by r5py
- **Raw path**: `data/raw/osm/sd_walk_network.graphml`
- **Bounding box**: Defined in `configs/san_diego.yaml`
- **Note**: Overpass is shared; downloads can sit in a server queue many minutes with little output. Timeouts and optional `overpass_url` mirrors live under `osm:` in `configs/defaults.yaml`.

### OpenStreetMap PBF (r5py / R5)
- **Purpose**: Street network for **r5py** `TransportNetwork` in `notebooks/03_accessibility_computation.ipynb`
- **Path**: `r5.osm_pbf` in `configs/san_diego.yaml` (default `data/raw/osm/san_diego_study.osm.pbf`)
- **How to build**: Clip a regional extract to the study `bbox` with **osmium** (`osmium extract -b min_lon,min_lat,max_lon,max_lat`).
  - **Recommended regional file**: [Geofabrik California](https://download.geofabrik.de/north-america/us/california-latest.osm.pbf) (~1.1 GB). Save to `r5.geofabrik_source_pbf` (default `data/interim/osm/california-latest.osm.pbf`) or pass `--input` to `scripts/extract_osm_pbf.py`.
  - **Automated**: `python scripts/download_data.py --config configs/san_diego.yaml --sources osm_pbf --download-geofabrik-ca` (streams the Geofabrik file if missing, then clips). Requires `osmium` on `PATH`.
  - **Manual clip**: `python scripts/extract_osm_pbf.py --input <path/to/california-latest.osm.pbf>`
- **Install osmium**: [osmium-tool](https://osmcode.org/osmium-tool/) — e.g. `conda install -c conda-forge osmium-tool`. Run `download_data.py` / `extract_osm_pbf.py` from an **activated** conda env so `CONDA_PREFIX` is set (scripts also look for `%CONDA_PREFIX%\Library\bin\osmium.exe`). Or set **`OSMIUM_EXE`** to the full path to `osmium.exe`. Optional WSL clip: set **`BAYESTRANSIT_OSMIUM_USE_WSL=1`** and install `osmium-tool` inside WSL.

---

## Licensing Notes
- GTFS feeds: public, but cite agency in paper
- Census data: public domain
- OSM: Open Database License (ODbL) — must credit OpenStreetMap contributors
- LODES: public domain
