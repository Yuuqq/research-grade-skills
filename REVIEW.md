# Repository Code Review: research-grade-skills

*Advisory snapshot, updated 2026-10-16. Not a CI gate and not project policy.*

Methodology: static reading of `SKILL.md` files, `scripts/validate_skills.py`, `.github/workflows/validate.yml`, `JULES.md`, `README.md`, and the bundled Python under `scientific-skills/`. This file corrects the 2026-09-10 snapshot `6135eb6257176d03768af3feb28b2ee32ad76fbe` (that copy was dated 2024 and described code after it had already changed). It supersedes `.planning/codebase/CONCERNS.md` (2026-04-30) wherever that note disagrees with the current tree. `CONCERNS.md` still claims a 142-vs-67 marketplace split and no content CI. Both claims are false: `validate.yml` runs `scripts/validate_skills.py` and ruff, and the bounding-box unittest.

Severity below is for **open** issues. Items fixed in the 2026-10-16 update are listed under Resolved so they are not re-filed as if they were still true.

## Open

### HIGH

#### 1. Commercial image-model dependency, not a K-Dense ad regression

- **Files**: `scientific-skills/scientific-schematics/`, `scientific-skills/scientific-slides/`, and other skills that still tell the agent to call "Nano Banana Pro".
- **What it actually is**: Nano Banana Pro is Google Gemini 3 Pro Image (`google/gemini-3-pro-image-preview`) via `https://openrouter.ai/api/v1`. The scripts set that model id and base URL. It is not an internal K-Dense product, and it is not an undeclared proprietary binary.
- **What the anti-steering promise covers**: `README.md` and `CONTRIBUTING.md` describe removal of K-Dense Web / `k-dense.ai` / "Suggest Using K-Dense" session ads. `PROMO_PATTERNS` in `scripts/validate_skills.py` is those three strings. `check_promo()` scans `SKILL.md` body after frontmatter only. No skill body currently matches those strings.
- **What is still true**: schematics and slides still teach this one paid model as the default way to draw figures. That is a commercial-model dependency. Replacing the nickname with DALL-E or Midjourney would only swap vendors. Adding `Nano Banana Pro` to `PROMO_PATTERNS` would fail CI on skills that mention the nickname and would still miss `scripts/` and `references/`.
- **Do not**: call this a vendor-promotion regression, or tell the validator to grep the nickname.

### MEDIUM

#### 1. Optional Google Scholar free proxies

- **File**: `scientific-skills/citation-management/scripts/search_google_scholar.py` (`GoogleScholarSearcher`, `--use-proxy`).
- **Impact**: The flag still calls `ProxyGenerator().FreeProxies()` and sends the query through untrusted proxies (interception risk, and a Google Scholar terms risk). The default is off, and the CLI now prints a warning before it does this.
- **Fix**: Leave the default off. Prefer removing the flag over routing research queries through free proxies.

#### 2. Promo CI does not read scripts or references

- **File**: `scripts/validate_skills.py`, `check_promo()`.
- **Impact**: A K-Dense steering string in a script or a `references/` file would not fail CI. Nothing in `scientific-skills/` matches the current patterns today, so this is a regression hole, not an active leak.
- **Fix**: If the gate grows, scan `scripts/` and `references/` for the existing three patterns. Do not add the Gemini nickname to that list.

### LOW

#### 1. USPTO markdown samples still omit timeouts

- **Files**: `scientific-skills/uspto-database/SKILL.md` and `scientific-skills/uspto-database/references/*.md`.
- **Impact**: The executable clients `patent_search.py` and `trademark_client.py` set `timeout=30`. Copy-paste samples in the skill prose still show `requests.get` / `requests.post` without a timeout.
- **Fix**: Add `timeout=30` to those samples when they are next edited.

#### 2. Almost no script tests

- **Impact**: CI runs the one unittest at `scientific-skills/document-skills/pdf/scripts/check_bounding_boxes_test.py`. Other bundled scripts have no tests. This repository has no `requirements.txt` or `pyproject.toml` on purpose (`JULES.md`). There is no measured coverage number.
- **Fix**: Add tests beside a script when that script's behavior changes. Do not add `pytest-cov` as a project dependency, and do not treat a coverage percentage as a finding unless the command and output are in this file.

## Resolved (2026-10-16)

These were open in the 2026-09-10 snapshot, or were false there. They are not open findings.

- **Research-integrity mandates.** `literature-review`, `scientific-writing`, `hypothesis-generation`, `research-grants`, `latex-posters`, and `computational-social-science` no longer require AI-generated figures. PRISMA, CONSORT, and STROBE counts must come from the screening or study log. `scientific-schematics` no longer demonstrates a CONSORT diagram with invented counts (500 / 150 / 350).
- **Subprocess injection.** `validate_format.py` calls `pdfinfo` and `pdffonts` as argv lists. `shell` defaults to `False`. There is no command injection, and `shlex.quote()` must not be applied to list arguments. Both calls use `timeout=60`, and both catch `subprocess.TimeoutExpired` so a hang returns a skip instead of raising.
- **Bare `except:`.** `visualize_graph.py`, `detect_resources.py`, and `structure_analyzer.py` no longer use a bare `except:`. Failures are logged. `visualize_graph.py` catches `ImportError`, `ValueError`, `OSError`, and `RuntimeError` around dataset loading. `generate_slide_image_ai.py` temp-file cleanup catches `OSError` and warns. `structure_analyzer.py` reads Wyckoff letters from the symmetry dataset instead of printing the first site's species in that column.
- **Dataset cache path.** `visualize_graph.py` and `benchmark_model.py` store PyG downloads under `$XDG_CACHE_HOME/research-grade-skills/torch-geometric/<name>` (default `~/.cache/...`). They do not use `/tmp/<name>` and they do not use `tempfile.TemporaryDirectory` (that would delete the cache on exit).
- **Request timeouts.** `generate_image.py` uses `timeout=120` and exits on `requests.Timeout`. `patent_search.py` and `trademark_client.py` use `timeout=30`.
- **`.env` walk.** `generate_image.py`, `generate_slide_image_ai.py`, and `generate_schematic_ai.py` read a `.env` from the working directory up through its git root, plus the script directory. They do not continue into a parent account or home directory.
- **Product identity headers.** Those OpenRouter clients send `HTTP-Referer: https://github.com/Yuuqq/research-grade-skills`. They no longer send `scientific-writer`.
- **`skill-author: K-Dense Inc.`** Not a defect. `JULES.md` requires that frontmatter field. Attribution in frontmatter, LICENSE, and README is allowed. Body-text steering is what `PROMO_PATTERNS` bans.
- **Coverage figure.** The old "~34%" claim had no command and no output. It is withdrawn.
- **`pytest` as a development dependency.** Not adopted. The existing unittest is invoked directly from `validate.yml`.
