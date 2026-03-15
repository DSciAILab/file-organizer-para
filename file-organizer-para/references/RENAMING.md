# Renaming Reference

## When to suggest rename

Suggest rename when ANY of the following conditions is true:

- Name matches a generic OS/app auto-generated pattern:
  IMG_XXXX, DSC_XXXX, DCIM_XXXX, P_YYYYMMDD_HHMMSS,
  scan0NNN, Digitalized_NNN, Screenshot*, Screen Shot*,
  Captura*, Bildschirmfoto*, screen-*, Untitled, Document(N),
  NONAME, sem-titulo, WhatsApp Image*, WhatsApp Video*,
  signal-YYYY-MM-DD*, telegram-*
- Name contains spaces (cross-platform portability risk)
- Name contains accented or special characters
  (cross-platform portability risk)
- Name contains version noise: "final", "Final", "FINAL",
  "-last", "v2-final", "final-final", "definitivo"
- Name has trailing copy markers: "(1)", "(2)", " - Copy",
  "-copia", "-backup", "-old", "-bkp"
- Extension is uppercase: .PDF, .JPG, .DOCX
  -> normalize to lowercase

Do NOT suggest rename when:
- File is inside Legado-pre-organizacao/ or Quarentena/
- File has non-latin Unicode characters (FLAGGED, ask human)
- User set "skip renames" in current session
- File is a software project config or executable script
- File has no extension and binary content


## File naming convention

Format:   YYYY-MM-DD_Description-words_v01.ext
Example:  2026-03-15_Proposta-Cliente-Alpha_v02.pdf

Rules:
- Date prefix: YYYY-MM-DD (enables lexicographic sort = date sort)
- Words: hyphen-separated, no spaces, no underscores mid-name
- ASCII only: no accents (contrato not contráto)
- Version suffix: -v1, -v2 ... never "FINAL" or "final-final"
- Extension: always lowercase, never modify extension content
- Collision: append -duplicata-N (e.g. relatorio-duplicata-2.pdf)
- Windows reserved names: never use as filename or folder name
  (CON, PRN, AUX, NUL, COM1-COM9, LPT1-LPT9, case-insensitive)


## Folder naming convention

Projects:    YYYY-MM_Descriptive-Name
             (e.g. 2026-03_Kitchen-Renovation)
Areas:       Descriptive-Name  (e.g. Financas, Work-SJJP)
Resources:   Descriptive-Name  (e.g. Templates, ML-Tutorials)
Subfolders:  NN_Name           (e.g. 01_Briefing, 02_Assets)
Archive:     mirror original structure. never rename on archive.
_Inbox:      always underscore-prefixed to sort at top of listing.


## Version series vs accidental duplicate

These two cases look similar but require opposite actions.

### Version series (intentional, keep all)

Signals in filename: v1/v2/v3, _r1/_r2/_r3,
-draft/-review/-approved, -rascunho/-revisao/-aprovado,
-01/-02/-03 when accompanied by same base name.

Action:
- Classify as VERSION SERIES, not duplicate.
- Do NOT suggest archiving without explicit user request.
- Suggest grouping under a NN_Versoes/ or NN_Revisions/
  subfolder inside the parent project folder.
- Optionally offer: "Archive all versions except latest?"
  Only execute with explicit approval.

### Accidental duplicate (unintentional copy, archive the copy)

Signals in filename: (1), (2), -copia, -copy, -duplicata,
" 2" suffix, or exact SHA-256 hash match regardless of name.

Action:
- Classify as ACCIDENTAL DUPLICATE.
- Suggest archiving the copy, keeping the canonical file.
- See DUPLICATE-DETECTION.md for full dedup workflow.


## Photo rename (when EXIF DateTimeOriginal available)

Format:   YYYY-MM-DD_NNN.ext
Example:  2025-07-14_001.jpg

Rules:
- NNN: zero-padded sequence counter, resets per date.
- Extension: preserved as-is (RAW extensions especially: .NEF, .CR2).
- GPS opt-in: if user approves, append location token:
  2025-07-14_Lisboa_001.jpg
- No EXIF available: fall back to file modification date,
  append _noexif tag as warning:
  2025-07-14_noexif_001.jpg
- Never auto-rename photos without explicit approval.
- Always show before/after sample before batch rename:
  "IMG_4821.jpg -> 2025-07-14_001.jpg  (EXIF: 2025-07-14 09:32)"
  "IMG_4822.jpg -> 2025-07-14_002.jpg  (EXIF: 2025-07-14 09:33)"
  -> [accept all] [review individually] [skip photo rename]


## Audio rename (when metadata available)

Format:   YYYY_Artist_Album_Track.ext
Example:  2024_Artist-Name_Album-Title_Track-Name.mp3

See MEDIA-HANDLING.md for full audio rename workflow and
batch presentation format.


## Transliteration rules (ASCII normalization)

Applied only when user approves rename.
Original name always recorded in manifest and index.

  á / à / ã / â  ->  a
  é / ê           ->  e
  í               ->  i
  ó / ô / õ       ->  o
  ú / ü           ->  u
  ç               ->  c
  ñ               ->  n

All other non-ASCII characters: FLAGGED, never auto-transliterate.
Present to user individually with suggested ASCII equivalent.


## Batch rename

When >= 10 files share the same rename pattern:
1. Group files by pattern (IMG_*, scan*, WhatsApp*, etc.).
2. Show sample of 3 before/after pairs:
   "47 files match IMG_XXXX pattern.
    Sample:
      IMG_4821.jpg -> 2025-07-14_001.jpg
      IMG_4822.jpg -> 2025-07-14_002.jpg
      IMG_4823.jpg -> 2025-07-14_003.jpg
    Rename all 47? (yes / review individually / skip)"
3. Execute in batches of batchWriteSize.
4. Report progress after each batch.
5. Any rename failure: mark failed, continue, report at end.


## Rename in Refine mode

When operating in Refine mode on an already-organized PARA folder:
- Suggest rename only for files that violate naming convention.
- Never rename files that already follow YYYY-MM-DD_* pattern.
- Prioritize sub-folder structure improvements over renames.
- Batch similar renames and ask once per pattern group.


## Manifest recording for renames

Every rename is recorded in the manifest with:
- originalName: the name before rename
- newName: the name after rename
- renamed: true (attribute on the operation, not a type)
- originalNamePreserved: in para-index.jsonl for searchability

"where was" and "trace" commands search both originalName
and newName fields across all manifests.
