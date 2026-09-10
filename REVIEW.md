# Repository Code Review: research-grade-skills

*Note: This is an advisory review document (Date: 2024-09-10). It is not a CI gate.*

This file documents the findings from a comprehensive code review of the `research-grade-skills` repository, organized by severity (CRITICAL, HIGH, MEDIUM, LOW).

## CRITICAL

### 1. Undeclared Vendor Dependencies (Vendor Promotion Regressions)
- **Files**: `scientific-skills/scientific-schematics/SKILL.md`, `scientific-skills/scientific-slides/SKILL.md`, `scientific-skills/markitdown/SKILL.md`, `scientific-skills/venue-templates/SKILL.md`, and references/scripts in `scientific-schematics` and `scientific-slides` (e.g., `generate_schematic_ai.py`, `generate_slide_image_ai.py`), plus many others referencing "Nano Banana Pro" (25 files total).
- **Impact**: The repository description and README explicitly state "All vendor promotion removed" and "no vendor ads". However, numerous skills extensively promote and instruct the use of "Nano Banana Pro" (which is Google Gemini 3 Pro Image via OpenRouter). This creates an undeclared proprietary dependency and explicitly violates the project's core anti-vendor-steering promise. The CI validation (`validate_skills.py`) checks for `k-dense.ai` and `Suggest Using K-Dense`, but misses this model name because the CI promo gate only watches `SKILL.md` body after frontmatter. Additionally, some skills (like `literature-review/SKILL.md`) make AI-generated figures mandatory, which creates research integrity issues.
- **Fix**: Correct the identification of the model. Treat this as an undeclared commercial-model lock-in. Update `scripts/validate_skills.py` to watch out for vendor dependencies beyond just the current PROMO_PATTERNS strings. Remove the mandate for AI-generated figures in literature reviews.

## HIGH

### 1. Reliability: Missing Timeouts in subprocess calls
- **Files**: `scientific-skills/venue-templates/scripts/validate_format.py` (Lines 59-64, 117-122).
- **Impact**: The `subprocess.run` calls execute external binaries (like `pdfinfo`, `pdffonts`) without setting a `timeout`. This can cause scripts to hang indefinitely if the underlying process blocks.
- **Fix**: Add a `timeout` argument (e.g., `timeout=60`) to `subprocess.run()` calls. Note: these are argv-list invocations, so `shell` defaults to `False`. There is no command injection risk here, and `shlex.quote()` should NOT be used on list arguments.

### 2. Reliability: Bare `except` or `except Exception: pass` Clauses
- **Files**: `scientific-skills/scientific-slides/scripts/generate_slide_image_ai.py` (Lines 658-662), `scientific-skills/torch-geometric/scripts/visualize_graph.py` (Line 278), `scientific-skills/get-available-resources/scripts/detect_resources.py` (Line 192), `scientific-skills/pymatgen/scripts/structure_analyzer.py` (Line 142).
- **Impact**: The scripts catch generic exceptions (and in some cases, bare `except:` clauses which swallow `KeyboardInterrupt` and `SystemExit`) and do nothing (`pass`). This is an anti-pattern that can silently swallow important errors and make debugging very difficult.
- **Fix**: Catch specific exceptions, such as `Exception`, `OSError` or `FileNotFoundError`, and log a warning or error message rather than silently passing. Avoid bare `except:` clauses.

## MEDIUM

### 1. Security: Hardcoded Insecure Temporary Directories
- **Files**: `scientific-skills/torch-geometric/scripts/benchmark_model.py` (Lines 144, 183), `scientific-skills/torch-geometric/scripts/visualize_graph.py` (Lines 276, 282).
- **Impact**: The scripts hardcode `/tmp/{dataset_name}` as the root directory for datasets. Hardcoding `/tmp/` paths can lead to symlink attacks, race conditions, or permission conflicts if multiple users run the script on the same machine.
- **Fix**: Use Python's built-in `tempfile` module (e.g., `tempfile.gettempdir()` or `tempfile.TemporaryDirectory`) to safely manage temporary storage paths, or default to a local `./data` directory relative to the current working directory.

### 2. Reliability: Network Requests Without Timeouts
- **Files**: `scientific-skills/uspto-database/scripts/patent_search.py` (Line 70), `scientific-skills/uspto-database/scripts/trademark_client.py` (Lines 55, 76).
- **Impact**: Using `requests.post()` and `requests.get()` without specifying a `timeout` argument means the program can hang indefinitely if the remote server (USPTO APIs) is unresponsive or drops packets.
- **Fix**: Always include a `timeout` argument in `requests` calls (e.g., `requests.get(url, headers=self.headers, timeout=30)`).

### 3. Test Coverage: Low Coverage for Scripts
- **Impact**: There are very few unit tests for scripts in the `scientific-skills/` directory (e.g., `check_bounding_boxes_test.py`).
- **Fix**: Create a testing plan to add unit tests for critical data processing and API integration scripts, particularly those interacting with external APIs (like OpenAlex, USPTO, bioRxiv) and file parsing (like DOCX, PDF, PPTX validations).

## LOW

### 1. Developer Experience: `pytest` and test infrastructure not standard
- **Impact**: The CI workflow `.github/workflows/validate.yml` runs `scripts/validate_skills.py` and `ruff`, but it does not invoke `pytest`. There are tests in the codebase (e.g., `check_bounding_boxes_test.py`), but they are not enforced on commit.
- **Fix**: Add a `pytest` step to `validate.yml` to ensure all existing and future unit tests are run in CI. Add `pytest` and `pytest-cov` to the development dependencies.
