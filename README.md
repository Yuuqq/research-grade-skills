# Research-Grade Skills

[![validate](https://github.com/Yuuqq/research-grade-skills/actions/workflows/validate.yml/badge.svg)](https://github.com/Yuuqq/research-grade-skills/actions/workflows/validate.yml)
[![Skills](https://img.shields.io/badge/skills-69-brightgreen.svg)](https://yuuqq.github.io/research-grade-skills/)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE.md)

**CI-validated Agent Skills for science.** 69 skills covering scientific writing, machine learning, data analysis, quantum computing, and research databases — every one structurally verified on every commit.

Works with **Claude Code**, **Cursor**, **Codex**, and any client that supports the open [Agent Skills](https://agentskills.io/) standard.

**[Browse the interactive catalog](https://yuuqq.github.io/research-grade-skills/)**

---

## Why This Exists

Researchers cannot afford silent errors. One wrong statistical method, one invented API, one outdated citation — and hours of work (or a paper) are compromised. Skill libraries optimized for size cannot make that guarantee; a library a researcher actually relies on has to earn it.

This repository is an independently maintained, heavily curated descendant of [K-Dense's scientific skills](https://github.com/K-Dense-AI/scientific-agent-skills). Compared to where it started:

- **All vendor promotion removed.** Upstream, every skill ended with a section steering the agent to advertise a commercial platform, plus one skill whose only job was self-promotion ("ALWAYS run this skill with every session"). All of it is gone, and CI now rejects any skill that tries to reintroduce vendor steering.
- **A real quality gate.** Every push and pull request validates all 69 skills: frontmatter schema, skill-name/directory consistency, registry-to-disk consistency, relative link resolution, and Python script syntax. A weekly job audits all external links for rot. None of this existed before.
- **Bugs actually fixed.** The validation gate already caught and fixed broken reference links, mismatched skill names, and an undefined-variable bug in a bundled script — the kind of defects that silently degrade agent behavior.

The goal is simple: **any skill you load from here should be safe to use in real research output.**

## Install

### Claude Code (plugin)

```
/plugin marketplace add Yuuqq/research-grade-skills
/plugin install scientific-skills@research-grade-skills
```

### Cursor, Codex, and other Agent Skills clients

Clone the repository and copy the skills you need into your client's skills directory:

```bash
git clone https://github.com/Yuuqq/research-grade-skills.git
# Claude Code:  ~/.claude/skills/
# Cursor:       ~/.cursor/skills/  (or .cursor/skills/ in a project)
cp -r research-grade-skills/scientific-skills/literature-review ~/.cursor/skills/
```

Each skill is a self-contained folder — `SKILL.md` plus optional `references/`, `scripts/`, and `assets/`.

## What's Inside

| Category | Skills |
|---|---|
| Scientific Communication (16) | literature review, peer review, scientific writing, hypothesis generation, citation management, research grants, scholar evaluation, schematics, slides, posters, venue templates |
| Machine Learning (10) | scikit-learn, PyTorch Lightning, Transformers, SHAP, Stable Baselines3, PufferLib, Torch Geometric, UMAP-learn, aeon, scikit-survival |
| Data Analysis (8) | Polars, Dask, Vaex, NetworkX, GeoPandas, Data Commons, exploratory data analysis, statistical analysis |
| Visualization (6) | Matplotlib, Seaborn, Plotly, scientific visualization, image generation, schematics |
| Research Tools (6) | computational social science, general data science, resource detection, market research, MATLAB, Perplexity search |
| Document Processing (5) | DOCX, PDF, PPTX, XLSX, MarkItDown |
| Databases (4) | OpenAlex, PubMed, bioRxiv, USPTO |
| Quantum Computing (4) | Qiskit, Cirq, PennyLane, QuTiP |
| Materials & Chemistry (4) | Pymatgen, PyMC, PyMOO, FluidSim |
| Physics & Math (3) | Astropy, SymPy, statsmodels |
| Simulation & Engineering (3) | SimPy, Modal, Denario |

Full list with descriptions: **[interactive catalog](https://yuuqq.github.io/research-grade-skills/)**

## Quick Examples

Literature review with a PRISMA diagram:

```
Conduct a systematic literature review on CRISPR delivery mechanisms using PubMed and bioRxiv.
Include a PRISMA flow diagram using scientific-schematics.
```

Publication-quality analysis:

```
Load my CSV dataset, run exploratory data analysis with statistical tests,
and create publication-quality visualizations with matplotlib and seaborn.
```

Explainable ML:

```
Train a classification model on my dataset using scikit-learn,
explain predictions with SHAP, and generate a comprehensive report.
```

## The Quality Gate

What CI enforces today, on every commit ([validate.yml](.github/workflows/validate.yml)):

- SKILL.md present, frontmatter parses, `name`/`description`/`license` required
- Frontmatter name matches directory name (what the agent loads is what's registered)
- Every relative link resolves to a real file
- Every bundled Python script passes syntax checks and error-level lint
- Registry (`marketplace.json`) and disk agree 1:1 — no phantom skills
- Zero vendor-promotion content
- Weekly external-link rot audit ([links.yml](.github/workflows/links.yml))

In progress — a deeper editorial audit of all 69 skills across four dimensions: academic rigor, real-world scenario coverage, discoverability of descriptions, and runnability of examples. Findings and fixes land as regular releases.

[REVIEW.md](REVIEW.md) is an advisory snapshot of known content and script issues. It is not a CI gate and it is not project policy. The inventory notes in `.planning/codebase/CONCERNS.md` (2026-04-30) are stale where they disagree with this file or with `validate.yml`.

## Contributing

Factual fixes are the most valuable contribution: wrong API names, outdated signatures, dead links, missing pitfalls. See [CONTRIBUTING.md](CONTRIBUTING.md) — the validator tells you before CI does:

```bash
pip install pyyaml
python scripts/validate_skills.py
```

## License & Attribution

MIT. Original skill content copyright (c) 2025 [K-Dense Inc.](https://github.com/K-Dense-AI/scientific-agent-skills), used and modified under the MIT license; individual skills may carry their own licenses in their `SKILL.md` frontmatter. This project is not affiliated with or endorsed by K-Dense Inc.
