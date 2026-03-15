# SESSION-STATE.md
# File Organizer PARA – Project Context & Session State

Last updated: 2026-03-15
Purpose: Resume development without losing context. Load this file at the start of any new conversation.

---

## 1. Project identity

- **Name:** file-organizer-para
- **Repository:** https://github.com/DSciAILab/file-organizer-para
- **Author:** Fernando Caravana
- **Organization:** DSciAILab
- **License:** MIT
- **Current version:** 12.0.0
- **Status:** v12.0.0 published. All repository structure and naming issues resolved.

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
├── SKILL.md                  ← main instructions, < 515 lines
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
    ├── MAINTENANCE-AND-COMMANDS.md
    └── RENAMING.md           ← naming conventions (new in v12)

README.md                     ← repo-level, for humans on GitHub
LICENSE                       ← MIT
DECISION-LOG.md               ← design decisions history
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
Step 2.5  Deduplication (SHA-256, three scan modes, version series protection)
Step 2.6  Triage Session (aggregate uncertainties, user resolves)
Step 3    Dependency Check (symlinks, hardlinks, configs, Git submodules)
Step 3.2  Quarantine (move items with broken dependencies to Quarentena/)
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

## 8. Key features (v12.0.0)

- PARA classification with onboarding profile (default areas, aliases, keywords)
- **Automatic Areas vs Resources disambiguation** based on content markers
- Filename keyword extraction and clustering
- Optional content analysis (three privacy levels)
- Media detection (EXIF, screenshots, date-based grouping)
- **Library-mode for Photos** (defaults to `5-Fotos/` instead of `4-Archive/`)
- Duplicate detection with **Version Series protection** (v1, v2, draft, etc.)
- **Quarantine system** for broken dependencies
- **Visible Inbox** (`_Inbox/` with underscore prefix)
- Manifest JSON with full audit trail and originalName recording
- Rollback scripts for every execution
- Searchable JSONL index across all runs
- **Annual maintenance schedule** for deep reviews
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
- Progressive disclosure (main instruction file, references on demand)

---

## 10. Pending tasks

| # | Task | Priority | Status |
|---|------|----------|--------|
| 1 | Test skill with Claude Code (real execution on sample folder) | High | Not started |
| 2 | Test skill with Gemini CLI (real execution on sample folder) | High | Not started |
| 3 | Test skill with OpenAI Codex (real execution on sample folder) | Medium | Not started |
| 4 | Submit to LobeHub Skills Marketplace | Low | Not started |
| 5 | Submit to MCPMarket | Low | Not started |
| 6 | Localization (pt-BR, es, fr, de) | Low | In progress (logic supports, docs in progress) |
| 7 | GUI/TUI preview of plan before execution | Low | Not started |
| 8 | Audio file metadata extraction (ID3, Vorbis, MP4 atoms) | Low | Planned |

---

## 11. Decisions index

Full decision history in DECISION-LOG.md. Key decisions:

- DEC-001 to DEC-027: Foundation, Analysis Pipeline, Traceability, and Final Polish up to v11.
- DEC-028 to DEC-034: v12 features (Annual maintenance, Inbox visibility, Quarantine system, Disambiguation, Photo Library mode, Version Series, Naming consolidation).

---

## 12. How to use this file

When starting a new conversation about this project:

1. Share this file (SESSION-STATE.md) with the assistant.
2. Share DECISION-LOG.md if discussing design rationale.
3. State what you want to work on next.

---

## 13. Competitors analyzed

| Tool | Type | Key insight absorbed |
|------|------|---------------------|
| AI File Sorter | Open-source, Qt6 GUI | Whitelist of categories → our onboarding. Consistent mode. Audio/video metadata. |
| Claw Drive | Bash CLI + JSONL | Persistent searchable index. Re-indexation. Custom metadata. |
| Paperless-ngx | Self-hosted DMS | Correspondent tracking. Auto-tagging by content. |
| Hazel | Mac, rule-based | Watch folder concept. Reliable rule execution. |
| Sparkle | Mac, GPT-4 cloud | Validated need for onboarding to avoid generic classification. |
| Local File Organizer | Python, local LLM | Privacy-first approach. Image analysis. |
| Wisfile | Mac/Windows, local AI | Content-based renaming. |
| LobeHub skills | Marketplace | No PARA-based organizer exists yet. Market gap confirmed. |

---

## 14. Technical constraints

- SKILL.md should stay around 500 lines (currently 510-515 covers all v12 core).
- Front-matter `name` field must match folder name.
- `metadata.version` must be a quoted string.
- `metadata.tags` must be a single string.
- No `user-invocable` field.
- Reference files loaded on demand by the agent.
- All text artifacts use UTF-8 without BOM.