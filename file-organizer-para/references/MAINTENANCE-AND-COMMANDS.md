# Maintenance and Commands Reference

## On-demand commands

### "new project" / "novo projeto"
1. Ask for project name and area (if separate trees).
2. Ask for deadline (optional but recommended).
3. Create folder: <root>/<tree>/1-Projetos/YYYY-MM_Name/
4. If profile.projectTemplate defined: use it.
   Otherwise use default sub-folders:
   01_Briefing, 02_Research, 03_Assets, 04_Deliverables, 05_Communication.
5. Ask: "Use default template or customize for this project?"
6. Add to profile.activeProjects if deadline provided.
7. Update index.

### "new area" / "nova area"
1. Ask for area name.
2. Create folder: <root>/<tree>/2-Areas/Name/
3. Ask for aliases (keywords that identify this area).
4. Add to profile.areas.
5. Update config.

### "archive project" / "arquivar projeto"
1. List active projects. User selects one.
2. Move entire project folder to 4-Arquivo/Projetos-concluidos/.
3. Remove from profile.activeProjects.
4. Generate manifest entry.
5. Update index entries for all files in the project.

### "maintenance" / "manutencao"
0. Check _Inbox/ item count.
   If items present: "You have N items in _Inbox/. Process now?
   (yes / later)" If yes, run triage on _Inbox/ contents.
1. Weekly check: projects with no modification > 30 days.
   Suggest archiving.
2. Monthly check: areas/resources with no modification > 90 days.
   Suggest review.
3. Check Legado-pre-organizacao/ for remaining items. Report count.
4. Check Quarentena/ for unresolved items. Report count and oldest.
5. Re-run dependency check on previously flagged items if relevant.
6. Suggest dedup on high-turnover folders (Downloads, Desktop).
7. Suggest index reindex for enrichment.

### "PARA status" / "status PARA"
Show:
- Root path, mode, trees, folder format.
- Number of items per category (Projects, Areas, Resources, Archive).
- Active projects with deadlines.
- _Inbox/ item count.
- Quarentena/ item count and oldest item date (if any).
- Legado-pre-organizacao/ item count (if any).
- Profile summary (areas, aliases).
- Last execution date and summary.
- Watch folders status.
- Disk space used by PARA root.

### "find duplicates" / "encontrar duplicados"
See DUPLICATE-DETECTION.md. Independent of workflow.

### "where was [name]?" / "onde estava [nome]?"
1. Search para-index.jsonl for entries where originalPath or path
   contains the search terms (case-insensitive, partial match).
2. Search manifest history for operations with matching sourcePath
   or destinationPath.
3. Present results chronologically:
   "relatorio-2024.pdf:
    2026-01-15: moved from ~/Downloads/ to ~/PARA/1-Projetos/...
    2026-04-02: archived to ~/PARA/4-Arquivo/..."
4. If no results: "No records found for [name]."

### "trace [name]" / "rastrear [nome]"
Same as "where was" but shows complete trajectory with all
intermediate locations and renames, including originalName
before any rename operations.

### "update areas" / "atualizar areas"
1. Show current profile.
2. User adds, removes, renames, merges areas.
3. User updates aliases.
4. User adds/removes active projects.
5. Save updated profile to config.

### "refine [folder]" / "melhorar [pasta]"
1. Validate folder is inside PARA root.
2. Enter Refine mode.
3. Scan only that folder.
4. Suggest improvements: sub-folder structure, cross-category moves,
   name standardization, internal dedup, tag enrichment.
5. Present plan. Wait for approval. Execute.

### "watch [folder]"
1. Register folder in config.watchFolders with last scan timestamp.
2. On subsequent invocations: scan for files newer than last scan.
3. Present new files with classification suggestions.
4. User approves. Agent executes.
5. Update last scan timestamp.
Note: agent is not a daemon. Watch depends on user invoking the
agent periodically or host scheduling.
Recommended: add ~/Downloads and _Inbox/ to watchFolders by default.

### "reindex"
1. Read all manifests in .para-manifest-history/.
2. Rebuild para-index.jsonl from scratch.
3. Optionally re-enrich descriptions using content analysis.
4. Report: "Index rebuilt. N entries. M enriched."

### "resolve quarantine" / "resolver quarentena"
1. List all items in Quarentena/ with their original dependency flag.
2. For each item, show: original flag, date quarantined, file info.
3. Ask: [resolve and move to PARA] [force move] [skip] [delete]
4. For "resolve and move": run mini dependency check, then plan.
5. Update index. Generate manifest entry.


## Periodic maintenance schedule

| Frequency | Duration | Actions |
|-----------|----------|---------|
| Weekly    | ~5 min   | _Inbox/ triage, stale projects check, Legado check |
| Monthly   | ~15 min  | Stale areas/resources, dedup on Downloads, index review, Quarentena/ review |
| Quarterly | ~30 min  | Full dependency re-check, profile review, archive cleanup |
| Annual    | ~60 min  | Full archive review (items > 2 years), purge Legado-pre-organizacao/, consolidate Resources, full reindex with content enrichment, validate all profile aliases |

Agent suggests these during "maintenance" command or proactively
if the user starts a conversation related to organization.
