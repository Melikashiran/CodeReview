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
# Code Review: Script 05 — Recombination Rate Analysis

---

## What the Script Does

**1. Kosambi Map Function**
Converts raw recombination fractions to centiMorgans (cM) to correct for double crossovers that would otherwise make intervals look shorter than they are. Applied per-interval before summing to a total map length.

**2. Crossover Count Aggregation**
Takes the raw haplotype-scored data (`by_vialday`) and counts non-crossover (NCO), single (SCO), double (DCO), and triple (TCO) crossover individuals per vial-day. Computes total crossover *events* as `sco + 2×dco + 3×tco`, then calculates a raw recombination rate per vial. Rows with missing values are dropped via `na.omit()`.

**3. Per-Interval Validation**
Breaks crossovers into the 3 X-chromosome intervals (y–cv, cv–v, v–f) and computes dataset-wide observed recombination fractions, then compares them to expected genetic distances (13.7, 19.3, 23.7 cM) as a sanity check. Applies Kosambi correction to observed values and stores the comparison in `recomb_intervals`. Sources `scripts/05b.COI.R` to calculate crossover interference (COI).

**4. Data Aggregation (Raw then Kosambi-corrected)**
Collapses per-day rows to the vial level using `summaryBy()`, summing all crossover counts within each `MaternalVial × vial_letter × PaternalStock × Treatment` combination. Filters out vials with fewer than 5 scored males. Does this twice — once without correction (for comparison) and once applying Kosambi to each interval before summing — producing the corrected `recomb_rate` object used for all downstream stats.

**5. Full-Interval GLMM**
Fits a binomial GLMM (`glmer`) with crossover successes/failures as the response, `MaternalVial` as a random effect, and `Treatment × PaternalStock × vial_letter` as the three-way fixed-effects interaction. Uses `optimx/nlminb` optimizer. Tests overall significance with `Anova()` (Type II chi-square) and saves results to `output/`.

**6. Post-Hoc Contrasts and Odds Ratio Plots**
Runs pairwise `emmeans` contrasts between diet treatments (0.5x vs 1x, 0.5x vs 2x, 1x vs 2x) separately within each `PaternalStock × vial_letter` combination. Converts log-odds estimates to odds ratios via `exp()`. Produces one odds ratio line plot per strain (DGRP_42 and DGRP_217) with significance stars, saved as PDFs. This block is repeated for the full interval and for each of the 3 individual intervals (8 plots total).

**7. Per-Interval GLMMs (y–cv, cv–v, v–f)**
Repeats the same GLMM and post-hoc structure from Block 5–6 three more times, once per marker interval, to test whether the diet effect is chromosome-wide or concentrated in a specific region. Each model saves an ANOVA table and post-hoc CSV to `output/`.

**8. Paper Summary Table and Manual Differences**
Aggregates Kosambi-corrected means by `Treatment × PaternalStock` and computes the absolute cM difference between diet pairs for each strain. Reports these as hardcoded arithmetic (e.g., `55.23938 - 51.92768`) for inclusion in the manuscript table.

---

## ✅ What Is Done Well

- Correct random effect structure (`1 | MaternalVial`) accounts for non-independence of offspring from the same F1 mother
- Kosambi correction applied per-interval before summing — biologically correct order
- Both strains analysed separately, allowing the genotype × diet interaction to emerge
- Results consistently written to `output/` as CSVs for reproducibility
- COI analysis factored into its own sourced script (`05b.COI.R`)

---

## ⚠️ What Could Be Better

| Issue | Problem |
|-------|---------|
| **8× copy-pasted odds ratio plot** | The `ggplot` odds ratio block is copy-pasted for every interval × strain combination. Should be a single reusable function |
| **Hardcoded subtracted values** | Numbers like `55.23938 - 51.92768` are manually typed results — if data changes they go silently wrong |
| **`_new` suffix in output filename** | `recomb-stats_anova-ycv_new.csv` is a development artifact that was never cleaned up |

## Personal Note — Data Reuse and Measurement Level Mismatch

I used this dataset for some of my own downstream analyses, but I encountered a key structural issue: fertility/fecundity was recorded at the MaternalVial level, while recombination was measured at the individual vial ID level. These are not the same unit of observation (a single maternal vial can contain multiple scored vials across different brooding periods).
Because of this mismatch, when I wanted to combine recombination and fecundity data in the same analysis, I had to fork the original dataset and recalculate fecundity at the vial level to match the recombination measurement unit. This aggregation step is not present in the original script, which means anyone else trying to do the same combined analysis would hit the same problem without realising it.
This is worth documenting as a limitation of the original data structure.
---
*Based on actual script code. April 2026.*
