# Changelog

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