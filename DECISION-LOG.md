# Decision Log

## Session 2026-03-15 (v11 → v12)

### DEC-036 | Installation instructions fix for updates

- **Decision:** Added `rm -rf` to installation commands in `README.md` before cloning/copying.
- **Reason:** Existing folders prevented `git clone` and `cp` from updating the skill correctly. Users reported failure to update existing projects.
- **Status:** Done (README.md).

### DEC-035 | Strict standalone photo destination

- **Decision:** Enforce `5-Fotos/` as the *only* destination for memory library photos. Explicitly prohibit placement inside `2-Areas` or `3-Resources`.
- **Reason:** Photos were being incorrectly classified based on context (e.g., Hobbies or Images) rather than their chronological library purpose. Correction ensures PARA categories stay focused on documents.
- **Status:** Done (references/MEDIA-HANDLING.md).

### DEC-034 | Naming conventions consolidation

- **Decision:** Created dedicated reference file for all naming conventions (`references/RENAMING.md`).
- **Reason:** Naming rules were fragmented across multiple files, causing inconsistencies.
- **Status:** Done (references/RENAMING.md).

### DEC-033 | Annual maintenance schedule

- **Decision:** Added an annual row to the maintenance cycle (60 min).
- **Reason:** Long-term tasks like Legado purge and full reindexing shouldn't clutter the quarterly cycle.
- **Status:** Done (references/MAINTENANCE-AND-COMMANDS.md).

### DEC-032 | Visible Inbox underscore prefix

- **Decision:** Enforce `_Inbox/` as the default name.
- **Reason:** Ensures the folder always appears at the top of listings and is easily identifiable by users.
- **Status:** Done (SKILL.md).

### DEC-031 | Area vs Resource disambiguation

- **Decision:** Added content-based redirection for learning materials matching an Area alias.
- **Reason:** Users often have "Finances" or "Health" courses that are Resources, not Areas. Reduces misclassification.
- **Status:** Done (SKILL.md).

### DEC-030 | Quarantine system for broken dependencies

- **Decision:** Implemented `Quarentena/` folder and "resolve quarantine" command.
- **Reason:** Users needed a way to defer resolution of items with broken links or system locks without stopping the entire workflow.
- **Status:** Done (references/DEPENDENCY-CHECKS.md).

### DEC-029 | Version Series protection

- **Decision:** Explicitly excluded files with version markers (v1, v2) from duplicate detection.
- **Reason:** Versioned copies are valuable history, not redundant duplicates.
- **Status:** Done (references/DUPLICATE-DETECTION.md).

### DEC-028 | Photo library standalone mode

- **Decision:** Added `mediaPolicy` option and standalone photo library destination.
- **Reason:** Large personal photo collections often overwhelm document-focused PARA structures.
- **Status:** Done (references/MEDIA-HANDLING.md).

## Older Decisions

### DEC-027 | Audit trail in manifests

- **Decision:** Record `originalName` and `originalPath` even for local moves.
- **Reason:** Critical for traceability if a user wants to find where a specific file came from months later.
- **Status:** Done (Step 6).

### DEC-026 | Deduplication algorithm

- **Decision:** Two-phase (size+extension then SHA-256 hash).
- **Reason:** Calculating hashes for thousands of files is slow. Size grouping filters 90% of non-duplicates instantly.
- **Status:** Done (Step 2.5).