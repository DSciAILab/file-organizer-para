# SESSION-STATE.md
# File Organizer PARA – Project Context & Session State

Last updated: 2026-03-10
Purpose: Resume development without losing context. Load this file at the start of any new conversation.

---

## 1. Project identity

- **Name:** file-organizer-para
- **Repository:** https://github.com/DSciAILab/file-organizer-para
- **Author:** Fernando Caravana
- **Organization:** DSciAILab
- **License:** MIT
- **Current version:** 11.0.0
- **Status:** Published, pending structure fix (DEC-027)

---

## 2. What this project is

An AI agent skill (SKILL.md format) that organizes local files and folders using Tiago Forte's PARA methodology. It runs inside any SKILL.md-compatible agent runtime (Claude, Gemini CLI, OpenAI Codex, Antigravity, Cursor, OpenClaw). No MCP server required. No external API calls. Everything runs locally on the user's filesystem.

---

## 3. Core design principles

1. Data preservation above all. Never delete user files.
2. Never move without plan + explicit approval.
3. The user decides. The skill recommends.
4. Every execution produces plan, manifest, report, and rollback.
5. Missing tool = NOT_CHECKED, never report OK without verifying.
6. Use the user's language (pt-BR and English supported).
7. Cross-platform (macOS, Linux, Windows, WSL) with documented capability matrix.

---

## 4. Architecture

```
file-organizer-para/          ← skill folder (agent reads this)
├── SKILL.md                  ← main instructions, <500 lines
├── CHANGELOG.md              ← version history
├── agents/
│   └── openai.yaml           ← Codex compatibility
└── references/               ← detailed specs (agent loads on demand)
    ├── PARA-METHOD-REFERENCE.md
    ├── DEPENDENCY-CHECKS.md
    ├── EXECUTION-STRATEGY.md
    ├── MANIFEST-SCHEMA.md
    ├── ROLLBACK-AND-RECOVERY.md
    ├── DUPLICATE-DETECTION.md
    ├── KEYWORD-AND-CONTENT-ANALYSIS.md
    ├── MEDIA-HANDLING.md
    └── MAINTENANCE-AND-COMMANDS.md

README.md                     ← repo-level, for humans on GitHub
LICENSE                        ← MIT
DECISION-LOG.md               ← this file (design decisions)
SESSION-STATE.md              ← project context for resuming sessions
```

---

## 5. Workflow pipeline

```
Step 0    Setup (root directory, onboarding profile, interaction mode)
Step 1    Source negotiation (validate origin, detect operation mode)
Step 2    Discovery (scan, count, exclude internals)
Step 2.2  Media Detection (images, videos, screenshots, EXIF)
Step 2.3  Keyword Analysis (filename tokenization, clustering, alias matching)
Step 2.4  Content Analysis (opt-in, levels 0/1/2)
Step 2.5  Deduplication (SHA-256, three scan modes)
Step 2.6  Triage Session (aggregate uncertainties, user resolves)
Step 3    Dependency Check (symlinks, hardlinks, configs, Git submodules)
Step 4    Plan (propose destinations, confidence levels, grouping)
Step 5    Confirm (show summary, require explicit "yes")
Step 6    Execute (lock, manifest, move/copy, verify, update index)
Step 7    Verify (confirm destinations, validate config edits)
Step 8    Report (summary, manual checklist, rollback script, PARA-CHANGELOG)
```

---

## 6. Operation modes

- **(a) Import:** Source outside PARA root. Full workflow. Default.
- **(b) Refine:** Source inside PARA root. Reorganize already-organized content. Skips root setup and structure creation.
- **(c) Maintain:** Source = root. General review, stale item detection, periodic cleanup.

---

## 7. Interaction modes

- **(a) Guided:** Agent auto-decides high-confidence items, asks about medium/low.
- **(b) Total Control:** Agent asks about every decision.
- **(c) Total Confidence:** Agent decides everything, shows final plan for approval.

---

## 8. Key features

- PARA classification with onboarding profile (default areas, user-defined areas/projects/resources, aliases, keywords)
- Filename keyword extraction and clustering
- Optional content analysis (three privacy levels)
- Media detection with EXIF parsing
- Duplicate detection (Quick/Safe/Deep)
- Interactive triage session
- Dependency safety checks
- Atomic folder detection (software projects)
- Mixed-content folder handling
- Manifest JSON with full audit trail
- Rollback scripts for every execution
- Searchable JSONL index across all runs
- Human-readable PARA-CHANGELOG.md
- Reverse file traceability ("where did X go?")
- Watch folder support
- Correspondent tracking
- Consistent mode bias
- Cross-platform capability matrix

---

## 9. Compliance targets

- agentskills.io specification (name, description, license, compatibility, metadata)
- OpenAI Codex skill format (agents/openai.yaml)
- Gemini CLI / Antigravity skill format
- Anthropic/Claude skill authoring best practices
- Progressive disclosure (<500 lines SKILL.md, references on demand)

---

## 10. Pending tasks

| # | Task | Priority | Status |
|---|------|----------|--------|
| 1 | Restructure repo: move skill files into `file-organizer-para/` subfolder | Critical | Pending |
| 2 | Replace `YOUR-USERNAME` with `DSciAILab` in README.md (3 occurrences) | Critical | Pending |
| 3 | Add DECISION-LOG.md and SESSION-STATE.md to repo | Medium | Pending |
| 4 | Test skill with Claude Code (real execution on sample folder) | High | Not started |
| 5 | Test skill with Gemini CLI (real execution on sample folder) | High | Not started |
| 6 | Test skill with OpenAI Codex (real execution on sample folder) | Medium | Not started |
| 7 | Submit to LobeHub Skills Marketplace | Low | Not started |
| 8 | Submit to MCPMarket | Low | Not started |
| 9 | Localization (pt-BR, es, fr, de) | Low | Not started |
| 10 | GUI/TUI preview of plan before execution | Low | Not started |
| 11 | Audio file metadata extraction (ID3, Vorbis, MP4 atoms) | Low | Not started |

---

## 11. Decisions index

Full decision history in DECISION-LOG.md. Key decisions:

- DEC-001 to DEC-010: Foundation (PARA method, skill format, safety rules, dependency checks, manifests, rollback, execution lock, cross-platform).
- DEC-011 to DEC-016: Analysis pipeline (flexible root, duplicate detection, keyword analysis, content analysis, media handling, project-linked photos).
- DEC-017 to DEC-020: Traceability and UX (cumulative history, reverse lookup, folder handling, triage session).
- DEC-021 to DEC-023: Methodology refinement (PARA gaps, onboarding profile, area vs project clarification).
- DEC-024 to DEC-027: Final polish (competitive research, compliance audit, refine mode, repo structure fix).

---

## 12. How to use this file

When starting a new conversation about this project:

1. Share this file (SESSION-STATE.md) with the assistant.
2. Share DECISION-LOG.md if discussing design rationale.
3. Point to the repository: https://github.com/DSciAILab/file-organizer-para
4. State what you want to work on next.

The assistant will have full context without needing to re-derive 27 decisions from scratch.

---

## 13. Competitors analyzed

| Tool | Type | Key insight absorbed |
|------|------|---------------------|
| AI File Sorter | Open-source, Qt6 GUI | Whitelist of categories → our onboarding. Consistent mode. Audio/video metadata. |
| Claw Drive | Bash CLI + JSONL | Persistent searchable index. Re-indexation. Custom metadata (expiry, policy number). |
| Paperless-ngx | Self-hosted DMS | Correspondent tracking. Auto-tagging by content. |
| Hazel | Mac, rule-based | Watch folder concept. Reliable rule execution. |
| Sparkle | Mac, GPT-4 cloud | Validated need for onboarding to avoid generic classification. |
| Local File Organizer | Python, local LLM | Privacy-first approach. Image analysis with LLaVA. |
| Wisfile | Mac/Windows, local AI | Content-based renaming. Free. |
| LobeHub skills | Marketplace | No PARA-based organizer exists yet. Market gap confirmed. |

---

## 14. Technical constraints

- SKILL.md must stay under 500 lines.
- Front-matter `name` field: lowercase, hyphens, 1-64 chars, must match folder name.
- Front-matter `description`: max 1024 chars, focus on WHAT + WHEN + triggers.
- `metadata.version` must be a quoted string, not a number.
- `metadata.tags` must be a single string (comma-separated), not an array.
- No `user-invocable` field (not part of agentskills.io spec).
- Reference files loaded on demand by the agent, not eagerly.
- All text artifacts use UTF-8 without BOM.