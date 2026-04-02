# Example Conventions File

Copy this file to your project root (or into WARP.md) and customize it. The builder agent reads these conventions and follows them.

## R Conventions

These are examples — replace with your own preferences.

**Standard errors:**
- Use robust (HC2) standard errors by default: `estimatr::lm_robust()`
- Cluster by respondent when there are repeated observations per person

**Plots:**
- Use `ggplot2` with `ggthemes::theme_tufte()`
- Base font size: 14
- Export as PDF via `ggsave()` with `cairo_pdf` device, width 6.5 inches
- Also export PNG at 300 DPI as backup

**Tables:**
- Use `modelsummary` for regression tables

**Statistics export:**
- Write LaTeX macros to `output/statistics.tex`
- Use `cat(..., file = here("output", "statistics.tex"))` pattern

**File paths:**
- Always use `here::here()` for relative paths

**Script naming:**
- Sequential numbering: `01_`, `02_`, `03_`
- snake_case throughout

## Adapt for your own workflow

You might prefer:
- `fixest` instead of `estimatr`
- `ggplot2` with a custom theme instead of `theme_tufte()`
- `stargazer` or `texreg` instead of `modelsummary`
- CSV or JSON instead of LaTeX macros
- Stata `.do` files instead of R scripts

The builder skill doesn't care which tools you use. It cares that you have conventions and follow them consistently.
