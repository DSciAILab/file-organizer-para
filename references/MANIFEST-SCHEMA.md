# Manifest Schema Reference

## Global pointer (para-root.json)

```json
{
  "version": 1,
  "activeRoot": "~/Documents/PARA",
  "knownRoots": ["~/Documents/PARA", "~/PARA-archive"]
}
```

## Root config (.para-config.json)

```json
{
  "version": 8,
  "root": "~/Documents/PARA",
  "mode": "separate",
  "trees": ["Work", "Personal"],
  "os": "darwin",
  "categoryFolderFormat": "number-hyphen",
  "maxDepthBelowCategoryRoot": 3,
  "scanModeDefault": "safe",
  "manifestPattern": "para-manifest-YYYY-MM-DD-HHMM.json",
  "historyDir": ".para-manifest-history",
  "tempDir": ".para-temp",
  "ignoredFiles": [".DS_Store", "thumbs.db", "desktop.ini",
    ".Spotlight-V100", ".Trashes", "$RECYCLE.BIN"],
  "safety": {
    "diskSafetyMarginPercent": 10,
    "minSafetyBytes": 1073741824,
    "windowsPathWarnAt": 240,
    "windowsPathHardLimit": 260,
    "lockTimeoutMinutes": 120,
    "heartbeatTimeoutMinutes": 15,
    "batchWriteThreshold": 1000,
    "batchWriteSize": 10,
    "maxRetryPerOperation": 3
  },
  "watchFolders": [],
  "profile": { "...see PARA-METHOD-REFERENCE.md..." },
  "createdAt": "2026-03-10T10:00:00Z",
  "updatedAt": "2026-03-10T14:32:01Z"
}
```

### Trees validation
- mode "single": trees must be empty array.
- mode "separate": trees must have >= 2 unique non-empty strings.
- trees > 5: alert (dispersed structure).

## Lock file (.para-lock.json)

```json
{
  "version": 2,
  "executionId": "abc-123",
  "manifestPath": ".para-manifest-history/para-manifest-2026-03-10-1432.json",
  "startedAt": "2026-03-10T14:32:00Z",
  "updatedAt": "2026-03-10T14:35:12Z",
  "source": "~/Downloads",
  "root": "~/Documents/PARA",
  "mode": "Import",
  "status": "in_progress"
}
```

### Lock rules
- Create before any write. Abort if creation fails.
- Update updatedAt on every operation or every 60s.
- Stale: updatedAt older than heartbeatTimeoutMinutes.
- Stale lock: ask user to resume, rollback or force-remove.
- Remove only on: success, completed rollback, safe abort.
- Reconciliation: executionId, manifestPath, root and source
  must match between lock and manifest.

## Manifest file (para-manifest-*.json)

```json
{
  "schemaVersion": 8,
  "executionId": "abc-123",
  "startedAt": "2026-03-10T14:32:00Z",
  "completedAt": null,
  "root": "~/Documents/PARA",
  "source": "~/Downloads",
  "mode": "Import",
  "scanMode": "safe",
  "folderFormat": "number-hyphen",
  "interactionMode": "guided",
  "operationIndex": 0,
  "directoriesCreated": [],
  "operations": [],
  "configEdits": [],
  "status": "in_progress"
}
```

### Operation entry

```json
{
  "index": 0,
  "type": "move",
  "status": "completed",
  "sourcePath": "~/Downloads/contrato_locacao_final (2).pdf",
  "destinationPath": "~/PARA/Work/2-Areas/Financas/contrato-locacao.pdf",
  "renamed": true,
  "originalName": "contrato_locacao_final (2).pdf",
  "bytesExpected": 245760,
  "bytesCopied": null,
  "verification": "not_applicable",
  "hash": null,
  "tags": ["financas", "casa", "contrato"],
  "correspondent": null,
  "confidence": "high",
  "startedAt": "2026-03-10T14:32:05Z",
  "completedAt": "2026-03-10T14:32:05Z",
  "error": null,
  "retryCount": 0
}
```

### Operation types
- move: rename on same filesystem.
- copy: copy+verify+remove for cross-filesystem.
- copy_only: copy without removing source.
- deduplicate: move duplicate to Archive/Duplicados.
- refine-move: move within PARA structure (Refine mode).

### Status values
- planned: in the plan, not started.
- in_progress: operation started.
- copied: file copied but not yet verified (copy/copy_only).
- verified: copy verified (size + hash match).
- source_removed: origin removed after verification.
- completed: operation finished successfully.
- failed: operation failed. Error recorded.
- rolled_back: operation was reversed.
- skipped: user chose to skip.

### Local rename shortcut
Move via native rename: planned -> completed (skip intermediate
states because rename is atomic).

### Retry logic
- retryCount starts at 0.
- On failure: increment retryCount, ask user.
- Max retries: maxRetryPerOperation from config.
- After max retries: mark as failed, continue to next.

### Deduplicate operation entry

```json
{
  "index": 12,
  "type": "deduplicate",
  "status": "completed",
  "kept": {
    "path": "~/Downloads/relatorio-anual-2024.pdf",
    "hash": "a3f8...c912"
  },
  "archived": [
    {
      "sourcePath": "~/Downloads/relatorio-anual-2024 (1).pdf",
      "destinationPath": "~/PARA/4-Arquivo/Duplicados/2026-03-10/relatorio-anual-2024 (1).pdf",
      "hash": "a3f8...c912",
      "bytesExpected": 14893056,
      "bytesCopied": 14893056,
      "verification": "hash-match"
    }
  ],
  "groupHash": "a3f8...c912",
  "timestamp": "2026-03-10T14:32:01Z"
}
```

### Manifest lifecycle
- Incomplete manifest on startup: offer resume or rollback.
- Complete manifest: archive to .para-manifest-history/.
- Corrupted manifest (invalid JSON): attempt partial parse.
  If unrecoverable, rename to .corrupted, warn user, do not
  delete. Offer fresh start.
- Unknown schemaVersion: warn user, attempt best-effort read.
  If incompatible, suggest updating skill.

### Security
- Manifests never store file contents.
- Config paths are masked in reports (first 4 chars + ***).
- Secrets (.env values, API keys) never recorded.
- Only paths, types, names, states, hashes and errors.

## Searchable index (para-index.jsonl)

One JSON line per organized file. See EXECUTION-STRATEGY.md for
field schema. Used for:
- "where was [name]?" command: search originalPath and path fields.
- "trace [name]" command: aggregate all entries across executions.
- Cross-location dedup: compare hash against existing entries.
- Cross-category search: filter by tags regardless of physical path.

### Index lifecycle
- Appended during execution (Step 6).
- Never deleted automatically.
- "reindex" command: re-read manifest history and rebuild index
  from scratch. Useful after manual file moves.
- Corruption: if invalid line found, skip and log warning.
  Offer reindex.

## Manifest history index (.para-manifest-history/index.json)

```json
{
  "executions": [
    {
      "executionId": "abc-123",
      "date": "2026-03-10T14:32:00Z",
      "mode": "Import",
      "source": "~/Downloads",
      "operationCount": 47,
      "status": "completed",
      "manifestFile": "para-manifest-2026-03-10-1432.json"
    }
  ]
}
```

Updated after each execution completes.