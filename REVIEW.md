# Repository Code Review: research-grade-skills

This file documents the findings from a comprehensive code review of the `research-grade-skills` repository, organized by severity (CRITICAL, HIGH, MEDIUM, LOW).

## CRITICAL

### 1. Undeclared Vendor Dependencies (Vendor Promotion Regressions)
- **Files**: `scientific-skills/scientific-schematics/SKILL.md`, `scientific-skills/scientific-slides/SKILL.md`, `scientific-skills/markitdown/SKILL.md`, `scientific-skills/venue-templates/SKILL.md`, and references/scripts in `scientific-schematics` and `scientific-slides` (e.g., `generate_schematic_ai.py`, `generate_slide_image_ai.py`), plus many others referencing "Nano Banana Pro" (25 files total).
- **Impact**: The repository description and README explicitly state "All vendor promotion removed" and "no vendor ads". However, numerous skills extensively promote and instruct the use of "Nano Banana Pro" (an apparent internal K-Dense AI product). This creates an undeclared proprietary dependency and explicitly violates the project's core anti-vendor-steering promise. The CI validation (`validate_skills.py`) checks for `k-dense.ai` and `Suggest Using K-Dense`, but misses `Nano Banana Pro`.
- **Fix**: Remove all references to "Nano Banana Pro" from `SKILL.md` files and associated scripts. If the skills are meant to rely on generic image generation (like DALL-E or Midjourney), rephrase to reflect that. Update `scripts/validate_skills.py` to add `re.compile(r"Nano Banana Pro", re.IGNORECASE)` to the `PROMO_PATTERNS` to prevent regressions.

## HIGH

### 1. Security: Insecure Use of `subprocess` (Command Injection Risks)
- **Files**: `scientific-skills/scientific-slides/scripts/validate_presentation.py` (Lines 247-252), `scientific-skills/venue-templates/scripts/validate_format.py` (Lines 59-64, 117-122).
- **Impact**: The `subprocess.run` calls execute external binaries (like `pdflatex`, `pdfinfo`, `pdffonts`) using paths and filenames that might originate from untrusted user input without explicitly setting `shell=False` or fully sanitizing the paths. While `shell=True` isn't used explicitly here, `bandit` flagged these as potential command injection vectors because they pass raw string inputs into system commands.
- **Fix**: Ensure that all paths passed to `subprocess.run` are strictly validated, absolute paths, or explicitly wrapped in `shlex.quote()` if shell execution is ever required. Ensure `shell=False` is explicit and only specific known binaries are called.

### 2. Reliability: Bare `except` or `except Exception: pass` Clauses
- **Files**: `scientific-skills/scientific-slides/scripts/generate_slide_image_ai.py` (Lines 658-662).
- **Impact**: The script attempts to clean up temporary files using `temp_file.unlink()`. If this fails, it catches a generic `Exception` and does nothing (`pass`). This is an anti-pattern that can silently swallow important errors (like permission issues or keyboard interrupts if catching `BaseException`, though here it's `Exception`) and make debugging very difficult.
- **Fix**: Catch specific exceptions, such as `OSError` or `FileNotFoundError`, and log a warning or error message rather than silently passing. For example:
  ```python
  except OSError as e:
      print(f"Warning: Failed to delete temp file {temp_file}: {e}", file=sys.stderr)
  ```

## MEDIUM

### 1. Security: Hardcoded Insecure Temporary Directories
- **Files**: `scientific-skills/torch-geometric/scripts/benchmark_model.py` (Lines 144, 183), `scientific-skills/torch-geometric/scripts/visualize_graph.py` (Lines 276, 282).
- **Impact**: The scripts hardcode `/tmp/{dataset_name}` as the root directory for datasets. Hardcoding `/tmp/` paths can lead to symlink attacks, race conditions, or permission conflicts if multiple users run the script on the same machine.
- **Fix**: Use Python's built-in `tempfile` module (e.g., `tempfile.gettempdir()` or `tempfile.TemporaryDirectory`) to safely manage temporary storage paths, or default to a local `./data` directory relative to the current working directory.

### 2. Reliability: Network Requests Without Timeouts
- **Files**: `scientific-skills/uspto-database/scripts/patent_search.py` (Line 70), `scientific-skills/uspto-database/scripts/trademark_client.py` (Lines 55, 76).
- **Impact**: Using `requests.post()` and `requests.get()` without specifying a `timeout` argument means the program can hang indefinitely if the remote server (USPTO APIs) is unresponsive or drops packets.
- **Fix**: Always include a `timeout` argument in `requests` calls (e.g., `requests.get(url, headers=self.headers, timeout=30)`).

### 3. Test Coverage: Extremely Low Coverage for Scripts
- **Impact**: Running `pytest --cov` reveals that while `check_bounding_boxes.py` has tests (`check_bounding_boxes_test.py`), almost all other Python scripts in the `scientific-skills/` directory have zero unit tests. Total script coverage is ~34% primarily due to lack of tests.
- **Fix**: Create a testing plan to add unit tests for critical data processing and API integration scripts, particularly those interacting with external APIs (like OpenAlex, USPTO, bioRxiv) and file parsing (like DOCX, PDF, PPTX validations).

## LOW

### 1. Developer Experience: `pytest` and test infrastructure not standard
- **Impact**: The CI workflow `.github/workflows/validate.yml` runs `scripts/validate_skills.py` and `ruff`, but it does not invoke `pytest`. There are tests in the codebase (e.g., `check_bounding_boxes_test.py`), but they are not enforced on commit.
- **Fix**: Add a `pytest` step to `validate.yml` to ensure all existing and future unit tests are run in CI. Add `pytest` and `pytest-cov` to the development dependencies.

### 2. Documentation: Hardcoded `skill-author` metadata
- **Impact**: In all `SKILL.md` files, `metadata.skill-author` is hardcoded to `K-Dense Inc.`. While this is mentioned in the `README.md` as attribution, it contradicts the claim that this is an independent descendant if K-Dense is permanently listed as the active author metadata for agent ingestion.
- **Fix**: Consider updating the frontmatter schema to reflect the independent nature of the repository (e.g., changing `skill-author` to a community handle, or moving the K-Dense attribution solely to the `LICENSE` or a `derived-from` field), to avoid confusing AI agents about the origin/support of the skills.
