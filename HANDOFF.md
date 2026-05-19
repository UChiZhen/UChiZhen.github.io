# Handoff: Chicago Spatial-Alpha Project

You (Claude/Codex) are picking up a graduate GIS final project after a cleanup pass. This file is your current project context. **First task**: read this entire doc, then read `final_project.qmd`, then inspect the rendered `index.html` and run the remaining review priorities in §9.

---

## 1. Project meta

- **Course**: graduate-level "GIS Applications in the Social Sciences"
- **Deliverable**: a single self-contained Quarto HTML report (`embed-resources: true`)
- **Working directory**: `/Users/zz/UChiZhen.github.io/`
- **Entry point**: `final_project.qmd`
- **Author**: Zhen Zeng (`zzeng@impactalpha.com`)
- **Current status**: renders end-to-end from the repo root; latest cleanup flattened the GitHub Pages layout and improved the linked-browser table and discussion text.

## 2. Hard requirements (graded against these)

| Requirement | Where satisfied |
|---|---|
| ≥2 data sources, not solely ACS | Chicago Data Portal Permits + CTA L Stations + ACS (3 total) |
| Smallest spatial scale (sub-county) | Census tracts, Cook County clipped to City of Chicago by centroid-in-polygon |
| ≥2 advanced methods, combined | **2SFCA** (proximity/exposure) + **Local Moran's I / LISA** (spatial autocorrelation) |
| Quarto HTML, self-contained | YAML has `embed-resources: true` |
| YAML: `code-fold`, `code-tools`, `toc`, `toc-float` | Present |
| Global chunk opts: `results: hide`, `warning: false`, `message: false` | Set in YAML `execute:` block |
| `::: {.panel-tabset}` for maps/tables | Used for data-source tabs and the three result maps |
| `gt` publication tables | Top-15 alpha table + LISA summary table |
| `leaflet` with `addLayersControl`, popups, legends | All three maps |
| `crosstalk` linked map + datatable | §4.4 of the qmd |
| `here()` for paths | All file I/O |
| No hardcoded API keys; `Sys.getenv()` | `CENSUS_API_KEY` only; lives in `~/.Renviron` |
| `renv`-ready | Softened wording: project can be pinned with renv; no `renv.lock` is currently committed |

## 3. Research framing

> **Question**: Where in Chicago do statistically significant clusters of *high transit accessibility* coincide with *low realized investment*, and what is their demographic profile?

The framing is **impact-investing "alpha"**: location-fundamentals signal (transit) that has *not yet* been matched by capital flow (permit dollars) is the spatial analog of risk-adjusted excess return. **High–High LISA clusters of the alpha index = candidate target neighborhoods.**

## 4. Methodology (what's in the qmd, and why)

### 4.1 Method A — 2SFCA (Luo & Wang 2003)

- **Supply**: CTA "L" rapid-transit stations (one unit per station).
- **Demand**: tract population.
- **Catchment**: 1,000 m Euclidean radius from tract centroid / station point.
- **Distance**: Euclidean (documented limitation; network distance is the obvious upgrade).
- Score scaled by 1e5 for readability.

### 4.2 Method B — Local Moran's I on the Spatial Alpha Index

- **Alpha index**: `z(SFCA) − z(log1p(permit_$_per_HU))`
- **Weights**: queen contiguity via `rgeoda::queen_weights()`.
- **Permutations**: 999, `set.seed(20260517)`.
- **Output**: 5-category cluster labels (HH / LL / HL / LH / Not significant), plotted as a categorical choropleth.

### 4.3 Decisions the user already committed to

These were chosen from a recommended-vs-alternatives prompt; **do not re-litigate** unless you find a concrete reason:

- Investment proxy = **Building permits $/HU** (not HMDA, not composite).
- Amenity = **CTA L stations only** (not + grocery, not jobs).
- Distance = **Euclidean 1 km** (not street-network).
- LISA flavor = **univariate on composite z-score** (not `local_bimoran`).

## 5. Data sources

| # | Source | Endpoint | Cached to |
|---|---|---|---|
| 1 | Chicago Data Portal — Building Permits | `data.cityofchicago.org/resource/ydr8-5enu.csv` | `data/raw/chi_permits.csv` |
| 2 | Chicago Data Portal — CTA L Stations | `data.cityofchicago.org/resource/8pix-ypme.geojson` | `data/raw/cta_l_stops.geojson` |
| 3 | ACS 5-yr 2022 via `tidycensus` | Census API | not cached on disk (in-session) |

- Socrata permits pull pages 50k rows/request, filters to construction-related types with `reported_cost > 0` in `2019-01-01 ≤ issue_date < 2025-01-01`.
- All Socrata chunks are read with `col_types = cols(.default = "c")` to avoid cross-chunk type drift; numeric coercion (`reported_cost`, lat/long) happens after `bind_rows`.
- TIGER tracts are 2022 vintage, projected to **EPSG:26971** (NAD83 / Illinois East, meters).

## 6. File layout

```
./
|-- final_project.qmd      entry point; renders to index.html
|-- index.html             GitHub Pages output
|-- HANDOFF.md             this file
`-- data/
    `-- raw/               Socrata caches land here
```

## 7. Environment

- macOS (Darwin 24.6.0), R from default user library.
- **API keys**: `CENSUS_API_KEY` is set in `~/.Renviron`. Do not commit it; do not reproduce it in the qmd; only access via `Sys.getenv()`.
- **Installed packages** confirmed present: `tidyverse, sf, tigris, tidycensus, leaflet, gt, DT, crosstalk, htmltools, httr2, janitor, scales, RColorBrewer, units`. The `one-time-setup` chunk only installs the two that were missing on this machine: `here`, `rgeoda`. If you change library dependencies, update that chunk accordingly.

## 8. Completed changes since original handoff

These changes have already been made in `final_project.qmd`, `index.html`, or repo organization:

- Flattened the project from `final_project/` into the GitHub Pages repo root (`/Users/zz/UChiZhen.github.io/`).
- Added `output-file: index.html` so GitHub Pages serves the report directly.
- Updated all `here()` paths from `here("final_project", "data", ...)` to `here("data", ...)`.
- Added `.gitignore` for `.DS_Store`, Quarto caches, R workspace/history files, and generated `final_project.html`/knit intermediates.
- Removed empty `data/processed/`; only `data/raw/` remains.
- Removed `.DS_Store` files from the repo.
- Fixed LISA labeling by using `lisa_clusters(lm_res, cutoff = 0.05)` before indexing into `lisa_labels()`.
- Added 2SFCA catchment-radius sensitivity table for 800 m / 1,000 m / 1,200 m.
- Removed unused `acs_vars` dead code and the incorrect `pop_nonwhite` placeholder.
- Updated Section 4.2 to include median renter share and median non-white share by LISA category.
- Fixed Section 4.4 linked DT table: clearer column names, wider table panel, `scrollX = TRUE`, nowrap styling, and column widths.
- Rewrote the discussion to remove the inaccurate "descriptive overlay" phrase and cite the actual ACS table values.
- Render verified after edits: `quarto render final_project.qmd` succeeds and creates `index.html`. Quarto reports warnings, but no render failure.
- No git staging, commit, or push has been performed.

Current untracked commit-visible files are expected to be:

```
.gitignore
HANDOFF.md
data/raw/chi_permits.csv
data/raw/cta_l_stops.geojson
final_project.qmd
index.html
```

## 9. Remaining review priorities

Review the project for improvements. **Don't refactor for taste.** Only propose changes that meaningfully raise the grade or robustness. For each finding, give: (a) location with file:line, (b) the issue, (c) the minimal patch.

### Tier 1 — correctness & reproducibility

1. **2SFCA implementation audit (mostly checked, worth re-verifying).** Re-derive Step 1 and Step 2 from Luo & Wang against the code in the `sfca` chunk. Confirm the matrix algebra is right and that the catchment-set logic doesn't double-count stations. If `station_pop` is zero anywhere, verify the `ifelse` guard does the right thing.
2. **LISA cluster labeling (patched).** Current code uses `lisa_clusters(lm_res, cutoff = 0.05) + 1`; verify if desired, but do not revert to the no-cutoff version.
3. **Centroid clipping.** Tracts are kept if their centroid lies inside the Chicago `places()` polygon. Check whether O'Hare's tract and any lakefront tract slip in/out incorrectly. The housing-unit ≥ 50 filter is a backstop, but the geometry test should be primary.
4. **Type-stable Socrata pulls.** The cached CSV is re-read with `cols(.default = "c")` and re-coerced. Make sure no downstream chunk silently relies on `read_csv` having guessed numeric.
5. **`renv` story (softened).** No `renv.lock` exists. Either leave as-is or create a lockfile only if the user wants dependency pinning; never include `CENSUS_API_KEY`.

### Tier 2 — methodological depth (good for grade)

6. **Sensitivity analysis on catchment radius (implemented).** Current report includes 800 m / 1,000 m / 1,200 m 2SFCA sensitivity after §3.1. Re-check wording/table only if needed.
7. **Enhanced 2SFCA (E2SFCA).** Weighting stations by distance with a Gaussian/log-logistic decay kernel inside the catchment. ~10 lines of code and is what current 2SFCA literature uses.
8. **Spatial Lag Model as an alpha alternative.** The discussion already promises this as an extension. If you can add a `spatialreg::lagsarlm()` regression of log permit $/HU on SFCA + ACS controls and surface residuals as model-based alpha — even in an appendix — it materially upgrades the methodology section. Keep the LISA story as the headline.
9. **Bivariate LISA cross-check.** Even with the user's univariate-composite choice, running `rgeoda::local_bimoran(SFCA, investment)` and noting whether HH-bivariate ≈ HH-univariate is a 5-line robustness check.

### Tier 3 — narrative & presentation

10. **Neighborhood naming (best remaining upgrade).** The HH tracts currently appear by GEOID only. Joining to the City of Chicago **Community Areas** layer (`data.cityofchicago.org/resource/igwz-8jzy`) by spatial intersection and adding a `community_area` column to the top-alpha `gt` table and the linked datatable would make the report immediately readable to non-GIS reviewers.
11. **Discussion interpretation (partly improved).** The discussion now interprets the High-High cluster using ACS summary values. It would still improve materially if (10) adds community-area names, then adds 2-3 sentences identifying which named areas appear.
12. **`gt` table polish.** Confirm color scales make sense for both colorblind viewers and printed B&W; check that the `data_color` reverse on permit $/HU actually maps "low investment" to the deeper red.

### Tier 4 — code hygiene (lowest priority — only flag, don't refactor)

13. The unused `acs_vars` named vector has been removed.
14. `clean_names()` is applied to `acs_raw` but the variable names in `transmute()` use both snake_case and the `_e` margin suffix; verify the names match what `clean_names()` actually produces for `tidycensus` wide output in your installed version.
15. The incorrect `pop_nonwhite = "B03002_001"` placeholder was removed with `acs_vars`.

## 10. Hard constraints — do not violate

- **No hardcoded secrets.** API key access goes through `Sys.getenv()` only.
- **No state- or country-level analysis.** Tract level is the floor *and* the ceiling for the assignment.
- **Don't break the YAML-required options** (`code-fold`, `code-tools`, `toc`, `toc-float`, `embed-resources`).
- **Don't replace `here()` with absolute paths.**
- **Don't add packages without updating the `one-time-setup` chunk.**
- **Don't add comments that explain WHAT well-named code already says** — only WHY when non-obvious. The existing code follows this; keep it that way.
- **Don't create new markdown/doc files unless asked.** This handoff is the exception.

## 11. How to deliver your review

Produce one of:

- **A patch** (`git diff`-style or direct edits) for findings you are confident about and that are <30 LOC each.
- **A prioritized punch list** for anything larger, with the location, the proposed change, and the expected impact on the grade.

Keep your final summary under ~400 words. The user values terse, decision-oriented output over exhaustive prose.
