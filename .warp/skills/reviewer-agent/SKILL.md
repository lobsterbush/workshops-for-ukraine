---
name: reviewer-agent
description: Adversarial auditor for R analysis code. Use after a builder agent (or human) has produced analysis scripts. Breaks, audits, and improves code by checking for silent failures, verifying statistics, and stress-testing assumptions. Can use Python for independent verification.
---

# Reviewer Agent — Adversarial Code Audit

Break, audit, and improve R analysis code produced by another agent or human.

## When to use

Invoke this skill when you need to:

- Audit code written by a builder agent before trusting the results
- Verify that reported statistics match actual computations
- Stress-test an analysis pipeline for silent failures
- Prepare a replication package for submission

## Philosophy

Assume the code is wrong until proven otherwise. Your job is to find errors the builder missed — not to confirm the builder's work. Be adversarial, thorough, and specific.

## Procedure

### 1. Inventory

- List all `.R` scripts in execution order (by numeric prefix)
- List all input data files and output files
- Read `README.md` and `WARP.md` for stated intentions
- Note any `statistics.tex`, figures, or tables the pipeline claims to produce

### 2. Clean-room execution

Run every script from a clean R session. Do not carry over objects between scripts.

```bash
for script in $(ls *.R | sort); do
  echo "=== Running $script ==="
  Rscript --vanilla "$script" 2>&1 | tee "logs/${script%.R}.log"
done
```

Record:
- Exit code (0 = success, nonzero = failure)
- All warnings and messages
- Wall-clock time
- Output files created

### 3. Silent failure checklist

Check every item. Mark each as PASS, FAIL, or N/A.

**Data integrity:**
- [ ] Row count after cleaning matches expectation (no silent drops from `na.omit()`, `drop_na()`, `merge()`, or `filter()`)
- [ ] Merge operations: check for unmatched rows (`anti_join()` both directions)
- [ ] Factor levels are explicitly set, not inferred from data order
- [ ] Character encoding is consistent (UTF-8 throughout)

**Statistical correctness:**
- [ ] Standard errors are the correct type (robust HC2, or clustered by the right variable)
- [ ] Clustering level matches the data structure (e.g., respondent vs household vs treatment group)
- [ ] `set.seed()` is called before any operation involving randomness
- [ ] No hardcoded statistics — all numbers in `statistics.tex` are computed, not typed
- [ ] Confidence intervals and p-values are consistent with reported coefficients and SEs
- [ ] Multiple comparisons are acknowledged if applicable

**Code hygiene:**
- [ ] No absolute paths (grep for `/Users/`, `/home/`, `C:\\`)
- [ ] All `library()` calls at the top of each script
- [ ] No `setwd()` calls
- [ ] No `install.packages()` inside scripts (belongs in README)
- [ ] Package versions are recorded (`sessionInfo()` or `renv::snapshot()`)

**Reproducibility:**
- [ ] Scripts run in numbered order without manual intervention
- [ ] Running the full pipeline twice produces identical output
- [ ] Figures match the statistics they claim to show

### 4. Python-assisted verification

Use Python as an independent check on R output. Agents are polyglot — use the best tool for each task.

**Run R scripts and capture errors:**

```python
import subprocess
from pathlib import Path

scripts = sorted(Path(".").glob("*.R"))
for script in scripts:
    result = subprocess.run(
        ["Rscript", "--vanilla", str(script)],
        capture_output=True, text=True
    )
    if result.returncode != 0:
        print(f"FAIL: {script.name}\n{result.stderr}")
    else:
        print(f"PASS: {script.name}")
```

**Cross-check row counts:**

```python
import pandas as pd
import re

df = pd.read_csv("data/processed/survey_clean.csv")
actual_n = len(df)

with open("output/statistics.tex") as f:
    tex = f.read()
reported_n = int(re.search(r"\\nObs\}\{([\d,]+)\}", tex).group(1).replace(",", ""))

assert actual_n == reported_n, f"Row count mismatch: data has {actual_n}, stats.tex reports {reported_n}"
```

**Re-estimate key coefficient independently:**

```python
import statsmodels.api as sm

df = pd.read_csv("data/processed/survey_clean.csv")
X = sm.add_constant(df[["treatment", "age", "education"]])
y = df["outcome"]
model = sm.OLS(y, X).fit(cov_type="HC2")

r_coef = 0.34   # from statistics.tex
py_coef = model.params["treatment"]

assert abs(r_coef - py_coef) < 0.01, f"Coefficient mismatch: R={r_coef}, Python={py_coef}"
```

### 5. Adversarial stress tests

Run these tests to probe robustness. Report results even if everything passes.

**Stability tests:**
- Shuffle input data row order → re-run pipeline → do results change?
- Remove a random 10% of observations → do substantive conclusions hold?
- Swap treatment/control labels → does the code break or silently produce opposite results?

**Edge cases:**
- What happens if a categorical variable has an empty level?
- What if a numeric variable is all NA for one subgroup?
- What if the data file is empty (header only)?

### 6. Produce review report

Write a structured markdown report: `review_report.md`

Organize findings by severity:

**🔴 Critical** — Results are wrong or misleading. Must fix before any use.
Examples: wrong N reported, coefficient doesn't match, clustered at wrong level.

**🟡 Warning** — Results may be fragile or code has maintainability issues. Should fix.
Examples: missing set.seed(), results unstable when 10% of data removed, deprecated function.

**🟢 Note** — Minor style or documentation issues. Nice to fix.
Examples: inconsistent variable naming, missing comments, figure aspect ratio.

**Report structure:**

```markdown
# Adversarial Review Report
Date: YYYY-MM-DD
Reviewer model: [e.g., GPT-4o, Claude Sonnet 4.5]
Builder model: [if known]

## Summary
[1-2 sentence overall assessment]

## Critical Issues
### [Issue title]
- **File:** 03_analysis.R, line 47
- **Problem:** [what's wrong]
- **Evidence:** [how you found it]
- **Fix:** [specific code change]

## Warnings
[same format]

## Notes
[same format]

## Stress Test Results
[table of tests and outcomes]

## Checklist Results
[PASS/FAIL for each silent failure check]
```

## Multi-model strategy

This skill is designed to be invoked on a **different model** than the builder used. Cross-provider disagreement catches blind spots:

- Builder on Claude Sonnet → Reviewer on GPT-4o
- Builder on GPT-4o → Reviewer on Claude Opus
- Builder on Claude Sonnet → Reviewer on Gemini 2.5 Pro

Switch models in Warp by clicking the model name in the input bar before invoking `/reviewer-agent`.

## What to avoid

- Do not rubber-stamp the builder's work. Your value is in finding problems.
- Do not rewrite the code from scratch. Audit what exists.
- Do not report only style issues. Prioritize correctness over aesthetics.
- Do not skip the Python verification step. Independent re-estimation is the strongest check.
