# PARA Method Reference

## Source

Tiago Forte, "The PARA Method" (fortelabs.com/blog/para/) and the
2023 book (ISBN 978-1982167585).

## Core principle

Organize by actionability, not by topic or file type.

## Four categories

Projects: short-term efforts with a specific outcome and deadline.
  Examples: file tax return 2026, kitchen renovation, SJJP annual event.

Areas: ongoing responsibilities with a standard to maintain.
  Examples: finances, health, career, family, home, work-SJJP.

Resources: topics of interest or reference material.
  Examples: investment research, recipe collection, ML tutorials.

Archive: inactive items from any of the above three categories.
  Examples: completed projects, inactive areas, outdated resources.

## Decision rules

1. Specific result with deadline -> Projects.
2. Continuous responsibility without end date -> Areas.
3. Reference or learning material -> Resources.
4. Inactive, completed or outdated -> Archive.

## Areas vs Projects (the common confusion)

An ongoing program (like SJJP) is an Area. Events within that program
(SJJP Annual Event 2026) are Projects within that Area.

The question to ask: "Does this have a completion date?"
- Yes -> Project.
- No -> Area.
- It is ongoing but has sub-items with deadlines -> Area with subProjects.

## Profile schema

Stored in .para-config.json under "profile":

```json
{
  "profile": {
    "areas": [
      {
        "name": "Financas",
        "aliases": ["finance", "banco", "imposto", "tax", "invoice"],
        "subProjects": []
      },
      {
        "name": "Trabalho-SJJP",
        "aliases": ["sjjp", "coach", "attendance", "staff"],
        "subProjects": [
          {
            "name": "SJJP-Evento-anual-2026",
            "deadline": "2026-09-15",
            "keywords": ["evento", "annual-event", "event"]
          }
        ]
      }
    ],
    "activeProjects": [
      {
        "name": "Imposto-2026",
        "area": "Financas",
        "keywords": ["irpf", "declaracao", "receita-federal"]
      }
    ],
    "resources": [
      {
        "name": "Templates",
        "keywords": ["template", "modelo"]
      }
    ],
    "interactionMode": "guided",
    "createdAt": "2026-03-10T14:00:00Z",
    "updatedAt": "2026-03-10T14:00:00Z"
  }
}
```

## Onboarding flow

1. Present default areas: Health, Finances, Career, Family, Home,
   Personal Development, Hobbies, Work.
2. User adjusts: remove, add, rename, merge.
3. For each item: "Has end date?" -> Area or Project.
4. Option (c): "Ongoing with sub-projects" -> Area with subProjects.
5. Ask for active projects with deadlines.
6. Ask for resource topics.
7. Ask for interaction mode: Guided, Full control, Full trust.
8. Save profile.

## Classification examples

| Item | Category | Reasoning |
|------|----------|-----------|
| Current tax return | Projects | Specific deadline |
| Lease contract | Areas/Home | Ongoing responsibility |
| Car manual | Areas/Vehicle | Ongoing reference for owned asset |
| Template proposal | Resources/Templates | Reusable reference |
| Old travel photos | Archive/Photos | Inactive |
| SJJP attendance sheet | Areas/Work-SJJP | Ongoing program |
| SJJP annual event plan | Projects | Has deadline |
| ML course notes | Resources/ML | Learning material |
| Completed renovation docs | Archive/Projects | Completed project |

## Naming conventions

### Folders
- Projects: YYYY-MM_Descriptive-name (e.g., 2026-03_Kitchen-renovation)
- Areas: Descriptive-name (e.g., Financas, Work-SJJP)
- Resources: Descriptive-name (e.g., Templates, ML-tutorials)
- Archive sub-folders: mirror original structure
- Sub-folders within: NN_Name (e.g., 01_Orcamentos, 02_Contratos)

### Files
- YYYY-MM-DD_Description.ext (e.g., 2026-03-10_Tax-receipt.pdf)
- Version: -v1, -v2 (never "FINAL" or "final-final")
- Hyphens between words, no spaces.
- ASCII safe: no accents in filenames (contrato, not contráto).
- Preserve original extension exactly.
- Collision: append -duplicata-N.
- Windows: validate reserved names (CON, PRN, NUL, COM1-9, LPT1-9).

### Unicode policy
- Non-latin characters in filenames: FLAGGED for user decision.
- Never auto-rename Unicode names. Report and ask.
- If user approves transliteration, record original name in manifest.

### Legacy-pre-organization
- Files moved from unsorted sources keep original names.
- No automatic rename on import. Rename only with explicit approval.
- Legacy folder serves as triage holding area.

## Folder hierarchy example

```
~/Documents/PARA/
  Work/
    1-Projetos/
      2026-01_SJJP-Evento-anual/
        01_Planning/
        02_Budget/
        03_Photos/
      2026-03_Server-migration/
    2-Areas/
      Trabalho-SJJP/
        Attendance/
        Staff/
      Carreira/
    3-Recursos/
      Templates/
      Tutorials/
    4-Arquivo/
      Projetos-concluidos/
        2025-06_Office-renovation/
      Duplicados/
        2026-03-10/

  Personal/
    1-Projetos/
    2-Areas/
      Financas/
      Familia/
      Casa/
    3-Recursos/
    4-Arquivo/
```

## Ambiguity handling

When classification is unclear:
1. Check profile aliases for keyword match.
2. Check keyword analysis clusters.
3. If still ambiguous, present options in triage session.
4. If user chooses "decide later", move to Legacy-pre-organization.
5. Batch similar decisions: "These 12 files all match 'SJJP'.
   Treat all as Work-SJJP Area? (yes / review individually)"

## Cross-category files

A file belongs in ONE physical location. If it relates to multiple
categories, the primary location is chosen by action priority
(Project > Area > Resource > Archive). Additional associations are
recorded as tags in the index (para-index.jsonl) for cross-search.

Example: lease contract lives in Areas/Home but is tagged
["financas", "casa", "contrato"] in the index. Searching "financas"
finds it even though it is physically in Areas/Home.