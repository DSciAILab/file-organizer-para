# Duplicate Detection Reference

## Definitions

Exact duplicate: two or more files with identical SHA-256 hash,
regardless of name, location, or timestamp.

Probable duplicate: two or more files with identical size AND
identical extension AND filename Levenshtein distance <= 3, but
different hash. Reported for manual review only. No automated action.

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
   similarity (Levenshtein <= 3 AND same extension). Flag as
   probable duplicate if matched.

## Integration with scan modes

| Mode | Behavior |
|------|----------|
| Quick | Skipped. Agent offers: "Check for duplicates?" |
| Safe | Phase 1 only. If candidates found, ask to run Phase 2. |
| Deep | Phase 1 + Phase 2 automatic. Results in discovery report. |

## Report format

```
Duplicates found: X groups, Y files, Z bytes recoverable

Group 1 (SHA-256: a3f8...c912, 14.2 MB):
  [KEEP]      ~/Downloads/relatorio-anual-2024.pdf  (modified: 2025-12-01)
  [DUPLICATE] ~/Downloads/relatorio-anual-2024 (1).pdf  (modified: 2025-12-01)
  [DUPLICATE] ~/Desktop/relatorio-anual-2024-copia.pdf  (modified: 2025-11-28)

Probable duplicates (same size, similar name, different hash):
  ~/Downloads/proposta-v3.docx (245 KB)
  ~/Downloads/proposta-v4.docx (245 KB)
  -> Recommendation: review manually.
```

## KEEP selection heuristic

Automatic suggestion, user can override:
1. File inside existing PARA structure > file outside.
2. Most recent modification date > older.
3. Shortest path > longer path.
4. Original name (no "(1)", "-copia", "-duplicata") > modified name.

## Actions per group

| Action | Behavior |
|--------|----------|
| archive-duplicates | Move duplicates to 4-Arquivo/Duplicados/YYYY-MM-DD/. KEEP file proceeds to planning. |
| keep-all | No action. All files proceed to planning independently. |
| swap-keep | User selects different file to keep. |
| skip-group | Ignore group entirely. |

## On-demand command

Triggers: "find duplicates", "encontrar duplicados", "check for
duplicates in [folder]", "any repeated files?"

Runs independently of PARA workflow. Works without configured root.

Sequence:
1. Ask for target folder.
2. Ask for depth: shallow (target only) or recursive.
3. Run Phase 1 + Phase 2.
4. Present report.
5. Ask for action per group.
6. If root exists: archive to 4-Arquivo/Duplicados/.
7. If no root: create [target]/Duplicados/[date]/ and archive there.
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

| File count | Behavior |
|------------|----------|
| < 1,000 | Run inline. |
| 1,000-10,000 | Warn with time estimate. |
| > 10,000 | Warn + suggest Phase 1 first. |
| > 50,000 | Warn + suggest limiting scope. |

## Safety rules

1. Never delete any file. Dedup = move to archive.
2. Never act on probable duplicates automatically.
3. Always show report before any action.
4. Always wait for explicit confirmation.
5. Archived duplicates retain original filename.
6. Manifest records every operation for rollback.
7. Rollback reverses archive moves.

## Exclusions

- Symlinks: compare targets, not links.
- Files matching ignore patterns.
- Files inside .git, node_modules, dependency directories.
- Files with size 0 bytes.
- Files without read permission (reported as "N files skipped").
- Hard links with same inode: not duplicates, report as hard links.