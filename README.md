# File Organizer PARA – AI Agent Skill

An AI agent skill that organizes messy files into a clean **PARA** (Projects, Areas, Resources, Archive) folder structure. Works with any SKILL.md-compatible agent runtime, including Claude, Gemini CLI, OpenAI Codex, and Antigravity.

---

## Why

Most people have hundreds or thousands of files scattered across Downloads, Desktop, and random folders. Manual sorting is tedious and inconsistent. This skill gives your AI agent a structured, opinionated methodology to do it for you, with rollback safety, duplicate detection, keyword analysis, media handling, and full traceability.

---

## Features

**Core Organization**
- Classifies files into Projects, Areas, Resources, or Archive using the PARA method.
- Onboarding profile asks you to define your areas, projects, and resources before scanning, so the agent already knows your mental map.
- Three operation modes: Import (external → PARA), Refine (reorganize within PARA), Maintain (scheduled cleanup).

**Analysis Pipeline**
- Keyword extraction from filenames (tokenization, frequency clustering, alias matching).
- Optional content analysis (Level 0: metadata only, Level 1: shallow read, Level 2: deep read with permission).
- Media detection with EXIF parsing, screenshot identification, and date-based grouping.
- Duplicate detection via SHA-256 hashing (Quick, Safe, Deep scan modes).
- Dependency checks for symlinks, hardlinks, config references, and Git submodules.

**Safety**
- Never moves or deletes without explicit user confirmation.
- Manifest JSON logs every action (source, destination, hash, timestamp, operation type).
- Rollback scripts generated automatically for every execution.
- Execution lock prevents concurrent runs on the same root.
- Three interaction modes: Guided (agent decides high-confidence items, asks about the rest), Total Control (asks about everything), Total Confidence (agent decides all, shows plan for final approval).

**Traceability**
- Cumulative manifest index across all runs.
- Human-readable PARA-CHANGELOG.md updated after each execution.
- Reverse lookup: ask "where did [filename] go?" or "where did [filename] come from?" and the agent searches all manifests.

**Extras**
- Watch folder support for continuous monitoring (e.g., ~/Downloads).
- Correspondent tracking for documents from external sources.
- Consistent mode bias to reduce over-fragmentation of categories.
- Cross-platform: macOS, Linux, Windows (with documented capability differences).

---

## Requirements

- A SKILL.md-compatible AI agent runtime. Tested with:
  - **Claude** (Claude Code, Claude Desktop, Claude Cowork)
  - **Gemini CLI / Antigravity**
  - **OpenAI Codex**
- File system access permissions granted to the agent.
- Optional: `exiftool`, `mdls` (macOS), or ImageMagick `identify` for EXIF extraction.
- Optional: `sha256sum` (Linux/macOS) or `certutil` (Windows) for duplicate detection. The agent falls back to built-in hashing if these are unavailable.

---

## Installation

### Claude (Global Skill)

```bash
git clone https://github.com/DSciAILab/file-organizer-para.git /tmp/file-organizer-para
mkdir -p ~/.claude/skills/file-organizer-para
cp -r /tmp/file-organizer-para/file-organizer-para/* ~/.claude/skills/file-organizer-para/
```

Verify:

```bash
ls ~/.claude/skills/file-organizer-para/SKILL.md
```

### Gemini CLI / Antigravity (Global Skill)

```bash
git clone https://github.com/DSciAILab/file-organizer-para.git /tmp/file-organizer-para
mkdir -p ~/.gemini/skills/file-organizer-para
cp -r /tmp/file-organizer-para/file-organizer-para/* ~/.gemini/skills/file-organizer-para/
```

For Antigravity, the path may be `~/.gemini/antigravity/skills/file-organizer-para/` depending on your version. Check Settings → Agent → Skills.

### OpenAI Codex

```bash
git clone https://github.com/DSciAILab/file-organizer-para.git /tmp/file-organizer-para
mkdir -p ~/.codex/skills/file-organizer-para
cp -r /tmp/file-organizer-para/file-organizer-para/* ~/.codex/skills/file-organizer-para/
```

Also see `file-organizer-para/agents/openai.yaml` for Codex-specific configuration.

### Workspace-Level (Any Runtime)

Instead of installing globally, copy the skill folder into your project:

```bash
cp -r /tmp/file-organizer-para/file-organizer-para .agent/skills/file-organizer-para
```

Or for Claude specifically:

```bash
cp -r /tmp/file-organizer-para/file-organizer-para .claude/skills/file-organizer-para
```

---

## Usage

Open a new conversation with your agent (Planning mode recommended) and use natural language:

**First run (Import mode):**

> "Organize my ~/Downloads folder using PARA"

The agent will:
1. Detect the skill and load instructions.
2. Ask where you want the PARA root (or suggest a default).
3. Run onboarding: ask about your areas, projects, and resources.
4. Scan the source folder.
5. Analyze keywords, detect media, find duplicates.
6. Present a triage session for ambiguous items.
7. Show the full plan and ask for confirmation.
8. Execute moves, generate manifest, rollback script, and report.

**Refine mode (reorganize within PARA):**

> "Refine the organization of my 2-Areas/Work-SJJP folder"

The agent detects the source is inside the PARA root and enters Refine mode, suggesting sub-folder restructuring, reclassification, and renaming without re-running the full import pipeline.

**Maintain mode:**

> "Run maintenance on my PARA root"

The agent checks for orphaned files, suggests archiving completed projects, re-runs deduplication, and updates the changelog.

**On-demand commands:**

> "Find duplicates in my PARA root"

> "Where did relatorio-Q4.pdf go?"

> "Where was contrato-locacao.pdf before?"

> "Show my PARA changelog"

---

## File Structure

```
file-organizer-para/
├── SKILL.md                              # Main skill instructions (< 500 lines)
├── CHANGELOG.md                          # Version history
├── agents/
│   └── openai.yaml                       # OpenAI Codex compatibility layer
└── references/
    ├── PARA-METHOD-REFERENCE.md          # PARA methodology deep dive
    ├── DEPENDENCY-CHECKS.md              # Symlink, hardlink, config detection
    ├── EXECUTION-STRATEGY.md             # Move/copy/rename logic, platform matrix
    ├── MANIFEST-SCHEMA.md                # JSON schema for action manifests
    ├── ROLLBACK-AND-RECOVERY.md          # Undo procedures and recovery flows
    ├── DUPLICATE-DETECTION.md            # Hash-based dedup algorithm and reports
    ├── KEYWORD-AND-CONTENT-ANALYSIS.md   # Filename tokenization and content reading
    ├── MEDIA-HANDLING.md                 # Photo/video/audio detection and EXIF
    ├── MAINTENANCE-AND-COMMANDS.md       # On-demand commands, watch folder, cron
    └── RENAMING.md                       # Naming conventions and renaming logic
```

---

## How PARA Works (Quick Summary)

| Category      | What goes here                                    | Key trait            |
|---------------|---------------------------------------------------|----------------------|
| 1-Projects    | Active efforts with a deadline and deliverable     | Has an end date      |
| 2-Areas       | Ongoing responsibilities with no end date          | Continuous           |
| 3-Resources   | Reference material, templates, learning content    | Might be useful      |
| 4-Archive     | Completed projects, inactive areas, old resources  | No longer active     |
| 5-Fotos       | Personal and professional photo/video library      | Permanent memory     |

The skill also creates `Legado-pre-organizacao/` for files that cannot be classified and `Quarentena/` for items with broken dependencies. Photos are organized into `5-Fotos/` by default.

For the full methodology reference, see `references/PARA-METHOD-REFERENCE.md`.

---

## Configuration

After the first run, the skill stores a profile at `<para-root>/.para-config.json` containing:

- Your defined areas (with aliases and keywords).
- Active projects (with deadlines and associated areas).
- Resources categories.
- Interaction mode preference (Guided / Total Control / Total Confidence).
- Scan mode preference (Quick / Safe / Deep).
- Watch folder paths (if configured).

This profile is loaded on subsequent runs so the agent remembers your organizational structure.

---

## Safety & Privacy

- **No cloud processing by default.** The skill instructs the agent to use only local filesystem operations. Content analysis (Level 1/2) is opt-in and processed by whatever LLM runtime you are already using.
- **Nothing is deleted.** The skill moves files; it never deletes. Even "archived duplicates" are moved to a dedup archive folder.
- **Rollback always available.** Every execution generates a shell script that reverses all moves.
- **Manifest is your audit trail.** Every action is logged with source path, destination path, SHA-256 hash, timestamp, and operation type.

---

## Contributing

Contributions are welcome. To propose changes:

1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/your-feature`).
3. Make changes to the relevant files in `file-organizer-para/`.
4. Ensure SKILL.md stays under 500 lines. Use reference files for detailed specs.
5. Update `CHANGELOG.md` with your changes.
6. Run validation if you have the tools: `skills-ref validate ./file-organizer-para`.
7. Open a pull request with a clear description of what changed and why.

### Guidelines

- Keep SKILL.md focused on what the agent needs to know during execution.
- Put detailed algorithms, schemas, and edge cases in `references/`.
- Use the front-matter fields defined by the agentskills.io specification.
- Test with at least one agent runtime before submitting.

---

## Roadmap

- [ ] GUI/TUI preview of the plan before execution.
- [ ] Integration with Paperless-ngx for scanned document tagging.
- [ ] Audio file metadata extraction (ID3, Vorbis, MP4 atoms).
- [ ] Email attachment organization (pull from .eml/.msg files).
- [ ] Johnny Decimal hybrid mode (optional numbering within PARA).
- [ ] Localization (pt-BR, es, fr, de).
- [ ] Published to LobeHub Skills Marketplace and MCPMarket.

---

## License

MIT. See [LICENSE](LICENSE) for details.

---

## Acknowledgments

- **Tiago Forte** for the PARA method (Building a Second Brain).
- The open-source community behind tools like AI File Sorter, Local File Organizer, Claw Drive, and Paperless-ngx, whose features informed the design of this skill.
- The agent skill specification at [agentskills.io](https://agentskills.io/specification) for providing a portable, runtime-agnostic standard.