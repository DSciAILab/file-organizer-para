# DECISION-LOG.md

## File Organizer PARA – Decision Log

Every design decision made during development, in reverse chronological order.
Use this to understand WHY things are the way they are.

---

## Session 2026-03-15 (v11 → v12)

### DEC-034 | Naming conventions consolidation

- **Decision:** Extract all naming rules and batch rename procedures into a dedicated `references/RENAMING.md`.
- **Reason:** Naming conventions were scattered; a single source of truth improves consistency.
- **Status:** Done (v12.0.0).

### DEC-033 | Version series vs Duplicates

- **Decision:** Files with version markers (v1, v2, draft, approved) are now classified as "Version Series" and protected from duplicate archival.
- **Reason:** Versioning is intentional; treating it as a duplicate leads to data loss of history.
- **Status:** Done (references/DUPLICATE-DETECTION.md).

### DEC-032 | Photos as a Library, not Archive

- **Decision:** Changed default photo destination from `4-Arquivo/Fotos/` to `5-Fotos/`. Standalone library structure is now the default.
- **Reason:** PARA Archive is for inactive items; Photos are a continuous memory library. Putting them in Archive conflicted with PARA semantics.
- **Status:** Done (references/MEDIA-HANDLING.md).

### DEC-031 | Automated Areas vs Resources disambiguation

- **Decision:** Added Step 2a. Files matching an Area alias but containing "notes", "tutorial", "resumo" etc. are redirected to `3-Recursos` with confirmation.
- **Reason:** Users often confuse active responsibilities (Areas) with reference material (Resources) on the same topic.
- **Status:** Done (SKILL.md Section 12).

### DEC-030 | Quarantine system

- **Decision:** Implementation of `Quarentena/` folder for items with broken dependencies (e.g., dead symlinks) that the user cannot resolve immediately.
- **Reason:** Prevents stalling the workflow; allows deferred resolution via "resolve quarantine" command.
- **Status:** Done (references/DEPENDENCY-CHECKS.md).

### DEC-029 | Inbox visibility

- **Decision:** Use underscore prefix for `_Inbox/` at root level.
- **Reason:** Ensures it stays at the top of directory listings for high visibility.
- **Status:** Done (SKILL.md Section 7).

### DEC-028 | Annual maintenance cycle

- **Decision:** Added Annual schedule (60 min) for deep review, legacy purge, and full reindex.
- **Status:** Done (Section 24).

---

## Session 2026-03-10 (v9 → v11)

### DEC-027 | Repository structure fix
- **Decision:** Move all skill files into `file-organizer-para/` subfolder. Keep README.md and LICENSE at repo root.
- **Reason:** agentskills.io spec requires folder name = front-matter `name` field. Flat structure breaks agent discovery.
- **Status:** Pending (commands provided, not yet pushed).

### DEC-026 | Compliance audit against 5 guideline groups
- **Decision:** Audit SKILL.md against agentskills.io, OpenAI Codex, Gemini CLI, Anthropic/Claude, and prompt-engineering best practices. Apply all fixes in v11.
- **Changes:** Rewrote `description` to include trigger phrases (≤1024 chars). Added `license: MIT`. Added `compatibility`. Quoted `metadata.version` as string. Changed `metadata.tags` from array to single string. Removed `user-invocable`. Created `agents/openai.yaml`.
- **Status:** Done (v11.0.0).

### DEC-025 | Refine mode for already-organized folders
- **Decision:** Add operation mode (c) Refine. When source is inside PARA root, skip full import. Focus on sub-folder improvement, cross-category moves, name standardization, internal dedup, tag enrichment.
- **Reason:** User asked "what if I want to improve organization of a folder that was already reorganized?"
- **Status:** Done (Section 6, SKILL.md v11).

### DEC-024 | Competitive research and feature absorption
- **Decision:** Deep search of 8 competitors (Sparkle, AI File Sorter, Local File Organizer, Claw Drive, Paperless-ngx, Wisfile, Hazel, LobeHub skills). Identified 6 features to absorb.
- **Absorbed:** Watch folder (Hazel), searchable JSONL index (Claw Drive), audio/video metadata (AI File Sorter), correspondent tracking (Paperless-ngx), consistent mode (AI File Sorter), re-index (Claw Drive).
- **Differentiators confirmed:** PARA methodology, dependency checks, manifest + rollback, onboarding profile, interactive triage, cross-platform capability matrix.
- **Status:** Watch folder and JSONL index done in v11. Audio/video metadata and correspondent tracking documented. Re-index in maintenance commands.

### DEC-023 | Areas vs projects clarification
- **Decision:** Ongoing programs without end date (e.g., SJJP) are Areas. Time-bound efforts within them (e.g., SJJP-Evento-anual) are sub-projects. Profile schema supports `subProjects` inside areas.
- **Reason:** User works in a permanent program (SJJP) and needed clarity on classification.
- **Status:** Done (Section 8, onboarding profile schema).

### DEC-022 | Onboarding profile with default areas
- **Decision:** Before scanning, present default areas (Health, Finances, Career, Family, Home, Personal Development, Hobbies, Work) and ask user to adjust. For each item, ask if ongoing (area) or has deadline (project). Collect aliases and keywords per area. Store in `.para-config.json`.
- **Reason:** Knowing the user's mental map before scanning reduces ambiguous triage questions from ~30 to ~5.
- **Status:** Done (Section 8, SKILL.md v11).

### DEC-021 | Critical review of PARA method gaps
- **Decision:** Identified 5 weaknesses. (1) Doesn't scale to 50k+ files without triage-first stage. (2) Project/area ambiguity needs interactive glossary. (3) Transversal files need complementary tag system. (4) Media handling needs optional separate hierarchy. (5) No automatic maintenance scheduling.
- **Reason:** Self-audit before finalizing v11.
- **Status:** All 5 addressed in v11.

### DEC-020 | Interactive triage session (Step 2.6)
- **Decision:** After discovery sub-steps (2.2-2.5), aggregate all uncertainties and present in a single triage session grouped by type: folders, classification, photos, duplicates. Three interaction modes: Guided, Total Control, Total Confidence.
- **Reason:** User asked for confirmation points where agent is uncertain. Single session avoids interruption fatigue.
- **Status:** Done (Section 11, Step 2.6, SKILL.md v11).

### DEC-019 | Folder handling rules
- **Decision:** Three rules. (1) Atomic folders (.git, package.json, etc.) always move as whole unit. (2) Cohesive folders (keyword analysis says content is related) propose moving whole. (3) Mixed-content folders: ask user (classify individually, move whole, or skip). Sub-rules: report empty folders without moving; single-file folders suggest moving file instead of wrapper.
- **Status:** Done (SKILL.md v11).

### DEC-018 | Reverse file traceability command
- **Decision:** On-demand command "where did [name] go?" / "where was [name]?" / "trace [name]". Searches all manifests by partial token match on sourcePath and destinationPath.
- **Reason:** User wants to find where a file was before or after organization.
- **Status:** Done (references/MAINTENANCE-AND-COMMANDS.md).

### DEC-017 | Cumulative history across executions
- **Decision:** Three additions. (1) Manifest index at `<root>/.para-manifest-history/index.json`. (2) File history command searching all manifests. (3) Human-readable `PARA-CHANGELOG.md` updated each run.
- **Reason:** Individual manifests existed but no cross-execution view.
- **Status:** Done (references/MANIFEST-SCHEMA.md, MAINTENANCE-AND-COMMANDS.md).

### DEC-016 | Photos belonging to projects
- **Decision:** Photos follow the PARA category of their context. Project photos go inside the project folder. Loose photos use heuristic chain: (1) already in project folder → auto-associate, (2) keyword/content match → suggest, (3) EXIF date matches project period → suggest (low confidence), (4) no match → default media rules (screenshots → Resources, dated → Archive/YYYY/MM, undated → Legado).
- **Status:** Done (references/MEDIA-HANDLING.md).

### DEC-015 | Media handling (photos/videos)
- **Decision:** Detect image/video/audio by extension. Classify screenshots → Resources, photos with EXIF → Archive/Fotos/YYYY/MM, photos without EXIF → Legado, videos → same yearly grouping. No AI vision labeling. Use exiftool/mdls/identify if available, otherwise NOT_CHECKED.
- **Status:** Done (references/MEDIA-HANDLING.md).

### DEC-014 | Content analysis opt-in levels
- **Decision:** Level 0 (default): filename + extension + metadata only. Level 1 (shallow): first 50 lines of text files when name-based confidence is low. Level 2 (deep): full content read, explicit user permission required.
- **Reason:** Privacy and performance trade-off. Reading 500 files feasible, 5000 not.
- **Status:** Done (references/KEYWORD-AND-CONTENT-ANALYSIS.md).

### DEC-013 | Keyword analysis from filenames (Step 2.3)
- **Decision:** Tokenize filenames by separators (_, -, space, dot, camelCase). Count frequency, discard singletons and filesystem stopwords. Cluster files sharing tokens with freq ≥3. Cross-reference with profile aliases. Present clusters to user with suggested tags.
- **Reason:** User noticed SJJP appearing in multiple filenames gave a classification hint.
- **Status:** Done (references/KEYWORD-AND-CONTENT-ANALYSIS.md).

### DEC-012 | Duplicate detection (three options combined)
- **Decision:** Implement all three: (A) workflow step between scan and plan, (B) on-demand command, (C) both. Algorithm: group by size → SHA-256 hash → present groups → user decides (keep newest, keep largest, keep specific, archive rest). Three scan modes: Quick (size only), Safe (size + partial hash), Deep (full hash).
- **Status:** Done (references/DUPLICATE-DETECTION.md).

### DEC-011 | Flexible root setup (three scenarios)
- **Decision:** (a) User specifies root explicitly, (b) user specifies only source, agent asks about root and suggests default by OS, (c) user delegates, agent applies heuristic (check global pointer, then OS default, then ask). All scenarios validate existence, write permission, free space, not inside system directory.
- **Status:** Done (Section 7, SKILL.md v11).

### DEC-010 | PARA method as organizational backbone
- **Decision:** Use Tiago Forte's PARA (Projects, Areas, Resources, Archive) as the classification framework. Four top-level folders numbered 1-4. Plus Legado-pre-organizacao for unclassifiable items and Quarentena for dependency-broken items.
- **Reason:** Recognized methodology, action-oriented, scales well for personal and professional files.
- **Status:** Done (core of the skill since v1).

### DEC-009 | Dependency checks before any move
- **Decision:** Check symlinks, hard links, config file references, .git submodules, PATH references before moving files. Three scan modes (Quick/Safe/Deep). Flagged items go to Quarentena or require user decision.
- **Status:** Done (references/DEPENDENCY-CHECKS.md).

### DEC-008 | Manifest JSON + rollback scripts
- **Decision:** Every execution produces a JSON manifest (source, destination, hash, timestamp, operation type, status) and a shell script that reverses all moves. Schema version 7 with support for deduplicate operations.
- **Status:** Done (references/MANIFEST-SCHEMA.md, ROLLBACK-AND-RECOVERY.md).

### DEC-007 | Execution lock to prevent concurrent runs
- **Decision:** Create `.para-lock.json` before any write. Heartbeat every 60s. Stale detection by timeout. Never execute without lock.
- **Status:** Done (Section 10, SKILL.md v11).

### DEC-006 | Never delete user files
- **Principle:** The skill moves, copies, renames. It never deletes. Even duplicate archiving moves files to a dedup folder.
- **Status:** Permanent rule (Section 1, principle 3).

### DEC-005 | User always confirms before execution
- **Principle:** Plan is always shown. Execution requires explicit "yes". Three interaction modes control how much the agent asks during analysis, but final plan approval is always required.
- **Status:** Permanent rule (Section 1, principle 2).

### DEC-004 | Cross-platform support with capability matrix
- **Decision:** Document what works on macOS/Linux vs Windows vs WSL. Three levels: SUPPORTED, BEST_EFFORT, NOT_CHECKED. Never claim a check was done if the tool wasn't available.
- **Status:** Done (Section 4, SKILL.md v11).

### DEC-003 | SKILL.md under 500 lines
- **Decision:** Keep main instruction file under 500 lines per agentskills.io progressive disclosure guidance. Detailed algorithms, schemas, and edge cases go in references/ folder.
- **Status:** Done (SKILL.md ≈490 lines).

### DEC-002 | Agent Skill format (SKILL.md + references/)
- **Decision:** Use the agentskills.io standard. YAML front-matter with name, description, license, compatibility, metadata. Markdown body with instructions. References folder for deep-dive documents.
- **Status:** Done.

### DEC-001 | Project inception
- **Decision:** Build an AI agent skill for file organization using PARA method, targeting Claude, Gemini CLI, OpenAI Codex, and Antigravity runtimes.
- **Author:** Fernando Caravana
- **Status:** Done (v11.0.0 published at github.com/DSciAILab/file-organizer-para).