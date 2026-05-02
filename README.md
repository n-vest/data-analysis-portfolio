# Negative Integer Magnitude Comparison — Adult Data Analysis

**Study:** Dissertation Study 1 (Adult participants)  
**Script:** `MathEdStudy.Rmd`  
**Author:** Nick Vest  
**Last updated:** 2025-06-13

This R Markdown script contains the full analysis pipeline for a reaction time study examining how adults compare the magnitudes of positive, negative, and mixed-sign integer pairs across four presentation conditions. It covers data cleaning, distance-effect analyses, semantic congruence effect (SCE) analyses, and exploratory executive function analyses.

---

## Study Design

Participants judged which of two numbers was greater or lesser across three pair types:

| Pair Type | Description |
|-----------|-------------|
| **Positive** | Both numbers positive (e.g., 3 vs. 7) |
| **Negative** | Both numbers negative (e.g., −2 vs. −8) |
| **Mixed** | One positive, one negative (e.g., −3 vs. 5) |

Each pair type was presented in four conditions formed by crossing two factors:

| Factor | Levels |
|--------|--------|
| **Presentation** | Sequential, Simultaneous |
| **Order** | Intermixed (mixed trial types), Blocked (same trial type per block) |

Outcomes are reaction time (RT) and classification of individual-level distance and semantic congruence effect profiles.

---

## Data Files

| File | Description |
|------|-------------|
| `all_data.csv` | Trial-level RT data, processed from raw JSON via `dataupload.R` |
| `EFData_20250628_adults.csv` | Participant-level executive function scores (working memory, inhibitory control, cognitive flexibility) |

Both files are expected at the absolute paths currently hardcoded in the script. Update these paths if running on a different machine.

---

## Pipeline Overview

### 1. Data Cleaning

Starting from 35,520 raw trials:

1. **Error-rate exclusion** — remove participants with >45% errors (6 participants excluded by ID)
2. **Accuracy filter** — remove incorrect trials (reduces to ~28,219 trials)
3. **Global RT cutoffs by condition:**
   - Sequential conditions: 50–3,000 ms
   - Simultaneous conditions: 200–3,000 ms
   - Removes ~4.4% of remaining trials
4. **Per-participant SD trimming** — ±3 SD within each SubID × Comparison_Type × condition cell; removes ~2.1%
5. **Final N:** ~26,400 trials

**Contrast coding applied globally:**
- `Order`: Intermixed = 0.5, Blocked = −0.5
- `Presentation`: Sequential = −0.5, Simultaneous = 0.5
- `Predicate_C`: Greater = −0.5, Lesser = 0.5
- `Distance_C`: centered within each Comparison_Type
- `Distance_01`: Near/Far binary (threshold varies by pair type)

---

### 2. Distance Effect Analyses

**Descriptives:** Mean RT by Comparison_Type, Near/Far, and condition. RT distribution plots (raw and log-transformed).

**Group-level LMERs** (`lme4`/`lmerTest`):

```
RT ~ Distance_C × Order × Presentation + Predicate_C + (random effects | SubID)
```

Run separately for Positive, Negative, and Mixed pairs. Mixed-pair models also include four re-parametrized simple-effects models (one per condition as reference cell) to decompose the three-way interaction.

**Individual-level analysis:**

Per-participant OLS: `RT ~ Distance_C + Predicate_C`. Extracts the Distance_C beta and classifies each participant using a ±0.25 SD threshold:

| Classification | Criterion |
|----------------|-----------|
| **Standard** | beta < −0.25 SD (faster for far pairs) |
| **Inverse** | beta > +0.25 SD (faster for near pairs) |
| **None** | beta within ±0.25 SD |

Proportion plots show classification distributions by Comparison_Type and condition.

---

### 3. Semantic Congruence Effect (SCE) Analyses

Restricted to Positive and Negative pairs (Size is undefined for Mixed pairs).

The SCE captures the interaction between predicate (choose greater vs. lesser) and magnitude size (large vs. small number) on RT.

**Group-level LMERs:**

```
RT ~ Size_C × Predicate_C × Order × Presentation + (random effects | SubID)
```

Run for Positive and Negative pairs, with additional simple-effects models for Negative pairs (same four reference-cell re-parametrizations as the distance models).

**Individual-level analysis:**

Per-participant OLS: `RT ~ Predicate_C:Size_C`. Extracts the interaction beta and classifies using the same ±0.25 SD threshold:

| Classification | Criterion |
|----------------|-----------|
| **Standard** | beta > +0.25 SD (congruence facilitates RT) |
| **Inverse** | beta < −0.25 SD (congruence slows RT) |
| **None** | within ±0.25 SD |

Proportion plots show distributions for Positive and Negative pairs overall, and for Negative pairs broken down by condition.

---

### 4. Exploratory Executive Function Analyses

Uses participant-level EF scores linked to individual-level distance-effect classifications for Mixed pairs. Each EF measure is split at the median.

| EF Measure | Operationalization |
|------------|-------------------|
| **Working memory** | Mean WM score; higher = better |
| **Inhibitory control** | Mean RT on incongruent trials; higher RT = lower IC |
| **Cognitive flexibility** | Mean switch cost; higher switch cost = lower CF |

For each measure: cross-tabulation with DE profile (Extension / Reflection / Components), chi-square test, row proportions, and proportion bar plot.

An additional analysis examines cognitive flexibility × representation change (binary: whether a participant's distance-effect profile shifted across conditions).

---

## Custom Functions

| Function | Description |
|----------|-------------|
| `varDescribe(Data, Detail, Digits)` | Compact wrapper around `psych::describe`; Detail level 1–3 controls output width |
| `varDescribeBy(Data, IVList)` | `varDescribe` split by a grouping variable |
| `varRecode(Var, Old, New)` | Recodes a vector by position-matched replacement; preserves NAs and warns on mismatch |
| `extract_de_betas(df)` | Per-participant OLS for distance effects; returns beta, p-value, 95% CI |
| `classify_de(df)` | Classifies distance-effect profiles using ±0.25 SD threshold |
| `extract_sce_betas(df)` | Per-participant OLS for SCE; returns beta, p-value, 95% CI |
| `classify_sce(df)` | Classifies SCE profiles using ±0.25 SD threshold |

---

## Dependencies

Install all packages before knitting:

```r
install.packages(c(
  "psych", "car", "tidyverse", "effectsize", "ggplot2", "dplyr",
  "effects", "see", "ggrain", "lme4", "lmerTest", "emmeans", "ggpattern"
))
```

---

## Rendering

```r
rmarkdown::render("MathEdStudy.Rmd")
```

Output: `MathEdStudy.html`

---

## Notes

- Data file paths are absolute and machine-specific. Update before running on a new system.
- The `bobyqa` and `nloptwrap` optimizers with high `maxfun` limits are set deliberately to help complex random-effects structures converge. Do not reduce these without checking convergence warnings.
- Individual-level classification uses ±0.25 SD of the sample's beta distribution, not a fixed threshold. Classifications are relative to the sample and will shift if the sample composition changes.
- The EF proportion plots use hardcoded proportion values rather than computing them from `prop.table()` output. Verify these match the chi-square output before reporting.
