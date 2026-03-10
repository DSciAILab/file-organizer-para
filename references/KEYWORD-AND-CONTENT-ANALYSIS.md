# Keyword and Content Analysis Reference

## Part 1: Keyword Analysis (Step 2.3)

### Purpose
Extract meaningful tokens from filenames, identify recurring patterns,
and use them to boost PARA classification confidence.

### Algorithm

#### Phase 1: Tokenization
- Split filename by delimiters: underscore, hyphen, space, dot
  (except final extension), camelCase boundaries.
- Normalize: lowercase.
- Preserve: acronyms (ALL-CAPS sequences >= 2 chars), dates
  (4-digit years, YYYY-MM patterns, Q1-Q4), version numbers (v1, v2).
- Remove: pure numbers < 3 digits, single characters.

#### Phase 2: Frequency
- Count occurrence of each token across all scanned files.
- Discard tokens with frequency 1 (no group formed).
- Discard filesystem stopwords: "copy", "copia", "final", "new",
  "old", "backup", "temp", "tmp", "draft", "untitled", common
  extensions (pdf, doc, jpg, png, xlsx, etc.).

#### Phase 3: Clustering
- Group files sharing high-frequency tokens (threshold: 3+ files).
- Rank clusters by file count.
- Cross-reference each token against profile aliases.
- If token matches a profile alias, auto-associate with that
  area/project. Confidence: High.
- If token is frequent but not in profile: present as new cluster.
  Confidence: Medium.

#### Phase 4: Presentation

```
KEYWORD ANALYSIS - Patterns found in file names

Tag: SJJP (8 files) -> matches profile: Work-SJJP
  Examples: SJJP_Coach_Attendance-2026.xls, relacao-de-staff-SJJP.docx
  -> Auto-classified as: Areas/Work-SJJP (High confidence)
  -> [confirm] [change] [ignore]

Tag: invoice (5 files) -> no profile match
  Examples: invoice-acme-jan.pdf, invoice-acme-feb.pdf
  -> Suggestion: Area? Resource?
  -> [area: Financas] [resource: Templates] [new area] [ignore]

Tag: 2026 (23 files) -> temporal marker
  -> Used for date extraction, not classification.
  -> [confirm] [ignore]

Tag: ACME (4 files) -> no profile match
  Examples: invoice-acme-jan.pdf, contract-acme-2025.pdf
  -> Suggestion: Client or project?
  -> [project] [area] [add to profile as alias] [ignore]
```

### Integration with profile
- Validated clusters update profile aliases for future runs.
- Example: user confirms ACME as client under Areas/Financas.
  "acme" is added to Financas aliases in .para-config.json.

### Consistent classification mode
When processing large batches:
- After N files (default 5) classified to the same category with
  the same token, subsequent matches auto-suggest the same category.
- User can override individually.
- Reduces triage fatigue on repetitive file sets.


## Part 2: Content Analysis (Step 2.4)

### Purpose
Read file content to improve classification for files where name
and keywords alone are insufficient.

### Three levels

**Level 0 (default)**: Name + extension + metadata only.
No file content read. Used in Quick and Safe scan modes.

**Level 1 (shallow)**: Read first 50 lines of text files, or first
page of PDFs (if parser available). Extract keywords from content.
Applied only to files with Low classification confidence.
Used in Deep scan mode automatically.

**Level 2 (deep)**: Read complete content of selected files.
Only on explicit user request, file by file or batch-approved.
Never automatic.

### Privacy rules
1. Agent must announce before reading any file content:
   "I will read the content of N files to better classify them.
   Confirm? (yes / no / select which files)"
2. User can exclude specific files or file types.
3. Content is used only for classification. Not stored in manifest
   or index. Only extracted keywords/tags are stored.
4. Sensitive patterns detected (SSN, credit card numbers, API keys)
   are never recorded. Agent notes "sensitive content detected" and
   moves on.

### Supported formats

Text: .txt, .md, .csv, .json, .xml, .yml, .yaml, .ini, .cfg,
  .conf, .log, .html, .htm, .tex, .rst.
PDF: .pdf (requires parser: pdftotext, mdls, or similar).
Office: .docx, .xlsx, .pptx (requires unzip + XML parse or similar).
Not supported: .doc, .xls, .ppt (legacy binary), images (use
  media handling instead), audio/video (use metadata instead).

### Output
Content analysis produces:
- Extracted keywords (top 10 by relevance).
- Suggested category with confidence level.
- Correspondent if detected (sender name, organization).
- Brief description (1 sentence) for index entry.

These feed into the plan (Step 4) and the index (para-index.jsonl).

### Performance
- Level 1: ~1-3 seconds per file depending on size.
- Level 2: varies widely. Agent should estimate and warn.
- > 100 files at Level 1: warn about duration.
- > 500 files at Level 2: strongly recommend limiting scope.


## Part 3: Triage Session (Step 2.6)

### Purpose
Present ALL accumulated uncertainties from Steps 2.2-2.5 in one
consolidated session before planning. Reduces mid-workflow interruptions.

### Format

```
TRIAGE - I need your help with N decisions before building the plan.

FOLDERS (4 decisions):
1. Downloads-antigos/ (23 files, mixed content)
   -> [classify individually] [move whole to ___] [skip]

2. SJJP/ (15 files, cohesive cluster detected)
   -> [project] [area] [move whole to ___] [split]

CLASSIFICATION (6 decisions):
3. contrato-locacao.pdf -> Area, but which?
   -> [Areas/Casa] [Areas/Financas] [other: ___] [skip]

4. SJJP_budget_Q1.xlsx -> Project SJJP or Area Financas?
   -> [project: SJJP] [area: Financas] [skip]

PHOTOS (2 decisions):
5. 12 photos with EXIF date matching Reforma-cozinha project
   -> [associate with project] [keep in Photos/2025/] [skip]

6. 832 remaining photos without association
   -> [organize by YYYY/MM] [move to Legacy] [skip]

DUPLICATES (1 decision):
7. 14 groups found (37 files, 2.3 GB)
   -> [review by group] [archive all duplicates] [ignore]

Auto-decide high-confidence items and only ask the rest?
-> [yes, decide High confidence] [no, show everything]
```

### Interaction modes
- Guided: agent decides High confidence, asks Medium and Low.
- Full control: agent asks every decision.
- Full trust: agent decides everything, shows final plan only.

Mode selected during onboarding, changeable anytime via
"update areas" or at the start of any triage session.