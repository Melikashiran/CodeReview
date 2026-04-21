# CodeReview
## [paper link](https://link.springer.com/article/10.1007/s10709-025-00255-2)
## [Github link](https://github.com/StevisonLab/Diet-and-recombination-in-Dmel)  
> **Language:** R (100%)

---

## 1. What the Novak et al. Paper Actually Measured
### Genetic Stocks
The study used **two inbred lines from the Drosophila Genetic Reference Panel (DGRP)**:

| Stock | Starvation Sensitivity | Recombination Plasticity Found? |
|---|---|---|
| **DGRP_42** | High (more sensitive to starvation stress) | Yes — significant differences between 0.5× and 2× diet |
| **DGRP_217** | Low (more resistant) | No significant plasticity |

These were chosen deliberately to contrast genotypes with known differences in stress response, allowing the study to investigate whether genetic background modulates dietary effects on recombination.

---

### The 4 Phenotypic Marker System (Traditional Method)

The paper used the **classic *D. melanogaster* phenotypic marker approach** — the same method the Stevison Lab has relied on across multiple prior studies. Recombination was measured across the **X chromosome** using **4 visible recessive markers** that define 3 intervals:

| Marker | Gene Symbol | Phenotype | Chromosomal Position (approx.) |
|---|---|---|---|
| **yellow** | *y* | Yellow body colour | 0.0 cM — far left of X (telomere-proximal) |
| **vermilion** | *v* | Bright red eyes | 33.0 cM |
| **crossveinless** | *cv* | Missing crossveins on wing | 13.7 cM |
| **forked** | *f* | Short, bent bristles | 56.7 cM |

### Dietary Treatments Applied

Females were raised on **three caloric density diets** before and during mating:

| Treatment | Diet | Calories Relative to Standard |
|---|---|---|
| Low calorie | 0.5× | Half the standard |
| Standard | 1× | Control |
| High calorie | 2× | Double the standard |

Recombination rates were compared across these three diets for each DGRP strain.

---

## 2. What the Scripts Do 
### `Caloric_Density_Analysis.R` *(main analysis driver)*

- Loads raw F2 phenotypic class counts from `rawdata/` — these are the scored offspring from each dietary treatment × DGRP strain combination.
- Calculates **crossover frequency per interval** from the 4-marker X chromosome system (3 intervals: *y–cv*, *cv–v*, *v–f*).
- Runs **mixed-model statistics** (`glmer` from `lme4`) to test for effects of diet and genetic background on recombination rate, with vial as a random effect.
- Performs **post-hoc pairwise comparisons** (0.5× vs 1×, 1× vs 2×, 0.5× vs 2×) — the significant result was 0.5× vs 2× in DGRP_42 only.
- Analyses physiological traits: body mass, oocyte count, testis length, TUNEL assay DNA damage scores.
- Outputs summary tables to `output/`.

### `Plots_Manuscript.R` *(publication figure generator)*

- Reads the processed results from `output/` or directly from `rawdata/`.
- Generates all manuscript figures: recombination rate bar/dot plots per diet × strain, physiology boxplots, TUNEL visualisations, DEG summary.
- Exports figures to `images/` in publication-ready format (PDF or PNG).

### `scripts/` folder *(helper scripts)*

- Data wrangling and cleaning of raw phenotypic count files.
- Possibly interval-specific recombination calculations as helper functions.
- RNA-seq import / DEG count summarisation scripts.
---

## 3. Positives points of the Github
### Separate Analysis and Plotting Scripts
Having `Caloric_Density_Analysis.R` and `Plots_Manuscript.R` as two distinct R scripts is a genuine best practice. It separates statistical inference from figure generation, so plots can be adjusted without re-running models.

### Three-Folder Data Hierarchy
`rawdata/` → `output/` → `images/` creates a clear, reproducible data flow that someone cloning the repo can immediately navigate.

### DOI + Publication Release
Minting a Zenodo snapshot (`10.5281/zenodo.18090408`) at the time of publication is best practice for scientific reproducibility and citability.

### MIT License and Full README Abstract
The README is immediately useful to anyone finding the repo through a literature search.

---

## 4. Areas for Improvement (Code Organisation)

### 4.1 Driver Scripts at Root Instead of Inside `scripts/`
`Caloric_Density_Analysis.R` and `Plots_Manuscript.R` live at the repo root alongside `LICENSE` and `README.md`. They should be inside `scripts/` with numeric prefixes:

```
scripts/
├── 01_caloric_density_analysis.R
├── 02_plots_manuscript.R
└── helpers/
    └── plot_theme.R
```

### 4.2 No `run_all.R` Entry Point
There is no single script to reproduce the full analysis in order. Add a root-level `run_all.R`:

```r
source("scripts/01_caloric_density_analysis.R")
source("scripts/02_plots_manuscript.R")
```

### 4.3 No Dependency Pinning (`renv`)
R package versions are not locked. Add `renv::snapshot()` to generate a `renv.lock` file, or at minimum include a `session_info.txt`. This is especially important for `lme4` and `ggplot2`, which have had API changes over time.

### 4.4 No Inline Documentation of Modelling Decisions
The choice of random effects structure, the family used in `glmer`, and the post-hoc correction method should be documented with inline comments explaining *why*, not just *what*.

---

