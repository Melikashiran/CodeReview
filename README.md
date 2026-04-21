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
| **DGRP_42** | High (more sensitive to starvation stress) | ✅ Yes — significant differences between 0.5× and 2× diet |
| **DGRP_217** | Low (more resistant) | ❌ No significant plasticity |

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

