---
name: builder-agent
description: Plan, write, run, and revise multi-step R analysis workflows. Use when starting a new analysis, re-analyzing data, or building a replication package. Produces numbered .R scripts, publication-ready figures, LaTeX macros, and documentation.
---

# Builder Agent — R Analysis Workflow

Build a complete, reproducible R analysis pipeline from a research task description.

## When to use

Invoke this skill when you need to:

- Analyze a dataset from scratch
- Re-analyze forgotten or inherited data
- Build a replication package
- Generate publication-ready figures and tables
- Produce LaTeX-ready statistics

## Procedure

Follow these steps in order. Do not skip steps.

### 1. Understand the project

- Read WARP.md, AGENTS.md, and README.md if they exist
- Scan for existing scripts, data files, and codebooks
- Identify the data format, size, and key variables
- Ask clarifying questions only if the research question is ambiguous

### 2. Plan the pipeline

Create a numbered script plan before writing any code:

```
01_clean_data.R    — Load raw data, clean, recode, write analysis-ready RDS
02_eda.R           — Descriptive statistics, missingness, distributions
03_analysis.R      — Main models, extract coefficients, write statistics.tex
04_figures.R       — Publication-ready plots, export PDF + PNG
05_robustness.R    — Alternative specifications, sensitivity checks
```

Announce the plan to the user before proceeding. Adjust numbering and names to fit the project.

### 3. Write each script

Every `.R` script must follow these conventions:

**File paths:**
- Use `here::here()` for all paths. Never use absolute paths.
- Load `library(here)` at the top of every script.

**Naming:**
- snake_case for all file names and variable names
- Sequential numbering: `01_`, `02_`, `03_`, etc.

**Header:**
- Every script starts with a comment block:

```r
# ==============================================================================
# 01_clean_data.R
# Purpose: Load and clean raw survey data
# Input:   data/raw/survey_2024.csv
# Output:  data/processed/survey_clean.rds
# Author:  [user] + AI agent
# ==============================================================================
```

**Packages:**
- Load all packages at the top with `library()`
- Use `estimatr::lm_robust()` for OLS with robust standard errors
- Use `estimatr::lm_robust(..., clusters = cluster_var)` for clustered SEs
- Default to robust (HC2) standard errors unless data has repeated observations per unit, in which case cluster by unit
- Use `ggplot2` + `ggthemes::theme_tufte()` for all plots
- Use `modelsummary` or `texreg` for regression tables

**Statistical output:**
- Write all key statistics to `statistics.tex` as LaTeX macros:

```r
sink(here("output", "statistics.tex"))
cat(sprintf("\\newcommand{\\nObs}{%s}\n", format(nobs, big.mark = ",")))
cat(sprintf("\\newcommand{\\mainCoef}{%.3f}\n", coef_main))
cat(sprintf("\\newcommand{\\mainSE}{%.3f}\n", se_main))
cat(sprintf("\\newcommand{\\mainP}{%.3f}\n", p_main))
sink()
```

**Figures:**
- Export all figures as PDF via `ggsave()` using `cairo_pdf` device
- Width: 6.5 inches for single-column, 13 inches for full-width
- Minimum 300 DPI for PNG backup copies
- Always specify explicit width and height
- Labels must be large enough to read in print

```r
ggsave(here("figures", "main_effect.pdf"),
       plot = p, device = cairo_pdf, width = 6.5, height = 4.5)
ggsave(here("figures", "main_effect.png"),
       plot = p, width = 6.5, height = 4.5, dpi = 300)
```

### 4. Run and verify

- Run each script in order: `Rscript 01_clean_data.R`, then `02_eda.R`, etc.
- Check for errors and warnings. Fix iteratively.
- Confirm output files exist after each script completes.
- If a script fails, read the error, fix the code, and re-run. Do not move on until it succeeds.

### 5. Generate documentation

- Create or update `README.md` with:
  - Project title and description
  - Data sources
  - Requirements (R version, packages)
  - Replication instructions: "Run scripts in numbered order"
  - Output file descriptions

- Create or update `WARP.md` with project context for future sessions.

### 6. Self-check before finishing

Verify all of the following before reporting completion:

- [ ] All scripts run without errors in numbered order
- [ ] No absolute paths anywhere in any script
- [ ] All figures exported as PDF (cairo_pdf) and PNG (300 DPI)
- [ ] `statistics.tex` contains macros for every reported number
- [ ] Standard errors are robust (or clustered where appropriate)
- [ ] `set.seed()` is called before any randomness
- [ ] README.md exists with replication instructions
- [ ] All `library()` calls are at the top of each script
- [ ] Variable names are snake_case throughout

## What to avoid

- Never use `setwd()`. Use `here::here()` instead.
- Never hardcode statistics in LaTeX. Always compute and export via macros.
- Never use default `lm()` standard errors. Use `estimatr::lm_robust()`.
- Never suppress warnings silently. Investigate and resolve them.
- Never commit data files larger than 100MB to git.
