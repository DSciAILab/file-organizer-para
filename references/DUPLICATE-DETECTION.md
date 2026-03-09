## Section 10-A – Duplicate Detection

### 10-A.1 – Definitions

**Exact duplicate**: two or more files with identical SHA-256 hash,
regardless of name, location, or timestamp.

**Probable duplicate**: two or more files with identical size AND
identical extension AND filename Levenshtein distance ≤ 3, but
different hash. Reported as "possible duplicates" for manual review
only. No automated action.

### 10-A.2 – Detection Algorithm (two-phase)

Phase 1 – Size grouping:
1. Group all scanned files by exact byte size.
2. Discard groups with only one member.
3. Result: candidate groups.

Phase 2 – Hash comparison:
1. For each candidate group, compute hash.
2. Files ≤ 100 MB: full SHA-256.
3. Files > 100 MB: partial hash first (first 64 KB + last 64 KB,
   SHA-256). If partial hashes match, compute full SHA-256 to confirm.
4. Group by identical full hash = exact duplicates.
5. Within same-size groups where hash differs, check filename
   similarity (Levenshtein ≤ 3 AND same extension). Flag as
   "probable duplicate" if matched.

### 10-A.3 – Integration with Scan Modes

| Scan Mode | Duplicate Detection Behavior                    |
|-----------|--------------------------------------------------|
| Quick     | Skipped by default. Agent offers: "Quer que eu   |
|           | verifique duplicados? Vai levar mais tempo."      |
|           | If user accepts, runs Phase 1 + Phase 2.          |
| Safe      | Runs Phase 1 only (size grouping). If candidate   |
|           | groups found, agent reports count and asks:        |
|           | "Encontrei N grupos com tamanho idêntico.          |
|           | Calcular hash para confirmar duplicados?"          |
| Deep      | Runs Phase 1 + Phase 2 automatically. Results     |
|           | included in discovery report.                      |

### 10-A.4 – Duplicate Report Format

When duplicates are found, the agent presents:

```
Duplicados encontrados: X grupos, Y arquivos, Z bytes recuperáveis

Grupo 1 (SHA-256: a3f8…c912, 14.2 MB):
  [MANTER]  ~/Downloads/relatorio-anual-2024.pdf  (modificado: 2025-12-01)
  [DUPLICADO] ~/Downloads/relatorio-anual-2024 (1).pdf  (modificado: 2025-12-01)
  [DUPLICADO] ~/Desktop/relatorio-anual-2024-copia.pdf  (modificado: 2025-11-28)

Grupo 2 (SHA-256: b7e1…d433, 3.1 MB):
  [MANTER]  ~/Downloads/foto-perfil.png  (modificado: 2026-01-15)
  [DUPLICADO] ~/Documents/foto-perfil.png  (modificado: 2025-09-20)

Possíveis duplicados (mesmo tamanho, nome similar, hash diferente):
  ~/Downloads/proposta-v3.docx (245 KB)
  ~/Downloads/proposta-v4.docx (245 KB)
  → Recomendação: revisar manualmente.
```

Selection heuristic for [MANTER] (automatic suggestion, user overrides):
1. File inside an existing PARA structure > file outside.
2. Most recent modification date > older.
3. Shortest path > longer path.
4. Original name (no "(1)", "-copia", "-duplicata") > modified name.
User can override any suggestion before confirmation.

---

## Section 10-B – Workflow Integration (Option 1)

### Step 2.5 – Deduplication (between Discovery and Dependency Check)

This step is OPTIONAL and triggered based on scan mode (see 10-A.3)
or explicit user request.

Sequence:
1. After Step 2 (Discovery) completes, if duplicates were detected
   (or user requests check), present the Duplicate Report (10-A.4).
2. Ask user for action per group. Available actions:

   | Action             | Behavior                                      |
   |--------------------|-----------------------------------------------|
   | archive-duplicates | Move duplicates to 4-Arquivo/Duplicados/      |
   |                    | with subdirectory named by date                |
   |                    | (e.g., 4-Arquivo/Duplicados/2026-03-09/).     |
   |                    | Original (MANTER) proceeds to PARA planning.  |
   | keep-all           | No action on this group. All files proceed     |
   |                    | to PARA planning independently.                |
   | swap-keep          | User selects a different file as the one to    |
   |                    | keep. Others become duplicates.                |
   | skip-group         | Ignore this group entirely. Files still         |
   |                    | proceed to PARA planning but no dedup action.  |

3. If user chooses archive-duplicates, the archived files are added
   to the execution plan (Step 4) as type "deduplicate".
4. The MANTER file continues to Step 3 (Dependency Check) and
   Step 4 (Planning) normally.
5. If user declines deduplication entirely ("não, organiza direto"),
   skip to Step 3 with all files.

### Manifest entry for deduplicate operations

```json
{
  "type": "deduplicate",
  "status": "completed",
  "kept": {
    "path": "~/Downloads/relatorio-anual-2024.pdf",
    "hash": "a3f8...c912"
  },
  "archived": [
    {
      "sourcePath": "~/Downloads/relatorio-anual-2024 (1).pdf",
      "destinationPath": "~/Documents/PARA/4-Arquivo/Duplicados/2026-03-09/relatorio-anual-2024 (1).pdf",
      "hash": "a3f8...c912",
      "bytesExpected": 14893056,
      "bytesCopied": 14893056,
      "verification": "hash-match"
    }
  ],
  "groupHash": "a3f8...c912",
  "timestamp": "2026-03-09T14:32:01Z"
}
```

---

## Section 10-C – On-Demand Command (Option 2)

### Triggers

PT: "encontrar duplicados", "buscar duplicados", "verificar duplicados em [pasta]",
    "tem arquivos repetidos?", "limpar duplicados"
EN: "find duplicates", "check for duplicates in [folder]", "deduplicate [folder]",
    "any repeated files?"

### Behavior

This command runs INDEPENDENTLY of the PARA organization workflow.
It can be invoked at any time, even if no PARA root is configured.

Sequence:
1. Ask user for target folder (or use current context if obvious).
2. Ask for scan depth: shallow (target folder only) or recursive
   (target + all subfolders).
3. Run Phase 1 + Phase 2 (10-A.2) on the target.
4. Present Duplicate Report (10-A.4).
5. Ask user for action per group (same options as 10-B Step 2.5).
6. If PARA root exists and user chooses archive-duplicates,
   move to 4-Arquivo/Duplicados/[date]/.
7. If NO PARA root exists and user chooses archive-duplicates,
   create a subfolder [target]/Duplicados/[date]/ and move there.
   Inform user: "Sem root PARA configurado, arquivei os duplicados
   em [target]/Duplicados/[date]/."
8. Generate manifest for all actions taken (same schema as 10-B).
9. If no duplicates found, report: "Nenhum duplicado encontrado
   em [pasta]. [N] arquivos analisados, [X] grupos de tamanho
   idêntico verificados por hash."

### Constraints
- Never delete files. Only move to archive/duplicates folder.
- Always confirm before any move.
- If target folder has > 10,000 files, warn user about estimated
  time before starting hash computation.
- Respect the same ignore patterns as the main PARA workflow
  (e.g., .DS_Store, node_modules, .git).

---

## Section 10-D – Combined Mode (Option 3)

When the full PARA workflow runs with deduplication enabled
(Step 2.5), the on-demand command remains available afterward
for periodic re-checks.

Recommended usage pattern:
1. First organization: run full workflow with Deep scan
   (auto-detects duplicates via 10-A.3).
2. Periodic maintenance: use on-demand command (10-C) to check
   specific folders (e.g., Downloads) for new duplicates that
   accumulated since last organization.
3. The on-demand command checks BOTH the source folder AND the
   existing PARA root to catch cross-location duplicates
   (e.g., a file in Downloads that already exists in
   1-Projetos/projeto-x/).

### Cross-location deduplication

When PARA root exists AND user runs on-demand dedup on a source
folder:
1. Hash all files in source folder (Phase 1 + Phase 2).
2. Compare against manifest history: if a file's hash matches
   a previously moved file already in the PARA structure,
   flag it as "already organized."
3. Report format adds a line:

```
Já organizados (existem no PARA root):
  ~/Downloads/contrato-locacao.pdf → já em 2-Areas/Casa/contrato-locacao.pdf
  ~/Downloads/manual-carro.pdf → já em 2-Areas/Veiculo/manual-carro.pdf
  → Ação sugerida: arquivar ou remover da fonte.
```

4. User decides per file: archive to Duplicados, keep in source,
   or skip.

---

## Section 10-E – Performance and Safety

### Performance guidelines

| File count   | Expected behavior                                |
|--------------|--------------------------------------------------|
| < 1,000      | Run inline, no special warning.                  |
| 1,000-10,000 | Warn: "~N minutos estimados para hash completo." |
| > 10,000     | Warn + suggest Phase 1 only first, then hash     |
|              | only the candidate groups.                       |
| > 50,000     | Warn + suggest limiting to specific subfolders   |
|              | or file types.                                   |

### Safety rules

1. NEVER delete any file. Deduplication = move to archive folder.
2. NEVER act on "probable duplicates" automatically. Report only.
3. ALWAYS show the full duplicate report before any action.
4. ALWAYS wait for explicit user confirmation per group or
   batch ("arquivar todos os duplicados encontrados").
5. Archived duplicates retain original filename. If collision
   in archive folder, append -duplicata-N.
6. Manifest records every deduplicate operation with full hash,
   source, destination, and timestamp for rollback.
7. Rollback of deduplicate operations follows the same procedure
   as Section 26 (move files back to original location using
   manifest sourcePath).

### Excluded from deduplication

- Symlinks (compare targets, not links themselves).
- Files matching ignore patterns (Section 16).
- Files inside .git, node_modules, or other dependency directories.
- Files with size 0 bytes (empty files are common and not
  meaningful duplicates).
- Files the agent cannot read (permission denied). Reported as
  "N arquivos não puderam ser verificados (sem permissão)."