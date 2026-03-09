# PARA File Organizer

An [Agent Skill](https://agentskills.io) that organizes local files and folders using Tiago Forte's [PARA method](https://fortelabs.com/blog/para/), with dependency safety checks, execution planning, rollback, and continuous maintenance.

Compatible with [OpenClaw](https://openclaw.ai), [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Cursor](https://cursor.com), [OpenAI Codex](https://openai.com/codex), [Gemini CLI](https://cloud.google.com/gemini), and any tool that supports the [Agent Skills standard](https://agentskills.io/specification).

---

## The Problem

Most people's file systems look the same: hundreds of files scattered across Desktop, Downloads, and Documents with no consistent structure. Finding anything takes longer than it should. Cleaning up feels overwhelming because there's no clear system for deciding where things go.

Existing file organizers sort by extension or date. That helps with the obvious cases (PDFs in one folder, images in another), but it doesn't solve the real problem: knowing whether a file belongs to an active project, an ongoing responsibility, a reference topic, or something you're done with.

## The Solution

This skill teaches your AI agent to organize files using the PARA method, a system designed by productivity expert Tiago Forte that categorizes everything into four folders based on actionability:

**1-Projetos (Projects)** — Short-term efforts with a specific goal and deadline. A tax declaration, a client proposal, a trip you're planning. These end.

**2-Areas (Areas)** — Ongoing responsibilities with no end date. Your finances, your health, your car, your team management. These continue.

**3-Recursos (Resources)** — Reference material on topics you're interested in. Templates, saved articles, tutorials, design inspiration. No direct responsibility.

**4-Arquivo (Archive)** — Inactive items from the three categories above. Completed projects, areas you've moved on from, resources you no longer need.

The key insight from Forte: organize by what you're *doing* with the information, not by what *type* of information it is. A PDF could belong in any of the four categories depending on its role in your life right now.

## What Makes This Skill Different

This isn't a simple "move PDFs to a PDF folder" tool. Before moving a single file, the skill:

**Checks for dependencies.** Symlinks, hard links, software projects (detected by markers like .git, package.json, Cargo.toml), config files with absolute paths, cloud sync boundaries, scheduled tasks, PATH references. Anything that could break if moved is flagged before you approve the plan.

**Presents a full plan.** Every file gets a proposed destination, a confidence level, and a dependency status. You see the entire plan in a table and approve (or modify) before anything happens.

**Moves safely.** Same-filesystem moves use atomic rename. Cross-filesystem moves use copy, verify (by size and hash for critical files), then remove source. Name collisions get `-duplicata-N` suffixes instead of silent overwrites.

**Generates rollback.** Every execution produces a manifest (JSON log of every operation) and a rollback script (.sh or .ps1) that can undo the entire organization. If something goes wrong mid-execution, the skill can resume from where it stopped.

**Handles interruptions.** If the process is interrupted (crash, timeout, closed session), the next run detects the incomplete manifest and offers to resume, rollback, or start fresh.

**Maintains over time.** Weekly and monthly maintenance routines flag stale projects (inactive >30 days) and dormant areas/resources (>90 days) for archiving.

## How It Works

The skill follows a strict 9-step workflow:

```
Passo 0: Setup        -> Configure root directory, tree structure, format
Passo 1: Source        -> Negotiate which directory to organize
Passo 2: Discover      -> Scan files, count, summarize
Passo 3: Dependencies  -> Check for links, configs, projects, cloud sync
Passo 4: Plan          -> Propose destinations with confidence levels
Passo 5: Confirm       -> Show summary, wait for explicit "yes"
Passo 6: Execute       -> Move/copy with verification, log everything
Passo 7: Verify        -> Confirm destinations, update timestamps
Passo 8: Report        -> Full report, manual checklist, rollback script
```

Nothing happens without your approval. The skill recommends, you decide.

## Installation

### OpenClaw

```bash
# Clone the repository
git clone https://github.com/fernandocaravana/file-organizer-para.git

# Copy to OpenClaw skills directory
cp -r file-organizer-para ~/.openclaw/skills/

# Restart your OpenClaw session
```

### Claude Code

```bash
# For personal use (available across all projects)
cp -r file-organizer-para ~/.claude/skills/

# For a specific project (shared via git)
cp -r file-organizer-para .claude/skills/
```

### Cursor / Codex / Gemini CLI

```bash
# Project-level (recommended)
cp -r file-organizer-para .claude/skills/
```

The `.claude/skills/` directory is part of the Agent Skills standard. Despite the name, it works across all compatible tools, not just Claude.

### Verify Installation

```bash
ls ~/.openclaw/skills/file-organizer-para/SKILL.md
# Should show the file. If not, check the path.
```

## Usage

Start a new session in your AI tool and say any of these:

**Portuguese triggers:**
- "organizar meus arquivos"
- "limpar meu desktop"
- "montar estrutura PARA"
- "arquivar projetos antigos"
- "organizar minha pasta de downloads"

**English triggers:**
- "organize my files"
- "clean up my desktop"
- "set up PARA folders"
- "archive old projects"
- "sort my downloads folder"

### First Run

On the first run, the skill will ask you:

1. **Where to create the PARA root** (e.g., ~/Documents/PARA)
2. **Single tree or separate trees** (e.g., Trabalho + Pessoal)
3. **Folder format** — `1-Projetos` (no spaces) or `1. Projetos` (with spaces)

After setup, it asks which directory you want to organize (Desktop, Downloads, Documents, or a custom path), scans it, checks for dependencies, presents a plan, and waits for your approval.

### Quick Commands

After initial setup, you can also use:

| Command | What it does |
|---|---|
| "novo projeto" / "new project" | Creates a new project folder with recommended subfolders |
| "nova area" / "new area" | Creates a new area folder |
| "arquivar projeto" / "archive project" | Moves a project to Archive |
| "manutencao" / "maintenance" | Runs periodic review of stale items |
| "status PARA" / "PARA status" | Shows current configuration and counts |

## File Structure

```
file-organizer-para/
├── SKILL.md                              # Core skill (~450 lines)
├── CHANGELOG.md                          # Version history
├── README.md                             # This file
├── LICENSE                               # MIT
└── references/
    ├── PARA-METHOD-REFERENCE.md          # PARA definitions, examples, naming
    ├── DEPENDENCY-CHECKS.md              # All safety checks and options
    ├── EXECUTION-STRATEGY.md             # Move/copy strategies, edge cases
    ├── MANIFEST-SCHEMA.md                # JSON schemas for config, lock, manifest
    ├── ROLLBACK-AND-RECOVERY.md          # Recovery, rollback, corrupted manifests
    └── MAINTENANCE-AND-COMMANDS.md       # On-demand commands, periodic reviews
```

The SKILL.md file stays under 500 lines, following the [Agent Skills specification](https://agentskills.io/specification). Detailed reference material lives in `references/` and is loaded by the agent only when needed (progressive disclosure, Level 3).

## PARA Folder Structure (Example)

After organizing, your files might look like this:

```
~/Documents/PARA/
├── Trabalho/
│   ├── 1-Projetos/
│   │   ├── 2026-03_Proposta-ClienteX/
│   │   │   ├── 01_Briefing/
│   │   │   ├── 02_Rascunhos/
│   │   │   └── 03_Versao-final/
│   │   └── 2026-04_Relatorio-Q1/
│   ├── 2-Areas/
│   │   ├── Gestao-equipe/
│   │   └── Clientes/
│   ├── 3-Recursos/
│   │   └── Templates-de-documentos/
│   └── 4-Arquivo/
│       ├── Projetos-concluidos/
│       └── Areas-inativas/
├── Pessoal/
│   ├── 1-Projetos/
│   │   └── 2026-05_Aniversario-filho/
│   ├── 2-Areas/
│   │   ├── Financas-pessoais/
│   │   ├── Saude/
│   │   └── Veiculo/
│   ├── 3-Recursos/
│   │   └── Artigos-salvos/
│   └── 4-Arquivo/
│       └── Legado-pre-organizacao/
```

## Safety Features

| Feature | How it works |
|---|---|
| **No deletions** | The skill never deletes user files. Period. |
| **Plan before action** | Full plan presented and approved before any move |
| **Dependency detection** | Symlinks, hard links, software projects, configs, cloud sync, scheduled tasks |
| **Atomic operations** | Same-filesystem moves use rename (atomic). Cross-filesystem uses copy+verify+remove |
| **Hash verification** | Mandatory for files >=1GB, network filesystems, databases, executables, compressed files |
| **Rollback** | Every execution generates a reversible manifest and rollback script |
| **Resume on interruption** | Detects incomplete executions and offers resume, rollback, or fresh start |
| **Lock mechanism** | Prevents concurrent executions with heartbeat-based stale detection |
| **Secret masking** | Config files are inspected for paths but secrets are never shown |

## Configuration

After first run, the skill stores configuration in two places:

**Global pointer** — `~/.config/openclaw/para-root.json` (macOS/Linux) or `%APPDATA%/OpenClaw/para-root.json` (Windows). Points to your active PARA root.

**Root config** — `<root>/.para-config.json`. Contains all settings: mode, trees, scan defaults, depth limits, thresholds, and ignore patterns.

Key configurable values:

| Setting | Default | What it controls |
|---|---|---|
| `scanModeDefault` | safe | Quick, Safe, or Deep dependency scanning |
| `maxDepthBelowCategoryRoot` | 3 | Maximum folder nesting inside categories |
| `batchWriteThreshold` | 1000 | Operations before switching to batch manifest writes |
| `maxRetryPerOperation` | 3 | Retries per failed operation |
| `diskSafetyMarginPercent` | 10 | Minimum free space margin for cross-filesystem copies |
| `heartbeatTimeoutMinutes` | 15 | Lock stale detection threshold |

## Requirements

- Any AI tool that supports the [Agent Skills standard](https://agentskills.io/specification)
- Local filesystem access (the skill does not work on remote-only storage)
- Read permission on the source directory
- Write permission on the PARA root directory

## Methodology

This skill implements an opinionated, filesystem-focused interpretation of Tiago Forte's PARA method. The core categories and decision tree come directly from Forte's work. The operational extensions (naming conventions, dependency checks, manifests, rollback, maintenance routines) are additions built for safe filesystem automation.

For the original methodology, see:
- Forte, T. ["The PARA Method."](https://fortelabs.com/blog/para/) Forte Labs, 2023.
- Forte, T. *The PARA Method: Simplify, Organize, and Master Your Digital Life.* Atria Books, 2023.

## Contributing

Found a bug or have an improvement? Open an issue or submit a PR.

When modifying the skill:
1. Keep SKILL.md under 500 lines.
2. Move detailed content to the appropriate file in `references/`.
3. Update CHANGELOG.md with the change type (fix, feature, breaking).
4. Test in at least one Agent Skills-compatible tool before submitting.

## License

MIT