# Blog 05 — Is Living Alone Easier Where More People Do It?

## Overview

This project examines living-alone prevalence and rental affordability across the 50 U.S. states using the **2024 American Community Survey (ACS) one-year sample**, accessed through **IPUMS USA**. It asks whether states where living alone is more common are also places where renting alone is more affordable.

The study compares two **different populations**: (1) all residents when measuring the prevalence of living alone and (2) adult, one-person renter households with valid positive income when measuring rent burden. This is a descriptive, state-level analysis, not a causal study. Washington, D.C., is retained separately in the processed data and statistical reports but is excluded from the 50-state maps and main comparison.

The reproducible R workflow consists of **four scripts**, producing **one processed RDS, two CSV reports, and three PNG figures**.

## Data Sources and Usage Requirements

### 1. IPUMS USA — 2024 ACS one-year microdata

- **Underlying survey:** U.S. Census Bureau, 2024 American Community Survey (ACS) one-year sample.
- **Data access and harmonization:** [IPUMS USA](https://usa.ipums.org/usa/).
- **Sample:** `us2024a` (`YEAR = 2024`, `SAMPLE = 202401`).
- **Local extract:** `data_raw/usa_00002.xml` (DDI/codebook) and `data_raw/usa_00002.dat.gz` (compressed person-level records).
- **API state file:** `data_raw/extract_id.txt` holds the extract task number, not the data itself.

Script 01 requests rectangular, fixed-width person records with the household members and variables needed to reproduce the study, including household weights (`HHWT`), person weights (`PERWT`), and 80 household replicate weights (`REPWT1`–`REPWT80`). The extraction number (`00002`) is **not fixed**: a new IPUMS request can return another `usa_XXXXX` filename. Scripts 02–03 locate a single matching `.xml`/`.dat.gz` pair automatically; keep exactly one such pair in `data_raw/`.

**Important — access is not the same as redistribution permission.** These data come from **IPUMS USA and the U.S. Census Bureau. The relevant terms for the IPUMS extract are the [IPUMS USA Terms of Use](https://usa.ipums.org/usa/terms.shtml). IPUMS generally requires permission to redistribute its sample data, except for a limited journal-publication provision described in its terms. **Before including the raw IPUMS extract in a public GitHub repository, the maintainer must verify that the intended redistribution is authorized.** Any repository recipient must also follow the applicable IPUMS terms and citation requirements; receiving a copy does not grant unrestricted redistribution rights. If permission has not been obtained, the microdata should not be publicly shared, and reproducers should obtain their own extract through IPUMS instead.

The derived `data_processed/single_person_renters_clean.rds` contains household-level microdata and should likewise **not be publicly redistributed without confirming permission**. The repository's code does not confer a separate license to the underlying data. Never commit an IPUMS API key or the private `.Renviron` file.

### 2. U.S. Census Bureau — 2024 state boundaries

- **File:** `data_raw/cb_2024_us_state_20m.zip`.
- **Official download:** [2024 cartographic boundary ZIP, states, 1:20,000,000](https://www2.census.gov/geo/tiger/GENZ2024/shp/cb_2024_us_state_20m.zip).

Script 04 uses this ZIP to draw state boundaries and downloads it **only if it is missing**. The shapefile is read directly from the ZIP; no permanent unzipped shapefile directory is created. It supplies geometry, not income, rental, or population measurements. Attribute the boundary data to the U.S. Census Bureau.

## Repository Structure

Paths are relative to the RStudio project root, `RodgersYe_6400_Blog05/`.

```text
RodgersYe_6400_Blog05/
├── code/
│   ├── 01_get_raw_data.R                 # Obtain or reuse the IPUMS extract
│   ├── 02_clean_data.R                   # Clean records; write the RDS
│   ├── 03_analyze_data.R                 # Validate and calculate statistics
│   └── 04_make_figures.R                 # Download/read boundaries; create PNGs
├── data_raw/
│   ├── usa_00002.xml                    # IPUMS DDI/codebook; extract ID may vary
│   ├── usa_00002.dat.gz                 # IPUMS person-level microdata; restricted redistribution
│   ├── extract_id.txt                   # Account-specific API task state
│   └── cb_2024_us_state_20m.zip         # Census state boundaries
├── data_processed/
│   ├── single_person_renters_clean.rds  # Derived household microdata; check redistribution rights
│   ├── data_validation_report.csv       # Consolidated validation checks
│   └── statistical_summary.csv         # Estimates for analysis and figures
├── figures/
│   ├── fig01_severe_burden_intervals.png
│   ├── fig02_severe_rent_burden_map.png
│   └── fig03_living_alone_population_share_map.png
├── RodgersYe_6400_Blog05.Rproj          # RStudio project
├── README.md
├── .gitignore                           # Git exclusions; inspect before publication
├── .Renviron                            # Local secret; DO NOT UPLOAD
├── .Rhistory                            # Local R session history; not needed for reproduction
├── .Rproj.user/                         # Local RStudio state; not needed for reproduction
└── .git/                                # Local Git metadata; not research input
```

## Software Requirements

Use **R and RStudio** (recommended), with the following R packages:

```r
install.packages(c(
  "here", "ipumsr", "data.table", "tidyselect",
  "sf", "ggplot2", "scales", "patchwork", "ggrepel"
))
```

## How to Reproduce the Research

### Step 1 — Open the project

Clone or download the Blog 05 repository and open `RodgersYe_6400_Blog05.Rproj` in RStudio. Run R from this **project root**. All four scripts use project-relative paths and are stored in `code/`.

### Step 2 — Set up IPUMS access

Register with [IPUMS USA](https://usa.ipums.org/usa/) and obtain your own API key from the [IPUMS Developer Portal](https://developer.ipums.org/). Create a **local, uncommitted** `.Renviron` file in the project root containing:

```text
IPUMS_API_KEY=YOUR_PERSONAL_API_KEY
```

Restart R (or load the file using `readRenviron(here::here(".Renviron"))`). Do not publish the key.

The default `auto` mode in script 01 checks for a complete local pair first. If none exists, it resumes the locally saved extract task or submits a new task, waits for IPUMS to prepare it, and downloads the `.xml` and `.dat.gz` files **directly into `data_raw/`**. The task number is saved in `data_raw/extract_id.txt`.

Optional API modes can be selected in the console before running script 01:

| Mode | Meaning |
|---|---|
| `"auto"` (default) | Reuse a complete local pair, or resume/submit and download. |
| `"check"` | Check sample information and show the extract specification, without submitting. |
| `"submit"` | Submit a new extract when appropriate and save its task ID, without downloading. |
| `"download"` | Resume download from a previously saved task ID; do not submit a new extract. |

### Step 3 — Run the four scripts in order

From the RStudio project root:

```r
options(blog5.ipums_stage = "auto")

source(here::here("code", "01_get_raw_data.R"))
source(here::here("code", "02_clean_data.R"))
source(here::here("code", "03_analyze_data.R"))
source(here::here("code", "04_make_figures.R"))
```

Each script runs when sourced; you do not need to call its main function again. Their dependencies and outputs are:

| Script | Inputs | Outputs |
|---|---|---|
| `01_get_raw_data.R` | Existing raw pair, or IPUMS API credentials/task | `data_raw/usa_XXXXX.xml`, `data_raw/usa_XXXXX.dat.gz`, and (on new submission) `extract_id.txt` |
| `02_clean_data.R` | IPUMS XML + compressed microdata | `data_processed/single_person_renters_clean.rds` |
| `03_analyze_data.R` | Raw pair + cleaned RDS | `data_processed/data_validation_report.csv` and `data_processed/statistical_summary.csv` |
| `04_make_figures.R` | Both CSVs + Census boundary ZIP | Three PNG figures in `figures/` |

Scripts 02 and 03 read the raw extract in chunks. Script 03 rereads raw records to verify household composition and source consistency, so a **full rerun requires the raw pair even if the RDS already exists**. Script 04 can run separately from matching, completed CSV reports if you only need to regenerate the figures. It reads the Census ZIP from `data_raw/`, downloading it there if necessary.

### Step 4 — Check that the run succeeded

Review the validation report and the generated files:

```r
validation <- data.table::fread(
  here::here("data_processed", "data_validation_report.csv")
)
validation[, .N, by = status]
validation[status %in% c("FAIL", "WARN")]
validation[section == "pipeline" & check == "analysis_completed"]

file.exists(here::here("data_processed", "statistical_summary.csv"))
file.exists(here::here("figures", "fig01_severe_burden_intervals.png"))
file.exists(here::here("figures", "fig02_severe_rent_burden_map.png"))
file.exists(here::here("figures", "fig03_living_alone_population_share_map.png"))
```

The analysis should finish with an `analysis_completed` row marked `PASS`, with **no `FAIL` entries**. A failed run can leave a validation CSV without updating the statistical CSV; do not mistake an older output for a successful rerun. Script 04 checks that the two CSVs belong to the same completed analysis and, when the RDS is present, checks its source checksum.

## Sample Definition and Estimation

**Adult solo-renter sample (scripts 02–03):** Household residents (`GQ %in% c(1, 2)`), exactly one household member (`NUMPREC == 1`), reference person (`PERNUM == 1`), valid age 18+, one of the 50 states or D.C., cash renter (`OWNERSHPD == 22`), finite positive gross rent (`RENTGRS > 0`), and finite positive household weight (`HHWT > 0`). There are no additional restrictions on sex, marital status, employment, or number of bedrooms.

**Rental affordability:** Monthly gross rent (`RENTGRS`, including utilities/fuel) is multiplied by 12 and divided by valid positive annual pretax personal income (`INCTOT`). Missing, invalid, zero, negative, or ambiguously coded income is **not** used as a denominator in rent-burden estimates. Severe rent burden is a ratio **strictly greater than 50%**; the statistical CSV also includes a **strictly greater than 30%** indicator. Household estimates use `HHWT`.

**Living-alone prevalence:** Figure 3 uses **all persons**, not only adult renters. Its numerator is the `PERWT`-weighted count of household residents with `GQ %in% c(1, 2)`, `NUMPREC == 1`, and `PERNUM == 1`. Its denominator is the `PERWT`-weighted population of the state, **including group-quarters residents**.

**Uncertainty:** Script 03 uses the 80 ACS household replicate weights with successive difference replication for rent-burden proportions and constructs normal-based 95% confidence intervals. Figure 1 shows available severe-burden intervals; Figures 2 and 3 show point estimates. The person-level living-alone shares are not supplied with analogous confidence intervals by this project.

These different denominators and survey weights are essential when comparing the maps: a high living-alone population share does not by itself establish that renting alone is affordable.

## Output Guide

| File | What to inspect |
|---|---|
| `data_processed/single_person_renters_clean.rds` | Filtered one-person renter records, derived income/rent variables, weights, and embedded source metadata. |
| `data_processed/data_validation_report.csv` | Checks, status (`PASS`, `WARN`, `FAIL`, `INFO`), and the completed-analysis marker. |
| `data_processed/statistical_summary.csv` | Long-format estimates by geography, age group, and metric; population measures also appear here. |
| `figures/fig01_severe_burden_intervals.png` | State severe-burden point estimates with available 95% confidence intervals and pooled benchmark. |
| `figures/fig02_severe_rent_burden_map.png` | Severe rent burden among positive-income adult one-person renters by state. |
| `figures/fig03_living_alone_population_share_map.png` | Living-alone share of each state's total population. |

For the statistical CSV, `metric == "burden_share_over50_positive"` and `age_scope == "all_adults"` select the main severe-burden measure; `metric == "living_alone_population_share"` and `age_scope == "all_ages"` select the population measure. Main state rows use `geography == "state"`. Values such as `0.331` represent **33.1%**, not 0.331%. D.C. appears separately and is not plotted with the 50 states.

## Limitations

This analysis is descriptive and limited to 2024 state-level observations. Differences between state estimates do not establish causation or apply to a particular city or available apartment. The two map measures cover different populations; burden rates exclude people without valid positive incomes. Current monthly gross rent is annualized, while income refers to the prior 12 months. The study does not infer after-tax disposable income, savings, or the cost of newly listed rentals. Fresh IPUMS extracts and software versions may lead to small revisions in results.

## Data Citation and Attribution

**IPUMS USA:** Ruggles, Steven, Sarah Flood, Matthew Sobek, Daniel Backman, Grace Cooper, Julia A. Rivera Drew, Stephanie Richards, Renae Rodgers, Jonathan Schroeder, and Kari C. W. Williams. *IPUMS USA: Version 16.0* [dataset]. Minneapolis, MN: IPUMS, 2025. https://doi.org/10.18128/D010.V16.0. See the [IPUMS citation guidance](https://usa.ipums.org/usa/cite.shtml).

**Underlying survey and state boundaries:** U.S. Census Bureau, 2024 ACS one-year data and 2024 cartographic boundary files.

**Required usage review:** [IPUMS USA Terms of Use](https://usa.ipums.org/usa/terms.shtml). Public availability of a data file, if any, does not override these terms.
