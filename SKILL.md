---
name: file-organizer-para
description: >-
  Organizes local files and folders using Tiago Forte's PARA methodology
  with dependency safety checks, duplicate detection, keyword analysis,
  content analysis, media handling, execution planning, rollback, and
  ongoing maintenance. Use when the user asks to organize files, clean up
  folders, sort downloads, set up PARA structure, find duplicates, rename
  files, archive old projects, or maintain an existing PARA setup.
  Trigger phrases include: "organizar meus arquivos", "limpar meu desktop",
  "organizar minha pasta de downloads", "encontrar duplicados",
  "montar estrutura PARA", "organize my files", "clean up my desktop",
  "sort my downloads", "find duplicates", "set up PARA folders",
  "archive old projects", "rename my files", "any repeated files?",
  "improve organization of [folder]", "refine [folder]".
license: MIT
compatibility: >-
  Requires local filesystem read/write access. Works with any agent
  that supports the Agent Skills standard (Claude Code, Codex CLI,
  Gemini CLI, Antigravity, Cursor, OpenClaw). No MCP server required.
  macOS, Linux, Windows, WSL.
metadata:
  author: "Fernando Caravana"
  version: "11.0.0"
  tags: "file-organization, para-method, productivity, filesystem, deduplication"
---

# PARA File Organizer

Organizes local files using Tiago Forte's PARA methodology.
Never break references, never move without verification, never
execute without explicit approval.

For complete PARA definitions, classification examples, naming
conventions and methodological references, see
[references/PARA-METHOD-REFERENCE.md].

## 1. Operational principles

1. Preserving data comes before organizing.
2. Never move, rename or archive without plan and approval.
3. Never delete user files.
4. Cross-filesystem moves: remove origin only after copy + verify.
5. Apply PARA: Projects, Areas, Resources, Archive.
6. Impossible check = NOT_CHECKED. Never report OK without verifying.
7. The user decides. The skill recommends.
8. Use the user's language.
9. Every execution produces plan, manifest, report and rollback.
10. Resolve inconsistencies between disk, lock and manifest before
    any write.

## 2. Rule precedence

1. Avoid data loss and dependency breakage
2. Respect explicit user decision
3. Preserve software project atomicity
4. Resolve dependency and link risks
5. Maintain PARA coherence
6. Apply naming conventions
7. Cleanup and maintenance

## 3. Scope

- Local filesystem only.
- Only capabilities available in the runtime.
- Missing tool = NOT_CHECKED.
- Persist state in: global pointer, root config, lock, manifest, index.
- Produce: human-readable output, structured manifest, rollback.
- Internal artifacts (.para-config.json, .para-lock.json, .para-temp/,
  .para-manifest-history/, para-manifest-*.json, para-rollback-*,
  para-index.jsonl) never enter PARA classification or plan.
- All text artifacts use UTF-8 without BOM.

## 4. Capabilities

Classify as SUPPORTED, BEST_EFFORT or NOT_CHECKED.

macOS/Linux: inventory, folders, symlinks, hard links, configs with
absolute path = SUPPORTED. PATH, cron, launchd, systemd, Finder
aliases, network filesystem, cloud placeholders, EXIF metadata,
audio/video metadata = BEST_EFFORT.

Windows: inventory, folders = SUPPORTED. Symlinks, hard links, .lnk,
schtasks, registry PATH, cloud placeholders, ADS, ACLs, EXIF,
audio/video metadata = BEST_EFFORT.

WSL: Linux filesystem and /mnt = SUPPORTED. Registry, .lnk,
schtasks = NOT_CHECKED unless bridge available.

Hard rule: never claim Finder, registry, Dock, sidebar, schtasks
or shortcuts were checked unless the actual tool confirmed it.

## 5. State and persistence

Global pointer:
- macOS/Linux: ~/.config/openclaw/para-root.json
- Windows: %APPDATA%/OpenClaw/para-root.json

Root config: <root>/.para-config.json
- mode: "single" (trees=[]) or "separate" (trees with >= 2 unique names)
- categoryFolderFormat: "number-hyphen" or "number-dot-space"
- maxDepthBelowCategoryRoot: 3
- Alert if trees > 5 (dispersed structure)
- profile: areas, activeProjects, resources with aliases and keywords

Searchable index: <root>/para-index.jsonl
- One JSON line per organized file.
- Fields: path, originalPath, hash, tags, category, area, project,
  description, correspondent, customMetadata, organizedAt, executionId.
- Used for traceability, cross-location dedup and semantic search.

For complete config, lock, manifest and index schemas, see
[references/MANIFEST-SCHEMA.md].

## 6. Operation modes

Detected automatically based on source vs root relationship:

(a) Import: source outside root PARA. Full workflow. Default.
(b) Refine: source inside root PARA. Reorganization of already
    organized content. Skips root setup and structure creation.
    Focuses on: sub-folder improvement, cross-category moves,
    name standardization, internal dedup, tag enrichment.
(c) Maintain: source = root. General review, stale item detection,
    periodic cleanup. See Section 24.

The user can also request a mode explicitly.

## 7. Root directory setup (Step 0)

The user may:
(a) specify the root explicitly ("root at ~/Documents/PARA"),
(b) specify only the source folder without indicating root,
(c) delegate the decision ("you decide" / "escolhe o melhor lugar").

For (a): validate existence, write permission and space. Use the
exact path provided.

For (b): ask where the root should be. Suggest default by OS.
Accept response or alternative.

For (c): apply heuristic:
1. If global pointer exists with valid activeRoot, propose it.
2. Otherwise propose by OS:
   - macOS: ~/Documents/PARA
   - Linux: ~/Documents/PARA or ~/PARA if Documents missing
   - Windows: %USERPROFILE%\Documents\PARA
3. Show proposal and WAIT for explicit confirmation.

Mandatory validation (all scenarios):
- Path exists or can be created.
- Write permission confirmed.
- Free space >= diskSafetyMarginPercent.
- Path not inside system directory (/System, /Library, /usr,
  C:\Windows, C:\Program Files).
- Path not inside .git, node_modules or similar.

## 8. Onboarding profile (Step 0 continued)

First execution, after root is defined:

Present default areas and ask the user to adjust:
Health, Finances, Career, Family, Home, Personal Development,
Hobbies, Work.

For each item the user mentions, ask:
"Does [item] have an end date or is it ongoing?"
- Ongoing = Area.
- Has deadline = Project.
- Ongoing with sub-projects = Area with subProjects.

Then ask about active projects and resource topics.

Store in .para-config.json under "profile" with aliases and
keywords per item. Aliases feed Keyword Analysis (Step 2.3).

Interaction mode selection:
(a) Guided: agent decides high-confidence items, asks the rest.
(b) Full control: agent asks every decision.
(c) Full trust: agent decides everything, shows final plan.

Subsequent executions: load profile, ask "Changed anything since
last time?" If yes, update. If no, proceed.

For profile schema and examples, see [references/PARA-METHOD-REFERENCE.md].

## 9. Source negotiation (Step 1)

Distinguish root PARA (destination) from source directory (origin).
- Explicit path: validate existence and accessibility.
- "desktop"/"downloads"/"documents": detect automatically.
- Generic: ask which directory.
- Never assume source without confirmation.
- Validate read on source. No read = abort.
- Validate write on source when needed. No write = offer copy_only.

Overlap protection:
- Root inside source: exclude root from scan.
- Source = root: treat as Maintain mode.
- Source inside root: treat as Refine mode.
- Prevent recursion and self-nesting.

## 10. Execution lock

Create <root>/.para-lock.json before any write.
Update heartbeat: every operation or every 60s, whichever comes first.
Stale: updatedAt > heartbeatTimeoutMinutes. Long total duration =
alert, not stale.
Lock not creatable (permission, disk full) = abort, never execute
without lock.
Reconciliation: require coherence of executionId, manifestPath,
root and source between lock and manifest.
Remove lock only on: success, rollback completed, safe abort.

## 11. Mandatory workflow

Step 0.  Setup: read config, check lock, check incomplete manifests,
         onboarding profile (first run or if user requests update).
Step 1.  Source: negotiate origin, validate permissions, detect
         operation mode (Import/Refine/Maintain).
Step 2.  Discover: scan, count, exclude root if inside source,
         exclude internal artifacts, never follow symlinks recursively.
         >10,000 items: pause and ask. >10 min: pause.
Step 2.2 Media Detection: identify images, videos, screenshots,
         audio. Extract EXIF/metadata when tools available.
         See [references/MEDIA-HANDLING.md].
Step 2.3 Keyword Analysis: tokenize filenames, cluster by recurring
         tokens, cross-reference with profile aliases.
         See [references/KEYWORD-AND-CONTENT-ANALYSIS.md].
Step 2.4 Content Analysis (opt-in): read content of ambiguous files
         to improve classification. Three levels (0/1/2).
         See [references/KEYWORD-AND-CONTENT-ANALYSIS.md].
Step 2.5 Deduplication (optional): detect duplicates by scan mode
         or explicit request. Present report, wait for decision.
         See [references/DUPLICATE-DETECTION.md].
Step 2.6 Triage Session: present all accumulated uncertainties
         grouped by type. User resolves before planning.
Step 3.  Dependency check: run per mode (Quick/Safe/Deep), generate
         report, resolve flagged items.
         See [references/DEPENDENCY-CHECKS.md].
Step 4.  Plan: propose destination, indicate rename, confidence,
         dependency. Group repeated decisions. See Section 13.
Step 5.  Confirm: show plan summary. Never execute without "yes".
         See Section 14.
Step 6.  Execute: create lock, manifest, PARA structure, move/copy
         with verification, rename, update approved configs, record
         each operation, update heartbeat, update index.
         See [references/EXECUTION-STRATEGY.md].
Step 7.  Verify: confirm existence at destination, absence at origin
         when applicable, validate config edits, fill completedAt.
Step 8.  Report: full report, manual checklist, rollback script,
         update PARA-CHANGELOG.md, remove lock, ask for adjustments.

## 12. PARA classification

Decision tree:
1. Specific result with deadline? -> 1-Projetos / 1-Projects
2. Ongoing responsibility? -> 2-Areas
3. Reference or learning? -> 3-Recursos / 3-Resources
4. Inactive or completed? -> 4-Arquivo / 4-Archive

Profile areas and aliases boost classification confidence.
Low confidence for Work vs Personal: ask.
Ambiguous: propose batch or triage to Legacy-pre-organization.

For detailed definitions and examples, see
[references/PARA-METHOD-REFERENCE.md].

## 13. Planning

Required table:

| # | Origin | Destination | Action | Scope | Confidence | Dependency |
|---|--------|-------------|--------|-------|------------|------------|

Actions: Move, Move+Rename, Copy+Verify+Remove, Copy_only, Skip,
  Ask user, Archive-duplicate, Refine-move (for Refine mode).
Confidence: High, Medium, Low.
Dependency: No, Yes:<type>, NOT_CHECKED:<type>.

Summary: folders to create, items to move, rename, copy+verify,
copy_only, dependencies resolved, skipped, ambiguous, NOT_CHECKED,
duplicates found and action chosen.

## 14. Confirmation

Show before executing:

PARA ORGANIZATION PLAN
- Source, Root, Mode (Import/Refine/Maintain), Scan mode
- Folders to create: N
- Items to move: N, rename: N, copy+verify: N, copy_only: N
- Skipped: N, ambiguous: N, NOT_CHECKED: N
- Cross-filesystem: N, network: N, dependencies: N
- Duplicates: N groups, N files, N bytes recoverable
- Media: N images, N videos, N audio, N screenshots
- Space required vs available
- Batch write: yes/no

"Proceed? (yes / no / show details / edit)"

## 15. Naming conventions

Folders: YYYY-MM_Name (projects), Descriptive-name (areas/resources),
NN_Name (subfolders).
Files: YYYY-MM-DD_Description.ext, version -v1/-v2, never FINAL.
Hyphens between words, ASCII safe, preserve extension.
Collision: -duplicata-N. Windows: validate reserved names.
Non-latin Unicode: FLAGGED, never auto-rename.
Legacy-pre-organization: no automatic rename.

For complete conventions, see [references/PARA-METHOD-REFERENCE.md].

## 16. Scan modes

Quick: inventory, sizes, large files, symlinks.
  Duplicates: off. Agent offers to enable.
  Content analysis: off.
Safe (default): + software projects, hard links, configs, locks,
  cross-fs, space, permissions, network, cloud, metadata.
  Duplicates: phase 1 (size grouping). If candidates, ask for hash.
  Content analysis: level 0.
Deep: + PATH, shell rc, shortcuts, scheduled tasks.
  Duplicates: phase 1 + phase 2 (hash) automatic.
  Content analysis: level 1 for low-confidence items.
If source appears dev/automation, recommend Deep.

## 17. Ignore

Standalone: .DS_Store, thumbs.db, desktop.ini, .Spotlight-V100, etc.
Internal: .para-config.json, .para-lock.json, .para-manifest-history/,
  .para-temp/, para-manifest-*.json, para-rollback-*, para-index.jsonl,
  PARA-CHANGELOG.md.
Hidden: do not move in isolation, inspect in dependency check.
Software projects: .git, node_modules, .venv, etc = move only with parent.
.env: never ignore for dependency check.

## 18. Dependency check

Mandatory before plan. Status: OK, FLAGGED, NOT_CHECKED, ERROR.

Main checks: symlinks, hard links, software projects, configs with
absolute path, cloud boundary, files >1GB, read-only, in-use,
cross-filesystem, space, case-collision, path length, reserved names,
permissions, network filesystem, hash when required.

For complete check list and options, see
[references/DEPENDENCY-CHECKS.md].

## 19. Folder handling rules

In order of precedence:
1. Software project markers detected: move folder as atomic unit.
2. Keyword/content analysis shows cohesive group: suggest moving
   whole folder. Ask user to confirm.
3. Mixed content detected: ask user: (a) classify individually,
   (b) move whole folder to one destination, (c) skip folder.
4. Empty folder: report, do not move. Ask if user wants to delete.
5. Single file inside folder: suggest moving the file, not wrapper.

## 20. Duplicate detection

Exact duplicate: identical SHA-256 hash regardless of name or location.
Probable duplicate: same size + same extension + similar name
(Levenshtein <= 3), different hash. Report only.

Two-phase algorithm: size grouping then hash comparison.
Integration with scan modes: see Section 16.
Available as workflow step (2.5) and on-demand command (Section 23).

For complete algorithm, report format, manifest schema, performance
guidelines and safety rules, see [references/DUPLICATE-DETECTION.md].

## 21. Execution

Same-filesystem local: native rename (except: network, cloud boundary,
  hard links preserving count, explicit copy+verify request).
Cross-filesystem: copy -> verify (size + hash when required) ->
  remove origin. Verification failure = keep origin.
Collision: -duplicata-N, never overwrite.
Config edits: .bak backup before, validate syntax after.
Software projects: entire folder as unit.
Atomic write: temp file -> flush/fsync -> rename.
Batch write: after batchWriteThreshold, write in batchWriteSize lots.
Network: never trust rename, always copy+verify+remove.
Index update: after each successful operation, append line to
  para-index.jsonl with file metadata, tags and category.
PARA-CHANGELOG.md: append execution summary after completion.

For complete strategy, see [references/EXECUTION-STRATEGY.md].

## 22. Manifest

Every execution generates JSON manifest (schemaVersion 8).
Operations: type (move/copy/copy_only/deduplicate/refine-move),
status (planned -> in_progress -> copied -> verified ->
source_removed -> completed / failed / rolled_back).
Rename is attribute (renamed=true), not type.
Local move via rename: planned -> completed (intentional).
completedAt only when all operations finish.
Deduplicate: records kept, archived, groupHash.
Index: para-index.jsonl updated per operation for traceability.

For complete JSON schema and state rules, see
[references/MANIFEST-SCHEMA.md].

## 23. On-demand commands

- "new project" / "novo projeto": create folder with recommended subs.
- "new area" / "nova area": create area folder.
- "archive project" / "arquivar projeto": move to Archive.
- "maintenance" / "manutencao": run periodic review.
- "PARA status" / "status PARA": show config and counts.
- "find duplicates" / "encontrar duplicados": run duplicate detection
  independently. Works even without configured root.
  See [references/DUPLICATE-DETECTION.md].
- "where was [name]?" / "onde estava [nome]?": search all manifests
  and index for file history. Partial name match supported.
- "trace [name]" / "rastrear [nome]": show full trajectory of a file
  across all executions.
- "update areas" / "atualizar areas": modify onboarding profile.
- "refine [folder]" / "melhorar [pasta]": enter Refine mode for a
  specific folder inside the PARA root.
- "watch [folder]": enable watch mode for a folder. Agent reports
  new files periodically and suggests classification.

For details, see [references/MAINTENANCE-AND-COMMANDS.md].

## 24. Maintenance

Weekly (5 min): inactive projects >30 days, suggest archiving.
Monthly (15 min): areas/resources unchanged >90 days, suggest review.
Check Legacy-pre-organization. Re-run dependency check if relevant.
Re-run duplicate detection on high-turnover folders if user accepts.
Suggest index reindex for enriching descriptions of older entries.

## 25. Failure during execution

Individual: mark failed, write manifest, ask retry/skip/pause/abort.
Systemic (3+ consecutive): auto-pause, ask.
Critical (filesystem inaccessible): write best-effort, keep lock.
Config edit: offer backup restore, add to checklist.

## 26. Rollback

Reverse order of completed operations.
move: move back, restore name if renamed.
copy: if source exists, remove destination. If not, move back.
copy_only: remove destination (source intact).
deduplicate: move archived files back to original location.
refine-move: move back to previous location within PARA.
configEdits: restore backup.
Collision on rollback: never overwrite, use -rollback-collision-N.
Created folders: remove only if empty.
Partial rollback: record, continue, mark partial.
Script .sh/.ps1 generated after each execution.
Interactive rollback available via skill.

For complete procedures, see [references/ROLLBACK-AND-RECOVERY.md].

## 27. Report

PARA FILE ORGANIZER - REPORT
Date, Execution ID, Source, Root, Mode, Scan Mode, Format.
HONEST STATUS: completed, skipped, failed, NOT_CHECKED.
Metadata: preserved, best_effort, not_checked, failed.
Sections: structure created, files moved/renamed, config edits,
dependencies resolved, duplicates found and action taken,
media organized, keyword clusters used, ambiguous, skipped,
NOT_CHECKED, errors, statistics, rollback info, manual checklist.

Manual post-move checklist saved as
<root>/para-manual-checklist-YYYY-MM-DD-HHMM.md.

PARA-CHANGELOG.md appended with execution summary.

## 28. Edge cases

Empty folders, files without extension, circular symlinks, ambiguous
encoding, >10GB, >10,000 items in one folder, files in use, corrupted
manifest, revoked permissions, exhausted space, cloud sync conflicts,
unknown or old manifest schema, duplicates with hard links (same
inode = not duplicate, report as hard link), Refine mode moving files
between PARA categories, watch mode detecting rapid file creation.

For detailed handling, see [references/ROLLBACK-AND-RECOVERY.md]
and [references/EXECUTION-STRATEGY.md].