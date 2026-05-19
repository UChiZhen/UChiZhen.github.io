# Chicago Spatial Alpha Report

This repository hosts a graduate GIS final project analyzing where strong Chicago transit access coincides with comparatively low realized investment. The rendered report is published as a self-contained Quarto HTML page for GitHub Pages.

Live report: https://uchizhen.github.io/

## Research Question

Where in Chicago do statistically significant clusters of high transit accessibility coincide with low realized investment, and what is their demographic profile?

The project frames this as a spatial version of place-based alpha: tracts with strong location fundamentals, measured through CTA rail accessibility, but lower observed building-permit investment per housing unit may warrant further due diligence for catalytic or impact-oriented capital.

## Methods

The analysis is conducted at the Census tract scale in Chicago.

- 2-Step Floating Catchment Area (2SFCA): estimates tract accessibility to CTA L stations using a 1,000 meter Euclidean catchment around tract centroids and station points.
- Catchment sensitivity: recomputes 2SFCA at 800 m, 1,000 m, and 1,200 m to check whether the accessibility signal is stable.
- Spatial Alpha Index: combines standardized transit accessibility and investment intensity as `z(SFCA) - z(log1p(permit dollars per housing unit))`.
- Local Moran's I: identifies statistically significant spatial clusters of high and low alpha values using queen contiguity weights and 999 permutations.

The headline output is the High-High LISA cluster: contiguous tracts with high spatial alpha values.

## Data Sources

- Chicago Data Portal Building Permits: investment proxy based on reported permit value, cached at `data/raw/chi_permits.csv`.
- Chicago Data Portal CTA L Stations: transit supply input for 2SFCA, cached at `data/raw/cta_l_stops.geojson`.
- ACS 5-year 2022 via `tidycensus`: population, housing units, income, renter share, and race/ethnicity profile variables.

No API keys are committed. The Census API key is read only from `Sys.getenv("CENSUS_API_KEY")` if available.

## Repository Structure

```text
.
|-- index.html                # Rendered GitHub Pages report
|-- final_project.qmd         # Quarto source
|-- README.md                 # Project overview
|-- data/
|   `-- raw/
|       |-- chi_permits.csv
|       `-- cta_l_stops.geojson
`-- .gitignore
```

## Reproducing The Report

From the repository root:

```bash
quarto render final_project.qmd
```

The report renders to `index.html` via the Quarto YAML option `output-file: index.html`.

The source uses `here()` for paths and expects cached raw data under `data/raw/`. If the cached files are removed, the report can re-download the Chicago Data Portal inputs. ACS data are pulled through `tidycensus`.

## Notes And Limitations

- Accessibility is based on Euclidean distance, not street-network distance.
- CTA L stations are treated as one unit of transit supply per station.
- Building permits are a flow proxy for investment and do not capture all capital sources.
- The alpha index is descriptive, not causal; it is best interpreted as a screening signal for further neighborhood-level due diligence.
