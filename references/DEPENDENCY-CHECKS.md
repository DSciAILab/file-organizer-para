# Dependency Checks Reference

## When to run

Mandatory before any plan (Step 3). Scope depends on scan mode.

## Status values

- OK: checked and safe to move.
- FLAGGED: checked and risk found. Requires user decision.
- NOT_CHECKED: tool unavailable or check impossible. Never report OK.
- ERROR: check failed unexpectedly.

## Check list by scan mode

### Quick
- File inventory with sizes.
- Files > 1GB: FLAGGED.
- Symlinks detected: FLAGGED.

### Safe (default, includes Quick)
- Software project markers: .git, package.json, Cargo.toml, go.mod,
  .sln, .xcodeproj, Makefile, setup.py, pyproject.toml, .venv,
  node_modules, composer.json, Gemfile, pom.xml, build.gradle.
  -> Move atomically. Never extract internals.
- Hard links (nlink > 1): FLAGGED. Cross-filesystem = forced copy.
- Config files with absolute paths (.bashrc, .zshrc, .gitconfig,
  launch.json, .env, docker-compose.yml, nginx.conf, crontab):
  FLAGGED. Mask secrets in report (show first 4 chars only).
- Dockerfile, docker-compose.yml: conditional markers. Only flag as
  software project if accompanied by another marker.
- Cloud sync boundary: FLAGGED if file is in a synced folder
  (Dropbox, OneDrive, iCloud Drive, Google Drive).
- Cloud placeholders / online-only files: FLAGGED, do not move.
- Read-only files: FLAGGED.
- Files currently in use (lsof / fuser when available): FLAGGED.
- Cross-filesystem detection: compare device IDs. If different,
  force copy+verify+remove instead of rename.
- Space check: free space must be >= max(diskSafetyMarginPercent,
  minSafetyBytes). Insufficient = FLAGGED.
- Case collision: two files differing only in case going to same
  destination. FLAGGED.
- Path length: warn at 240 chars (Windows), hard fail at 260.
- Reserved names (Windows): CON, PRN, NUL, COM1-9, LPT1-9. FLAGGED.
- Permission check: verify write at destination.
- Network filesystem: FLAGGED. Never trust atomic rename on NFS/SMB.

### Deep (includes Safe)
- PATH references: search shell rc files (.bashrc, .zshrc, .profile,
  .bash_profile, fish config) for paths pointing to files being moved.
- Scheduled tasks: cron, launchd (.plist), systemd (.service),
  schtasks on Windows. FLAGGED if references found.
- Shell aliases and functions referencing moved paths.
- Finder aliases (.alias on macOS): BEST_EFFORT.
- Desktop shortcuts (.lnk on Windows): BEST_EFFORT.
- Registry PATH entries (Windows): BEST_EFFORT.

## Hash requirements

Hash (SHA-256) is mandatory when supported for:
- Files >= 1GB.
- Network filesystem transfers.
- Database files (.db, .sqlite, .mdb).
- Executables and binaries.
- Compressed archives (.zip, .tar.gz, .rar, .7z).
- Config files with sensitive content.
- Files that failed in a previous execution.
- Cross-filesystem moves (as part of copy+verify).

## Options per flag type

For each FLAGGED item, the user can choose:
- Proceed: move/copy anyway, accepting the risk.
- Skip: do not touch this file.
- Copy_only: copy but keep original in place.
- Manual: add to post-move checklist for manual handling.

For software projects:
- Move atomic: move entire folder as unit.
- Skip: do not move.
- Copy_only: copy entire folder, keep original.

For config edits:
- Update path: edit config to reflect new location. Backup first.
- Skip config: move file but do not edit config. Add to checklist.
- Manual: add to checklist only.

## Dependency report format

```
DEPENDENCY REPORT
Scan mode: Safe
Total items: 347
OK: 312
FLAGGED: 28
NOT_CHECKED: 7
ERROR: 0

FLAGGED items:
1. [SYMLINK] ~/Downloads/project-link -> ~/Code/myproject
   Options: [skip] [move target] [copy_only]

2. [SOFTWARE_PROJECT] ~/Downloads/my-react-app/ (.git, package.json)
   Options: [move atomic] [skip] [copy_only]

3. [CONFIG_PATH] ~/.bashrc references ~/Downloads/scripts/deploy.sh
   Options: [update path] [skip config] [manual]

4. [LARGE_FILE] ~/Downloads/database-backup.sql (4.2 GB)
   Options: [proceed with hash] [skip] [copy_only]

NOT_CHECKED items:
5. [FINDER_ALIAS] tool unavailable (mdls not found)
6. [SCHEDULED_TASK] launchd check skipped (no launchctl access)
```