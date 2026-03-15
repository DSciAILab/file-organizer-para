# Duplicate Detection Reference

## Definitions

Exact duplicate: two or more files with identical SHA-256 hash,
regardless of name, location, or timestamp.

Probable duplicate: two or more files with identical size AND
identical extension AND filename Levenshtein distance <= 3, but
different hash. Requires human decision. No automated action.

Version series: two or more files sharing a base name with explicit
version markers (v1/v2, _r1/_r2, -draft/-approved). NOT treated
as duplicates. See RENAMING.md for version series handling.


## Algorithm (two-phase)

### Phase 1: Size grouping
1. Group all scanned files by exact byte size.
2. Discard groups with only one member.
3. Result: candidate groups.

### Phase 2: Hash comparison
1. For each candidate group, compute hash.
2. Files <= 100 MB: full SHA-256.
3. Files > 100 MB: partial hash first (first 64 KB + last 64 KB,
   SHA-256). If partial hashes match, compute full SHA-256.
4. Group by identical full hash = exact duplicates.
5. Within same-size groups where hash differs, check filename
   similarity (Levenshtein <= 3 AND same extension).
   Before flagging as probable duplicate, check for version markers.
   If version markers present: classify as VERSION SERIES, not
   probable duplicate. See RENAMING.md.
   Otherwise: flag as probable duplicate.


## Integration with scan modes

| Mode  | Behavior                                              |
|-------|-------------------------------------------------------|
| Quick | Skipped. Agent offers: "Check for duplicates?"        |
| Safe  | Phase 1 only. If candidates found, ask to run Phase 2.|
| Deep  | Phase 1 + Phase 2 automatic. Results in triage.       |


## Report format

```
Duplicates found: X groups, Y files, Z bytes recoverable

Group 1 (SHA-256: a3f8...c912, 14.2 MB):
  [KEEP]      ~/Downloads/relatorio-anual-2024.pdf  (modified: 2025-12-01)
  [DUPLICATE] ~/Downloads/relatorio-anual-2024 (1).pdf  (modified: 2025-12-01)
  [DUPLICATE] ~/Desktop/relatorio-anual-2024-copia.pdf  (modified: 2025-11-28)
  -> KEEP heuristic: most recent + cleanest name
  -> [archive duplicates] [keep all] [swap keep] [skip group]

Probable duplicates (manual review required):
  (see Probable Duplicate Decision Flow below)

Version series (not duplicates, no action needed):
  ~/Downloads/proposta-v3.docx (245 KB)
  ~/Downloads/proposta-v4.docx (245 KB)
  -> Classified as VERSION SERIES. Suggest: group under NN_Versoes/
  -> [group versions] [keep as-is] [skip]
```


## KEEP selection heuristic

Automatic suggestion, user can always override:
1. File inside existing PARA structure > file outside.
2. Most recent modification date > older.
3. Shortest path > longer path.
4. Original name (no "(1)", "-copia", "-duplicata") > modified name.


## Actions per exact duplicate group

| Action             | Behavior                                                         |
|--------------------|------------------------------------------------------------------|
| archive-duplicates | Move duplicates to 4-Arquivo/Duplicados/YYYY-MM-DD/. KEEP proceeds. |
| keep-all           | No action. All files proceed to planning independently.          |
| swap-keep          | User selects a different file to keep.                           |
| skip-group         | Ignore this group entirely.                                      |


## Probable duplicate decision flow

Probable duplicates are NEVER acted on automatically.
Present each pair with structured options:

```
PROBABLE DUPLICATES - Manual review required (N pairs)

Pair 1:
  orcamento-reforma.xlsx       (18 KB, modified 2026-01-10)
  orcamento-reforma-v2.xlsx    (18 KB, modified 2026-01-15)
  Levenshtein: 3 | Same size | Different hash
  Note: version marker detected (-v2). Likely a version series.
  -> [keep both as versions] [archive older] [investigate] [skip]

Pair 2:
  relatorio-Q4.pdf             (2.1 MB, modified 2025-11-20)
  relatorio-Q4-final.pdf       (2.1 MB, modified 2025-11-21)
  Levenshtein: 6 | Same size | Different hash
  Note: "final" marker detected. Likely accidental copy.
  -> [archive non-final] [keep both] [investigate] [skip]

Pair 3:
  contrato-locacao.pdf         (890 KB, modified 2025-03-01)
  contrato-locacoo.pdf         (890 KB, modified 2025-03-01)
  Levenshtein: 1 | Same size | Different hash
  Note: likely typo in filename. Content may be identical.
  -> [archive second] [investigate content] [keep both] [skip]
```

After all pairs resolved, summary is added to the plan.
Unresolved pairs: moved to Legacy-pre-organizacao/ with tag
"probable-duplicate-unresolved" in index.


## On-demand command

Triggers: "find duplicates", "encontrar duplicados", "check for
duplicates in [folder]", "any repeated files?"

Runs independently of PARA workflow. Works without configured root.

Sequence:
1. Ask for target folder.
2. Ask for scan depth: shallow (target folder only) or recursive.
3. Run Phase 1 + Phase 2.
4. Present exact duplicates report.
5. Present probable duplicates decision flow.
6. Ask for action per group.
7. Archive destination:
   - If PARA root exists: 4-Arquivo/Duplicados/YYYY-MM-DD/
   - If no root: ask user for archive destination.
     Suggest: ~/Desktop/Duplicados-YYYY-MM-DD/ (outside scan target).
     Never create archive folder inside the scan target folder.
8. Generate manifest.


## Cross-location dedup

When PARA root exists and user runs on-demand dedup on source folder:
1. Hash files in source.
2. Compare against para-index.jsonl hashes.
3. If match found: flag as "already organized."
4. Report: "~/Downloads/contrato.pdf already exists at
   ~/PARA/2-Areas/Financas/contrato-locacao.pdf"
5. User decides: archive from source, keep in source, or skip.


## Performance

| File count  | Behavior                                        |
|-------------|-------------------------------------------------|
| < 1,000     | Run inline.                                     |
| 1,000-10,000| Warn with time estimate.                        |
| > 10,000    | Warn + suggest Phase 1 first, then ask Phase 2. |
| > 50,000    | Warn + strongly suggest limiting scope.         |


## Safety rules

1. Never delete any file. Dedup = move to archive folder.
2. Never act on probable duplicates automatically.
3. Never act on version series automatically.
4. Always show report before any action.
5. Always wait for explicit confirmation.
6. Archived duplicates retain original filename.
7. Manifest records every operation for rollback.
8. Rollback reverses archive moves to original location.


## Exclusions

- Symlinks: compare targets, not link files.
- Files matching ignore patterns (.DS_Store, thumbs.db, etc.).
- Files inside .git, node_modules, dependency directories.
- Files with size 0 bytes.
- Files without read permission (reported as "N files skipped").
- Hard links with same inode: not duplicates. Reported as hard links.
- Files in Quarentena/: skip until dependency is resolved.


## Known limitations

1. Recompressed images: visually identical photos recompressed by
   WhatsApp, Telegram, or iCloud have different SHA-256. Not detected
   as duplicates by hash. See MEDIA-HANDLING.md for perceptual hash
   option (opt-in, requires ImageMagick or Python imagehash).

2. Format duplicates: same document in different formats (.docx vs
   .pdf, .xlsx vs .csv) produce different hash, size, and extension.
   Not detected. User must identify format duplicates manually.

3. Edited copies: a document with minor edits (one word changed) has
   a different hash and does not qualify as exact or probable duplicate.
   Not detected. Only content analysis (Level 2) may surface this.

4. Zero-byte files: excluded from dedup. All empty files look
   identical by hash; treating them as duplicates would be misleading.
