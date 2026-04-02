# Agentic Coding for R — Workshops for Ukraine

Workshop materials for a 75–90 minute session on adversarial agentic coding for R, delivered as part of the [Workshops for Ukraine](https://sites.google.com/view/daborisov/main/workshops-for-ukraine) series.

## Authors

Charles Crabtree, Senior Lecturer, School of Social Sciences, Monash University and K-Club Professor, University College, Korea University.

## Overview

This workshop introduces agentic coding for R: using AI assistants that plan, write, run, and revise multi-step analysis workflows while keeping your work transparent and reproducible. The core innovation is **adversarial agentic coding** — pairing a "builder" agent with a separate "reviewer" agent that audits, stress-tests, and improves the code the first agent produced.

## Contents

- **index.html** — Reveal.js slide deck (open in any browser)
- **skills/builder-agent/SKILL.md** — Builder skill template: plans numbered R analysis pipelines
- **skills/reviewer-agent/SKILL.md** — Reviewer skill template: adversarial auditor with Python cross-verification
- **.warp/skills/** — Same skills in Warp's auto-discovery directory
- **images/** — Slide background images

## Viewing the slides

```bash
open index.html
```

Navigate with arrow keys. Press `F` for fullscreen. Timers are clickable.

Live version: [lobsterbush.github.io/workshops-for-ukraine](https://lobsterbush.github.io/workshops-for-ukraine/)

## Using the skills

1. Copy `skills/builder-agent/` and `skills/reviewer-agent/` into your project's `.warp/skills/` directory
2. In Warp, type `/builder-agent` to invoke the builder
3. Switch models, then type `/reviewer-agent` to invoke the reviewer

## Requirements

- A modern web browser to view slides
- [Warp](https://warp.dev) to use the skill templates
- No local install needed (Reveal.js loaded from CDN)

## License

MIT
