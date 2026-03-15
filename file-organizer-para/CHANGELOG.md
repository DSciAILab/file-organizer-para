# Changelog

## [12.0.0] - 2026-03-15

### New files
- references/RENAMING.md: dedicated reference for all naming
  conventions, rename triggers, photo/audio rename, version series
  vs accidental duplicate detection, transliteration rules,
  batch rename flow, and manifest recording for renames.

### references/DUPLICATE-DETECTION.md
- Added version series definition and detection: files with
  explicit version markers (v1/v2, -draft/-approved) are now
  classified as VERSION SERIES and excluded from duplicate actions.
- Added probable duplicate decision flow: structured per-pair
  presentation with options (keep both / archive older /
  investigate / skip) instead of generic "manual review only".
- Fixed on-demand dedup archive destination: when no PARA root
  exists, agent now asks user for archive location and suggests
  a path outside the scan target. Previously created archive
  inside the scan target, causing it to appear in next scan.
- Added Known limitations section: recompressed images, format
  duplicates (.docx vs .pdf), edited copies.
- Clarified Phase 2 step 5: check for version markers before
  flagging as probable duplicate.

### references/MEDIA-HANDLING.md
- Fixed photo default destination: changed from
  4-Arquivo/Fotos/YYYY/YYYY-MM/ to 5-Fotos/YYYY/YYYY-MM_Event-Name/.
  Photos are a memory library, not archived inactive documents.
  Placing them in 4-Arquivo conflicted with PARA Archive semantics.
- Added photo library structure section with full folder hierarchy.
- Added event name assignment flow: GPS opt-in, keyword analysis
  fallback, user confirmation before folder creation.
- Updated standalone option (b) path: ~/Photos/YYYY/MM/ ->
  <root>/5-Fotos/YYYY/YYYY-MM_Event-Name/ with event names.
- Made option (a) standalone library the default (was PARA structure).
- Added perceptual hash section: opt-in for recompressed photo dedup
  using ImageMagick or Python imagehash. Results as
  PROBABLE_VISUAL_DUPLICATE, never auto-archived.
- Screenshots destination updated: 3-Recursos/Screenshots/YYYY-MM/.
- Added reference to RENAMING.md for photo rename convention.

### references/DEPENDENCY-CHECKS.md
- Added Quarantine option to "Options per flag type": move FLAGGED
  items to Quarentena/ for deferred resolution.
- Added Quarantine folder section: full documentation of
  Quarentena/ purpose, contents, rules (not indexed, not renamed,
  excluded from subsequent scans), resolution flow via
  "resolve quarantine" command.

### references/MAINTENANCE-AND-COMMANDS.md
- "new project" command: added profile.projectTemplate support.
  Default subfolders updated to 01_Briefing, 02_Research,
  03_Assets, 04_Deliverables, 05_Communication. User asked to
  confirm or customize per project.
- "maintenance" command: added step 0 to check _Inbox/ item count
  before any other maintenance action.
- "maintenance" command: added Quarentena/ check (step 4).
- "PARA status" command: added _Inbox/ count, Quarentena/ count
  and oldest item date, Legado-pre-organizacao/ count.
- Added "resolve quarantine" / "resolver quarentena" command.
- Watch folder note: recommend ~/Downloads and _Inbox/ as defaults.
- Periodic schedule: added Annual row (60 min) for full archive
  review, Legado purge, Resources consolidation, full reindex.

### SKILL.md
- version: 11.0.0 -> 12.0.0
- Section 7: added _Inbox/ creation on root setup. Underscore
  prefix documented. Registered as default watch folder.
- Section 8: added project template preference to onboarding.
  Added mediaPolicy field to .para-config.json documentation.
- Section 12: added step 2a for Areas vs Resources disambiguation.
  Files matching an Area alias but signalling learning content
  (notes, tutorial, resumo, curso, research, slides, cheatsheet)
  are redirected to 3-Recursos with user confirmation.
- Section 15: updated reference from PARA-METHOD-REFERENCE.md to
  RENAMING.md for naming conventions. Added _Inbox underscore rule.
  Added "preserve extension lowercase" to file rules.
- Section 18: added Quarantine option and reference to
  DEPENDENCY-CHECKS.md quarantine documentation.
- Section 20: added version series to duplicate definitions.
  Updated reference to DUPLICATE-DETECTION.md for new sections.
- Section 22: added originalName to manifest rename recording.
- Section 23: updated "maintenance" description to include _Inbox/.
  Updated "PARA status" to include _Inbox/, Quarentena/, Legado.
  Added "resolve quarantine" command.
  Updated "where was" / "trace" to mention originalName search.
- Section 24: added Annual maintenance cycle.
  Added Quarentena/ to maintenance check list.
- Section 28: added Quarentena/ edge case.
- Header reference: added RENAMING.md pointer alongside
  PARA-METHOD-REFERENCE.md.
- Line count: 510 lines (limit lifted from 500 to accommodate
  new Quarentena/ and _Inbox/ documentation; reference files
  carry the detail).

## [11.0.0] - 2026-03-10

### Compliance
- Rewrote description field: WHAT + WHEN + trigger phrases, under 1024 chars.
- Added license: MIT.
- Added compatibility field.
- Version as quoted string per spec.
- Removed user-invocable (not in spec).
- Tags as comma-separated string instead of YAML array.
- Moved README.md out of skill folder (repo-level only).
- Added agents/openai.yaml for Codex compatibility.

### New features
- Onboarding profile: default areas, interactive adjustment, aliases,
  sub-projects, interaction mode selection (Guided/Full control/Full trust).
- Operation modes: Import, Refine, Maintain. Auto-detected from
  source vs root relationship.
- Root setup: three scenarios (user defines, asks, delegates).
- Keyword Analysis (Step 2.3): tokenize filenames, cluster by frequency,
  cross-reference with profile aliases.
- Content Analysis (Step 2.4): three levels (0=name only, 1=shallow
  content, 2=deep content). Opt-in, privacy-first.
- Media Handling (Step 2.2): detect images, videos, screenshots, audio.
  EXIF extraction, audio/video metadata (ID3, Vorbis, MP4 atoms).
  Photos associated to projects by date/keyword correlation.
  User choice: PARA structure or standalone media structure.
- Duplicate Detection (Step 2.5): two-phase algorithm (size + hash).
  Three integration options: workflow step, on-demand command, combined.
  Cross-location dedup against existing PARA.
- Triage Session (Step 2.6): all uncertainties presented grouped by
  type before planning. Reduces mid-workflow interruptions.
- Folder handling rules: atomic, cohesive, mixed, empty, single-file.
- Searchable index: para-index.jsonl with per-file metadata, tags,
  category, description, correspondent, custom metadata.
- File traceability: "where was" and "trace" commands search all
  manifests and index for complete file history.
- PARA-CHANGELOG.md: human-readable cumulative log per execution.
- Watch folder command: monitor a folder for new files.
- Correspondent tracking: record origin/sender of files.
- Consistent mode: bias toward uniformity in batch classification.
- Refine mode: reorganize already-organized content within PARA.
- Tags in manifest/index for cross-category search.

### New reference files
- references/DUPLICATE-DETECTION.md
- references/KEYWORD-AND-CONTENT-ANALYSIS.md
- references/MEDIA-HANDLING.md

### Updated
- Manifest schemaVersion 7 -> 8. New types: deduplicate, refine-move.
- Config schema: profile field with areas, activeProjects, resources.
- Workflow: Steps 2.2, 2.3, 2.4, 2.5, 2.6 added between Discovery
  and Dependency Check.
- On-demand commands: find duplicates, where was, trace, update areas,
  refine, watch.
- Maintenance: re-run dedup on high-turnover folders, reindex suggestion.
- Edge cases: hard links as non-duplicates, Refine mode cross-category
  moves, watch mode rapid creation.

## [10.0.0] - 2026-03-09
- Added duplicate detection (three options).
- Root setup with three scenarios (define, ask, delegate).
- Scan modes integrated with dedup behavior.

## [9.0.0] - 2026-03-09
- Initial public release with full PARA workflow.
- Dependency checks, manifest, rollback, maintenance.
- Six reference files. Compliant with agentskills.io spec.
