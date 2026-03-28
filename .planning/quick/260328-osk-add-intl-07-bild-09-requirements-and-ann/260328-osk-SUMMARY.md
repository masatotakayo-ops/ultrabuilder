# Quick Task 260328-osk: Add INTL-07, BILD-09 requirements and annotate INTL-02

**Completed:** 2026-03-28
**Commit:** 3dee5c2

## What Changed

### REQUIREMENTS.md

1. **INTL-02 annotated** -- added file-type context routing detail (TypeScript / Python / Rust / YAML language-specific instruction profiles for Builder agent)
2. **INTL-07 added** -- explicit capability manifests per swarm agent defining allowed tools, write access scope, and prohibited actions for Architect, Builder, Security, and QA agents
3. **BILD-09 added** -- Architect agent creates/maintains CONVENTIONS.md in target repo at checkpoints; Builder reads it before every code generation task
4. **Traceability table updated** -- added INTL-07 (Phase 4) and BILD-09 (Phase 5) rows
5. **Coverage count updated** -- 54 to 56 total v1 requirements

### ROADMAP.md

1. **Phase 4 requirements list** -- added INTL-07
2. **Phase 5 requirements list** -- added BILD-09

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Missing critical functionality] Added traceability rows and updated requirement count**
- **Found during:** REQUIREMENTS.md edits
- **Issue:** Adding new requirements without updating the traceability table and coverage count would leave the document inconsistent
- **Fix:** Added INTL-07 and BILD-09 rows to traceability table, updated count from 54 to 56
- **Files modified:** .planning/REQUIREMENTS.md

## Self-Check: PASSED
