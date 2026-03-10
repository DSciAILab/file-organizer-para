# Execution Strategy Reference

## Move strategy selection

### Same-filesystem local
Use native rename. Exceptions:
- Network filesystem: never trust rename atomicity.
- Cloud sync boundary: use copy+verify+remove.
- Hard links where preserving link count matters: copy.
- User explicitly requested copy+verify.

### Cross-filesystem
1. Copy file to destination.
2. Verify: compare size. If hash required (see DEPENDENCY-CHECKS.md),
   compute SHA-256 on both and compare.
3. Only if verification passes: remove origin.
4. If verification fails: keep origin, mark operation as failed,
   log error in manifest, do not remove anything.

### Network filesystem
Always copy+verify+remove. Never rename.
NFS: fsync is not guaranteed. After copy, read back and verify.
SMB: same approach.

## Atomic writes

For manifest, config and index files:
1. Write to temporary file in .para-temp/.
2. Flush and fsync.
3. Rename temp file to final path (atomic on local POSIX/NTFS).

## Batch writes

When total operations > batchWriteThreshold (default 1000):
- Write manifest in batches of batchWriteSize (default 10).
- After each batch: flush, update heartbeat.
- If interrupted mid-batch: manifest records completed operations.
  Incomplete batch is recoverable on resume.

## Collision handling

If destination file already exists:
1. Never overwrite.
2. Append -duplicata-N (N starting at 1, increment until unique).
3. Record renamed destination in manifest (renamed=true).

## Config edits

When a config file references a moved path:
1. Create backup: original.bak-YYYY-MM-DD-HHMM.
2. Edit config to update path.
3. Validate syntax of edited config (JSON parse, YAML parse, etc.).
4. If validation fails: restore backup, mark as failed, add to checklist.

## Software projects

Move entire folder as atomic unit. Never:
- Extract individual files from inside.
- Rename internal files.
- Modify internal configs.

After move: verify folder exists at destination with same structure.

## Metadata handling

| Move type | Metadata |
|-----------|----------|
| Local rename | Preserved (OS handles it) |
| Copy | Best_effort (timestamps may change) |
| Cross-filesystem | Best_effort |
| Network | Not_checked |

## Index update (para-index.jsonl)

After each successful operation, append a JSON line:

```json
{
  "path": "~/PARA/Work/2-Areas/Financas/contrato-locacao.pdf",
  "originalPath": "~/Downloads/contrato_locacao_final (2).pdf",
  "hash": "a3f8...c912",
  "tags": ["financas", "casa", "contrato", "aluguel"],
  "category": "Areas",
  "area": "Financas",
  "project": null,
  "description": "Lease contract",
  "correspondent": null,
  "customMetadata": {},
  "organizedAt": "2026-03-10T14:32:01Z",
  "executionId": "abc-123",
  "renamed": true,
  "originalName": "contrato_locacao_final (2).pdf"
}
```

Tags come from: profile aliases, keyword analysis clusters,
user-assigned tags during triage, content analysis output.

Correspondent comes from: content analysis (if enabled and detected),
user input during triage, or null.

## PARA-CHANGELOG.md

Appended after each execution:

```markdown
## 2026-03-10 14:32 (exec abc-123)
- Mode: Import
- Source: ~/Downloads
- 47 files moved, 3 renamed, 12 duplicates archived
- New projects created: SJJP-Evento-anual-2026
- New resources: Templates-financeiros
- Flagged: 2 symlinks skipped, 1 config updated
- Duration: 4m 12s
```

## Watch folder mode

When user activates "watch [folder]":
1. Agent notes the folder path and scan parameters.
2. On subsequent invocations or periodic checks, agent scans
   the watched folder for new files (modified date > last scan).
3. New files are presented with classification suggestions.
4. User approves or adjusts. Agent executes approved moves.
5. Watch state stored in .para-config.json under "watchFolders".

Note: the agent does not run as a daemon. Watch mode depends on
the user invoking the agent periodically or the host environment
providing scheduling (cron, launchd, etc.).

## Refine mode specifics

When mode = Refine:
- Skip root setup and PARA structure creation.
- Load existing profile.
- Scan only the target folder.
- Suggest: better sub-folder organization, files that belong in a
  different PARA category, name standardization, internal duplicates,
  tag enrichment in the index.
- Moves within PARA use type "refine-move" in manifest.
- Cross-category moves (e.g., from Areas to Projects) are valid
  and recorded with both source and destination category.

## Consistent classification mode

When processing large batches with many similar files:
- After the first few classifications establish a pattern, bias
  subsequent suggestions toward the same category.
- Example: if 5 files matching "invoice-*" were classified as
  Areas/Financas, suggest the same for the 6th without asking.
- User can override any individual suggestion.
- Disable by choosing "Full control" interaction mode.

## Correspondent tracking

When content analysis (level 1+) detects sender or organization:
- Record in index under "correspondent".
- Examples: "Receita Federal", "SJJP", "Landlord John".
- If not detected, field is null.
- User can manually assign during triage.