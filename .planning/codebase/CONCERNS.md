# Codebase Concerns

> **Superseded for inventory and CI claims.** This note is dated 2026-04-30 and describes a checkout that no longer exists (marketplace 142 vs disk 67, "no CI"). Current registry checks live in `scripts/validate_skills.py` and `.github/workflows/validate.yml`. Open findings are in the advisory [REVIEW.md](../../REVIEW.md) (updated 2026-10-16). Do not treat the critical items below as the current tree.

**Analysis Date:** 2026-04-30

## Inventory Integrity: Marketplace vs Disk Divergence

**Critical - The single source of truth is broken.**

- Issue: `.claude-plugin/marketplace.json` registers 142 skill paths, but only 67 skill directories exist on disk. 74 registered skills are completely missing from the working tree.
- Files: `.claude-plugin/marketplace.json`, `scientific-skills/` (directory listing)
- Impact: Users installing the plugin will reference skills that do not exist in the repository. Claude Code plugin loader will encounter missing paths. Any automated build or validation would fail immediately.
- Affected skills (registered but absent): `adaptyv`, `alphafold-database`, `anndata`, `arboreto`, `benchling-integration`, `biomni`, `biopython`, `bioservices`, `brenda-database`, `cellxgene-census`, `chembl-database`, `clinical-decision-support`, `clinical-reports`, `clinicaltrials-database`, `clinpgx-database`, `clinvar-database`, `cobrapy`, `cosmic-database`, `datamol`, `deepchem`, `deeptools`, `diffdock`, `dnanexus-integration`, `document-skills/docx`, `document-skills/pdf`, `document-skills/pptx`, `document-skills/xlsx`, `drugbank-database`, `ena-database`, `ensembl-database`, `esm`, `etetoolkit`, `fda-database`, `flowio`, `gene-database`, `geniml`, `geo-database`, `gget`, `gtars`, `gwas-database`, `histolab`, `hmdb-database`, `iso-13485-certification`, `kegg-database`, `labarchive-integration`, `lamindb`, `latchbio-integration`, `matchms`, `medchem`, `metabolomics-workbench-database`, `molfeat`, `neurokit2`, `omero-integration`, `opentargets-database`, `opentrons-integration`, `pathml`, `pdb-database`, `protocolsio-integration`, `pubchem-database`, `pydeseq2`, `pydicom`, `pyhealth`, `pylabrobot`, `pyopenms`, `pysam`, `pytdc`, `rdkit`, `reactome-database`, `rowan`, `scanpy`, `scikit-bio`, `scvi-tools`, `string-database`, `torchdrug`, `treatment-plans`, `uniprot-database`, `zarr-python`, `zinc-database`
- Fix approach: Restore the 74 missing skill directories from git history (`git checkout HEAD -- scientific-skills/<name>`), or update `marketplace.json` to reflect the actual on-disk state. The git history confirms these files were tracked and have been deleted from the working tree (465 unstaged deletions, ~193,766 lines removed).

## Unstaged Mass Deletion in Working Tree

**Critical - 465 files deleted but not committed.**

- Issue: `git status` shows 465 unstaged deletions across 76 skill directories. These files exist in git history but are absent from the working tree. This creates a state where the repository appears complete in git but the local checkout is missing most content.
- Files: All 76 skill directories listed in the `Missing from disk` section above, plus their `references/`, `scripts/`, and `assets/` subdirectories.
- Impact: Anyone cloning or checking out this repository gets an incomplete dataset. The release workflow (`git push` to main with `marketplace.json` changes) would create a release referencing skills that no longer ship.
- Fix approach: Either `git checkout HEAD -- scientific-skills/` to restore all deleted content, or explicitly commit the deletions and update `marketplace.json` to match. The current state is neither a clean deletion nor a complete checkout.

## Orphaned Skills on Disk (Not in Marketplace)

**Medium - 2-3 skills present but invisible to users.**

- Issue: Two skill directories exist on disk but are not registered in `marketplace.json`: `computational-social-science` and `general-data-science`. Additionally, `document-skills` as a parent directory is on disk but only its four sub-skills (`docx`, `pdf`, `pptx`, `xlsx`) are registered.
- Files: `scientific-skills/computational-social-science/`, `scientific-skills/general-data-science/`, `scientific-skills/document-skills/` (parent)
- Impact: Users cannot discover or install these skills through the plugin system. Contributors may modify them thinking they are active.
- Fix approach: Either register them in `marketplace.json` or remove them from the repository.

## Orphaned Reference Directory

**Low - 3 draft skills outside the skill structure.**

- Issue: The `reference/` directory contains three skill directories (`academic-reviewer`, `dspy`, `scientist`) that are outside the `scientific-skills/` directory structure and not registered in `marketplace.json`.
- Files: `reference/academic-reviewer/SKILL.md`, `reference/dspy/SKILL.md`, `reference/scientist/SKILL.md`
- Impact: These are invisible to the plugin system and unclear whether they are drafts, archived, or misplaced.
- Fix approach: Move to `scientific-skills/` and register if intended for release, or remove if drafts.

## No Content Validation or Quality Checks

**High - Zero automated quality enforcement.**

- Issue: There are no linting tools, schema validators, CI checks, Makefiles, or scripts that validate SKILL.md frontmatter, check for broken references, verify API endpoint URLs, or ensure skill content consistency. The only CI workflow is `.github/workflows/release.yml` which creates a GitHub release when `marketplace.json` changes -- it does not validate skill content.
- Files: `.github/workflows/release.yml` (only workflow)
- Impact: Skills can be merged with broken frontmatter, missing sections, invalid cross-references, or incorrect metadata. No automated gate prevents publishing degraded content.
- Fix approach: Add a CI workflow that validates: (1) every path in `marketplace.json` points to a directory containing `SKILL.md`, (2) every `SKILL.md` has required frontmatter fields (`name`, `description`, `license`), (3) no broken cross-skill references, (4) Python scripts have valid syntax.

## Python Script Portability: No Dependency Management

**High - 106 Python scripts with no formal dependency declarations.**

- Issue: The repository contains 106 Python scripts across skills (e.g., `scientific-skills/citation-management/scripts/search_pubmed.py`, `scientific-skills/generate-image/scripts/generate_image.py`, `scientific-skills/document-skills/pdf/scripts/*.py`). There are zero `requirements.txt` files and zero `pyproject.toml` files committed. The `.gitignore` explicitly excludes `pyproject.toml` and `uv.lock`.
- Files: `scientific-skills/*/scripts/*.py` (106 files), `.gitignore` (lines 8-9)
- Impact: Users running Python scripts will encounter `ModuleNotFoundError` for dependencies like `scholarly`, `requests`, `pypdf`, `pandas`, `numpy`, `biopython`, `bibtexparser`, `crossref-commons`, `selenium`. Dependencies are documented in prose within `SKILL.md` files but not in machine-readable format.
- Fix approach: Either (a) add per-skill `requirements.txt` files, (b) use inline `uv run --with <pkg>` commands in script invocations, or (c) add a centralized dependency manifest. At minimum, each script should declare its imports and document required packages at the top.

## Cross-Skill Dependencies Not Formally Declared

**High - Skills depend on other skills with no dependency graph.**

- Issue: Multiple skills reference other skills as dependencies or complementary tools, but there is no formal dependency declaration in the SKILL.md frontmatter or any manifest file.
- Key dependency chains observed:
  - `literature-review` depends on `scientific-schematics` (MANDATORY: "every literature review MUST include at least 1-2 AI-generated figures using the scientific-schematics skill") -- `scientific-skills/literature-review/SKILL.md:31`
  - `market-research-reports` depends on `scientific-schematics` and `generate-image` -- `scientific-skills/market-research-reports/SKILL.md:48,74`
  - `peer-review` depends on `scientific-schematics` -- `scientific-skills/peer-review/SKILL.md:32`
  - `scientific-slides` depends on `scientific-schematics` and `pptx` -- `scientific-skills/scientific-slides/SKILL.md:174,304`
  - `scientific-writing` depends on `venue-templates` -- `scientific-skills/scientific-writing/SKILL.md:686`
  - Multiple skills depend on `scientific-schematics` for diagram generation (at least 10 skills reference "Nano Banana Pro" for AI-powered figure generation)
- Impact: If a dependency skill is missing or broken, dependent skills produce incomplete or incorrect output without any warning. The "Nano Banana Pro" references suggest a specific AI image generation pipeline that may not be available to all users.
- Fix approach: Add a `depends-on` field to SKILL.md frontmatter declaring skill-level dependencies. Validate dependency chains at CI time.

## Inconsistent SKILL.md Frontmatter Schema

**Medium - No enforced schema across 66+ skill files.**

- Issue: SKILL.md files have inconsistent frontmatter fields. Required fields are not enforced.
- Observed fields across skills:
  - `name`: Present in all skills (consistent)
  - `description`: Present in all skills (consistent)
  - `license`: Present in all, but 6 have "Unknown" and format varies (e.g., "MIT license" vs "MIT License" vs "Apache-2.0 license" vs "Apache License, Version 2.0")
  - `compatibility`: Only 4 of 66 skills declare this field (`generate-image`, `matlab`, `perplexity-search`, `transformers`)
  - `allowed-tools`: Only 18 of 66 skills declare this field; 16 use `[Read, Write, Edit, Bash]`, 2 use `[Read, Write, Edit, Bash, Python]`
  - `skill-author`: Present in all skills (consistent)
  - `version`: Not present in any skill file
- Files: All `scientific-skills/*/SKILL.md` files
- Impact: No machine-readable way to determine which skills require API keys, which tools they need, or what license applies without parsing prose text. Users cannot filter skills by compatibility.
- Fix approach: Define a required frontmatter schema. Standardize license values to SPDX identifiers. Require `compatibility` for any skill needing external API keys.

## No Versioning Strategy for Individual Skills

**Medium - Skills have no independent version tracking.**

- Issue: No SKILL.md file contains a `version` field. The only version is the marketplace-level version in `.claude-plugin/marketplace.json` (currently `2.17.0`). When a single skill is updated, the entire marketplace version must be bumped, but there is no way to know which skills changed or what version of each skill is installed.
- Files: `.claude-plugin/marketplace.json` (line 9: `"version": "2.17.0"`), all `scientific-skills/*/SKILL.md` files
- Impact: Users cannot determine if a specific skill has been updated. Breaking changes to one skill require a full marketplace release. No rollback path for individual skills.
- Fix approach: Add per-skill `version` fields to SKILL.md frontmatter. Consider using the marketplace version as a monorepo version with per-skill changelogs.

## API Key Requirements Scattered and Undocumented Centrally

**Medium - Users discover API key requirements by failing.**

- Issue: At least 10 skills require external API keys, but only 4 declare this in the `compatibility` frontmatter field. Other skills bury API key requirements in prose documentation.
- Skills requiring API keys (observed):
  - `generate-image`: OpenRouter API key (declared in `compatibility`) -- `scientific-skills/generate-image/SKILL.md:5`
  - `perplexity-search`: OpenRouter API key (declared in `compatibility`) -- `scientific-skills/perplexity-search/SKILL.md:5`
  - `transformers`: HuggingFace token (declared in `compatibility`) -- `scientific-skills/transformers/SKILL.md:5`
  - `datacommons-client`: DC_API_KEY -- `scientific-skills/datacommons-client/SKILL.md:219`
  - `citation-management`: NCBI API key (optional but recommended) -- `scientific-skills/citation-management/references/pubmed_search.md:362`
  - `cirq`: IonQ API key for hardware access -- `scientific-skills/cirq/references/hardware.md:163`
  - `denario`: OpenAI API key for LLM features -- `scientific-skills/denario/references/llm_configuration.md:60`
  - `markitdown`: OpenRouter API key for AI image descriptions -- `scientific-skills/markitdown/scripts/convert_with_ai.py:167`
  - `paper-2-web`: OpenRouter API key -- `scientific-skills/paper-2-web/references/installation.md:64`
  - `modal`: Modal authentication token -- `scientific-skills/modal/SKILL.md:29`
  - `matlab`: MATLAB or Octave installation (declared in `compatibility`) -- `scientific-skills/matlab/SKILL.md:5`
- Impact: Users install the full skill bundle expecting plug-and-play functionality but encounter cryptic failures when API keys are missing.
- Fix approach: Require `compatibility` field for all skills with external dependencies. Create a central `API_KEYS.md` documenting all external service requirements.

## License Metadata Inconsistencies

**Medium - 6 skills have "Unknown" license, formatting varies.**

- Issue: License metadata across skills uses inconsistent formats and 6 skills have "Unknown" as their license value.
- Observed values (66 skills on disk):
  - "MIT license" (34 skills)
  - "BSD-3-Clause license" (9 skills)
  - "Apache-2.0 license" (7 skills)
  - "Unknown" (6 skills)
  - "GPL-3.0 license" (2 skills)
  - "MIT License" (1 skill -- different capitalization)
  - URL-based values: `https://github.com/sympy/sympy/blob/master/LICENSE` (1), `https://github.com/pola-rs/polars/blob/main/LICENSE` (1), `https://github.com/matplotlib/matplotlib/tree/main/LICENSE` (1)
  - Verbose: "For MATLAB (https://www.mathworks.com/pricing-licensing.html) and for Octave (GNU General Public License version 3)" (1), "CeCILL FREE SOFTWARE LICENSE AGREEMENT" (1), "Apache License, Version 2.0" (1), "3-clause BSD license" (1)
- Files: All `scientific-skills/*/SKILL.md` files
- Impact: Users cannot programmatically filter skills by license. The README FAQ states "Users are responsible for reviewing and adhering to the license terms" but "Unknown" makes compliance impossible. URL-based values require manual lookup.
- Fix approach: Standardize all license values to SPDX identifiers (e.g., `MIT`, `BSD-3-Clause`, `Apache-2.0`, `GPL-3.0-only`). Investigate and fill in the 6 "Unknown" licenses.

## Documentation Currency Risk

**Medium - External APIs change, skills may become stale.**

- Issue: Skills document external API endpoints, query parameters, and response formats for 28+ scientific databases (OpenAlex, PubMed, bioRxiv, ChEMBL, UniProt, COSMIC, ClinicalTrials.gov, KEGG, Reactome, STRING, PDB, etc.). There is no automated check that these APIs still behave as documented. No skill files contain a "last verified" date or API version reference.
- Files: `scientific-skills/*-database/SKILL.md` and `scientific-skills/*-database/references/api_reference.md` files (28+ database skills)
- Impact: API endpoints change, response schemas evolve, authentication requirements shift. Skills referencing outdated API behavior will produce errors or incorrect results. This is especially critical for clinical/biomedical databases where accuracy matters.
- Fix approach: Add a `last-verified` date to SKILL.md frontmatter. Create a periodic CI job that smoke-tests API endpoints. Add API version tracking to database skill references.

## Content Duplication: "Nano Banana Pro" References

**Low - Hardcoded AI model references across 10+ skills.**

- Issue: At least 10 skills contain references to "Nano Banana Pro" for AI-powered figure generation, with instructions like "Nano Banana Pro will automatically generate, review, and refine the schematic." This appears to be an internal K-Dense product name embedded in open-source skill documentation.
- Files: `scientific-skills/citation-management/SKILL.md:39`, `scientific-skills/literature-review/SKILL.md:40`, `scientific-skills/hypothesis-generation/SKILL.md:37`, `scientific-skills/latex-posters/SKILL.md:40`, `scientific-skills/markitdown/SKILL.md:31`, `scientific-skills/paper-2-web/SKILL.md:48`, `scientific-skills/document-skills/docx/SKILL.md:20`, `scientific-skills/document-skills/pdf/SKILL.md:20`, `scientific-skills/document-skills/pptx/SKILL.md:20`, `scientific-skills/document-skills/xlsx/SKILL.md:76`
- Impact: Users outside K-Dense Web may not have access to "Nano Banana Pro." These references create a dependency on a proprietary service without declaring it in frontmatter. The instructions are embedded as mandatory steps ("MUST include") but the referenced tool may not be available.
- Fix approach: Either make "Nano Banana Pro" available as a skill (e.g., `scientific-schematics`), or make these references optional and document the fallback approach.

## Documentation Count Inconsistencies

**Low - README claims 140 skills, marketplace has 142.**

- Issue: The README badge states "140" skills, the README body states "140 scientific skills" in multiple places, but `marketplace.json` registers 142 skill paths. The `docs/open-source-sponsors.md` states "139 skills."
- Files: `README.md` (lines 3, 6, 67, 410), `docs/open-source-sponsors.md` (line 3), `.claude-plugin/marketplace.json` (142 entries)
- Impact: Minor credibility issue. Users may question accuracy of other claims.
- Fix approach: Update all count references to match the actual number in `marketplace.json`, or derive the count programmatically.

## No Test Infrastructure

**High - Zero tests for a repository with 106 Python scripts.**

- Issue: There are no test files, no test configuration, and no test runner setup. The only test file observed is `scientific-skills/document-skills/pdf/scripts/check_bounding_boxes_test.py` (a single test file in the entire repository). There is no `pytest.ini`, `conftest.py`, or test directory structure.
- Files: `scientific-skills/document-skills/pdf/scripts/check_bounding_boxes_test.py` (sole test)
- Impact: Python scripts can break silently. API integrations can fail without detection. No regression protection when skills are updated.
- Fix approach: Add a `tests/` directory with at minimum: (1) SKILL.md schema validation tests, (2) Python script syntax/import tests, (3) marketplace.json integrity tests. Use pytest with a lightweight CI workflow.

## .gitignore Excludes Dependency Lock Files

**Medium - Intentional but creates reproducibility gap.**

- Issue: The `.gitignore` excludes `pyproject.toml`, `uv.lock`, `.python-version`, and `main.py`. This means there is no committed dependency lock file and no Python version pinning for the repository.
- Files: `.gitignore` (lines 8-11)
- Impact: Different users installing skills at different times may get different dependency versions. Reproducibility of skill execution is not guaranteed.
- Fix approach: Either commit a minimal `pyproject.toml` with optional dependency groups per skill, or document the expected Python version range and dependency versions in each skill's SKILL.md.

## Security: Shell Command Patterns in Documentation

**Low - Historical concern, recently addressed.**

- Issue: A recent commit (`b6a6d69`) fixed shell injection patterns by replacing `shell=True` with safe subprocess patterns. However, no automated check prevents reintroduction of unsafe patterns.
- Files: Commit `b6a6d69` (fix(security): replace shell=True with safe subprocess patterns in documentation)
- Impact: Future skill contributions may reintroduce `shell=True` or other unsafe patterns.
- Fix approach: Add a lint rule or CI check that flags `shell=True` in Python files.

## License Compliance for Referenced Open-Source Projects

**Medium - 50+ open-source projects referenced, compliance not verified.**

- Issue: `docs/open-source-sponsors.md` lists 50+ open-source projects that skills depend on. Each project has its own license (MIT, BSD, Apache, GPL, etc.). Skills that include code examples derived from these projects must comply with their licenses. GPL-licensed skills (`denario` GPL-3.0, one unnamed skill) may have copyleft implications if bundled with MIT-licensed content.
- Files: `docs/open-source-sponsors.md`, `LICENSE.md`, `scientific-skills/*/SKILL.md` (license fields)
- Impact: GPL-licensed skills bundled in an MIT-licensed repository create a licensing tension. Users redistributing the plugin may inadvertently violate GPL terms. The 6 "Unknown" license skills cannot be legally assessed.
- Fix approach: Review all 6 "Unknown" licenses and determine their actual license. Verify GPL-3.0 skills are clearly documented as having different terms. Consider whether GPL skills should be in a separate optional bundle.

---

*Concerns audit: 2026-04-30*
