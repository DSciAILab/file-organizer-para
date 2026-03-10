# Rollback and Recovery Reference

## Recovery on startup

When the skill starts (Step 0), check for:
1. Stale lock file (updatedAt > heartbeatTimeoutMinutes).
2. Incomplete manifest (status != "completed").

If found, present options:
- Resume: continue from last completed operation.
- Rollback: reverse all completed operations.
- Abort: remove lock, keep files as-is. User handles manually.

## Rollback procedure

### Order
Reverse order of completed operations (last completed first).

### By operation type

**move (local rename)**:
1. Rename file back from destination to source.
2. If file was renamed (renamed=true), restore original name.
3. Mark operation as rolled_back in manifest.

**copy (cross-filesystem)**:
1. If source still exists at original location: remove destination copy.
2. If source was removed (status = source_removed): move destination
   back to original source path.
3. Mark as rolled_back.

**copy_only**:
1. Remove destination copy (source is always intact).
2. Mark as rolled_back.

**deduplicate**:
1. Move archived files back from Archive/Duplicados/ to their
   original locations (using sourcePath from manifest).
2. Mark as rolled_back.

**refine-move**:
1. Move file back to previous location within PARA.
2. Restore original name if renamed.
3. Mark as rolled_back.

**configEdits**:
1. Restore backup file (.bak-YYYY-MM-DD-HHMM).
2. Validate restored config.
3. Mark as rolled_back.

### Collision during rollback
If original location is now occupied:
1. Never overwrite.
2. Place file at original path with -rollback-collision-N suffix.
3. Report collision in rollback summary.

### Created directories
Remove only if empty after rollback. If not empty (other files
were placed there independently), keep and report.

### Partial rollback
If a rollback operation itself fails:
1. Record the failure.
2. Continue with remaining rollback operations.
3. Mark overall rollback as "partial".
4. Report which operations could not be reversed.

## Rollback script

Generated after each execution as:
<root>/para-rollback-YYYY-MM-DD-HHMM.sh (macOS/Linux)
<root>/para-rollback-YYYY-MM-DD-HHMM.ps1 (Windows)

Script contains the exact reverse commands for every completed
operation. Can be run manually outside the skill if needed.

## Interactive rollback

Available via on-demand command or when skill detects incomplete state.
Options:
- Rollback all: reverse everything.
- Rollback selective: show operations, user picks which to reverse.
- Rollback to checkpoint: reverse operations after a specific index.

## Manifest reconciliation

On startup, if lock exists but manifest state disagrees:
1. Compare executionId between lock and manifest.
2. If mismatch: stale lock from different execution. Ask user.
3. If match but status disagrees: trust manifest (it has more detail).
4. Verify files on disk match manifest expectations.
5. If disk state diverges (user moved files manually): report
   discrepancies, ask user before any action.

## Edge cases

**Corrupted manifest**: attempt JSON parse. If partial, recover
what is parseable. Rename original to .corrupted. Report.

**Unknown schema version**: attempt best-effort field mapping.
If critical fields missing, warn and suggest skill update.

**Permission revoked mid-rollback**: record which operations
could not be reversed, continue with others, report.

**Disk full during rollback**: prioritize restoring files to
source (which frees destination space). If both locations are
on the same disk, report and ask user to free space.

**Files modified after organization**: if a file at the destination
has been modified since the organization (different hash or size),
warn before rollback. Moving a modified file back may lose changes.
Ask user per file.