# Social Deprivation Index (SDI)

This repository contains code and documentation to construct the **Social Deprivation Index (SDI)** using U.S. Census Bureau American Community Survey (ACS) data at multiple geographic levels (county, census tract, ZCTA, PCSA).

We build on and extend the SDI originally developed and maintained by the **Robert Graham Center** and colleagues. Their work defines the core SDI concept and releases SDI values through **2019** (based on 2015–2019 ACS 5-year data). This repository:

- Documents the **original SDI specification** and components.
- Provides code to **reproduce SDI following the published methodology**.
- Provides **updated SDI estimates for 2020 and later** (ACS 5-year releases), using the same conceptual framework.
- Integrates SDI into a broader family of deprivation indices (ReADI, NA-ADI, NSS7, FDEP, etc.), with links to those repos.

> **Important:**  
> We are **not** the original creators of SDI. The original methodology and SDI through 2019 are authored and distributed by the Robert Graham Center. This repo is an independent, methods-transparent implementation and extension to more recent ACS years.

---

## 1. Background

Social determinants of health (SDOH) are central to understanding and addressing inequities in health care access, outcomes, and spending. The **Social Deprivation Index (SDI)** is a composite area-level measure designed to capture social and economic disadvantage and to support:

- identification of communities with high social risk,
- evaluation of associations between social deprivation and health outcomes,
- potential adjustment of quality measures and payment models.

The original SDI was developed by **Butler et al. (2013)** using 2005–2009 ACS data at the **Primary Care Service Area (PCSA)** level and later extended to counties, census tracts, and ZCTAs using subsequent ACS 5-year estimates.

This repository takes over **from ACS 2016–2020 onward**, maintaining consistency with the original SDI concept while making the construction process reproducible and fully open.

---

## 2. What this repository covers

### 2.1 Scope

- **Geographies**
  - Census tract
  - County
  - ZCTA (Zip Code Tabulation Area)
  - PCSA (Primary Care Service Area), where feasible given geography crosswalks

- **Time**
  - Documentation and reference to **historical SDI (2008–2012 through 2015–2019)** as released by the [Robert Graham Center](https://www.graham-center.org/maps-data-tools/social-deprivation-index.html).
  - **New SDI builds starting with 2020 SDI** (e.g., ACS 2016–2020, 2017–2021, etc.), constructed using the published SDI methodology.

### 2.2 Relationship to original SDI

- For **≤2019 SDI**:
  - We treat the Robert Graham Center SDI as **canonical**.
  - This repo may include code that reproduces or validates those releases, but the official values remain those published on their site.

- For **2020+ SDI**:
  - This repo is the primary source of code and (optionally) derived indices.
  - We follow the same conceptual and statistical approach (variables, factor analysis, centiles), updated to newer ACS years.

---

## 3. SDI definition and components

The SDI is a composite index summarizing seven area-level characteristics drawn from ACS 5-year estimates:

1. **Percent living in poverty**  
2. **Percent of adults (≥25) with less than 12 years of education**  
3. **Percent of non-employed adults aged 16–64**  
4. **Percent of households in renter-occupied housing**  
5. **Percent of households in crowded housing** (occupants per room ≥ 1.01)  
6. **Percent of single-parent families with children < 18**  
7. **Percent of households with no vehicle**

Each component is computed as an area-level proportion (e.g., tract-level % in poverty). Components are:

- converted to centiles across all areas at a given geography and period, and  
- combined using **factor analysis** to derive a single deprivation score.

The original SDI development:

- started from a larger set of candidate ACS measures,
- used factor loadings > 0.60 to select the final seven components,
- evaluated alternate choices for employment (non-employed vs unemployed), ultimately selecting **non-employed** due to stronger loadings.

We closely follow this specification for 2020+ SDI builds. Any deviations (e.g., handling of missing data, geography crosswalk updates) are documented in `docs/methods_sdi_20xx.md`.

---

## 4. Methodology (implementation here)

High-level workflow (details in the `R/` scripts and `docs/`):

1. **Download and prepare ACS data**
   - Pull ACS 5-year summary files for the relevant period (e.g., 2016–2020).
   - Extract the ACS tables needed for each component (poverty, education, tenure, vehicles, etc.).
   - Construct tract-/county-/ZCTA-level numerators and denominators.

2. **Compute component proportions**
   - Compute the seven component percentages per area, matching the original SDI definitions.

3. **Transform to centiles**
   - Rank each component across all areas for a given geography and convert to centiles (0–100).

4. **Factor analysis**
   - Perform factor analysis on the component centiles.
   - Confirm that factor structure is consistent with the original SDI (e.g., one dominant factor, similar loadings).
   - Use factor loadings as weights to construct the composite SDI score.

5. **Standardization and outputs**
   - Produce:
     - component raw proportions,
     - component centiles,
     - raw SDI factor scores,
     - standardized SDI score (e.g., z-score or rescaled index),
   - for each geography and time period.

6. **Quality checks**
   - Compare distributions of new SDI scores to historical SDI (where overlapping years exist).
   - Check stability of factor loadings across time and geographies.

---