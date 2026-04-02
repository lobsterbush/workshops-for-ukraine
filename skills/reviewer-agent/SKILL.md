---
name: reviewer-agent
description: Audit R analysis code produced by another agent or human. Checks that scripts run, numbers match, and figures are consistent with tables. Use after a builder agent has produced analysis scripts.
---

# Reviewer Agent — Code Audit

Verify systematically. Your job is to find errors the builder may have missed.

## Procedure

### 1. Inventory

- List all scripts in execution order (by numeric prefix)
- List input data files and expected output files
- Read README.md and WARP.md for stated intentions
- Note any statistics files, figures, or tables the pipeline claims to produce

### 2. Run everything from a clean session

Run every script from scratch. Do not carry over objects between scripts.

Record for each script:
- Did it run without errors?
- Were there warnings? What did they say?
- What output files were created?

### 3. Core checks (always do these)

These are the highest-value checks. Do all of them.

**Do the numbers match?**
- Count the rows in the actual data file. Compare to the N reported in any statistics file, table, or manuscript text. If they differ, explain why.
- Pick the main coefficient from the regression output. Does it match what's reported in the statistics file and in any figures?

**Do the figures match the tables?**
- If there's a coefficient plot and a regression table, do they show the same estimates? Agents sometimes re-run models with slightly different samples for different outputs.

**Are there absolute paths?**
- Search all scripts for `/Users/`, `/home/`, `C:\`. These break on other machines.

**Are statistics computed or hardcoded?**
- Check whether numbers in any statistics file or manuscript text are computed from data or typed by hand. Hardcoded numbers are a critical error.

**Are dropped observations documented?**
- If the cleaning script filters or removes rows, does it report how many were dropped and why? Silent drops are the most common source of N mismatches.

### 4. Extended checks (if time permits)

These are valuable but not always feasible in a single session.

- **Independent re-estimation:** Pick one key result and re-estimate it using a different tool or language (e.g., Python's statsmodels if the original was in R). If R and the other tool agree, the result is more trustworthy.
- **Reproducibility:** Run the full pipeline twice. Diff the outputs. If anything changed, there's a missing random seed or a non-deterministic step.
- **Undocumented drops:** Compare row counts at each stage of the pipeline to identify where observations disappear.

### 5. Write a review report

Produce `review_report.md` organized by severity:

**Critical** — Results are wrong or misleading. Must fix. Examples: reported N doesn't match data, coefficient in figure doesn't match table, fabricated citation.

**Warning** — Results may be fragile or unclear. Should fix. Examples: undocumented dropped observations, missing random seed, deprecated function.

**Note** — Style or documentation issues. Nice to fix. Examples: inconsistent naming, missing comments, figure could be clearer.

For each finding, state: which file, what the problem is, how you found it, and how to fix it.

## What to avoid

- Do not rewrite the code from scratch. Audit what exists.
- Do not report only style issues. Prioritize correctness.
- Do not skip the core checks. They catch the most important errors.
