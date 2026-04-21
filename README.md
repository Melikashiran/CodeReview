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

### Driver Scripts at Root Instead of Inside `scripts/`
`Caloric_Density_Analysis.R` and `Plots_Manuscript.R` live at the repo root alongside `LICENSE` and `README.md`. They should be inside `scripts/` with numeric prefixes:

```
scripts/
├── 01_caloric_density_analysis.R
├── 02_plots_manuscript.R
└── helpers/
    └── plot_theme.R
```

### No `run_all.R` Entry Point
There is no single script to reproduce the full analysis in order. Add a root-level `run_all.R`:

```r
source("scripts/01_caloric_density_analysis.R")
source("scripts/02_plots_manuscript.R")
```

### No Dependency Pinning (`renv`)
R package versions are not locked. Add `renv::snapshot()` to generate a `renv.lock` file, or at minimum include a `session_info.txt`. This is especially important for `lme4` and `ggplot2`, which have had API changes over time.

### No Inline Documentation of Modelling Decisions
The choice of random effects structure, the family used in `glmer`, and the post-hoc correction method should be documented with inline comments explaining *why*, not just *what*.

---
### The Problem with the Traditional 4-Marker Method (Novak et al.)
The classic testcross counts F2 adults sorted into phenotypic classes and treats those counts as direct readouts of crossover frequency. But this introduces a systematic bias: **not all F2 zygotes survive to be scored**.

> "Classic recombination experiments designed to test genetic and environmental treatments do not directly measure crossing-over, instead rates and distribution of meiotic events in F1 meiocytes is inferred from genetic markers in F2 adults. In *Drosophila melanogaster* females this procedure introduces a substantial 'missing data problem' because 75% of meiotic chromatids segregate to polar body nuclei and another **11% are transmitted to inviable F2 zygotes which cannot be scored** for recombination."

This means that if certain haplotypes are less viable (which the markers themselves can cause), the phenotypic class counts are **distorted** — and what looks like a change in recombination rate could partly be a change in **differential survival** of recombinant vs non-recombinant offspring.

---

### What My Chapter 1 Does Differently — 7 Markers + Viability Correction

My chapter extends the **Cx(Co)m model** (a probabilistic model of crossover patterning) to explicitly account for experimental mortality. Key differences:

| Feature | Novak et al. (4 markers, no viability correction) | Koury et al. (7 markers, viability-corrected) |
|---|---|---|
| **Markers** | 4 X-linked visible markers (y, cv, v, f) | 6 X-linked visible markers + survival measured |
| **Intervals scored** | 3 intervals on the X | full X chromosome map |
| **Survival** | Not directly measured or modelled | Explicitly measured — egg-to-adult viability assay |
| **Statistical model** | Standard GLM/GLMM on crossover counts | Extended Cx(Co)m model with viability parameters |
| **What is inferred** | Crossover frequency (potentially confounded by viability) | Crossover frequency **corrected for differential marker viability** |

The key conceptual advance in my work is that **differential mortality should be the starting assumption, not something to assume away**. This reframes how to interpret any experiment that uses visible markers in *Drosophila* — including Novak et al.

---

### Why This Matters for Interpreting Novak et al.

The Novak et al. results — specifically that DGRP_42 shows a significant 0.5× vs 2× difference in crossover frequency — are interesting biologically, but they were obtained without checking whether differential marker viability varied between dietary treatments. If, say, the low-calorie diet differentially reduces survival of certain phenotypic classes, then the apparent increase in crossover frequency at 0.5× could be partly or wholly a viability artefact rather than a true change in the meiotic crossover rate.

My chapter provides the statistical framework (and the software/model) to test and correct for exactly this. Applying Cx(Co)m viability-corrected approach to a dataset like Novak et al. would be a natural extension and a methodological improvement on the traditional approach the field has relied on.

---
