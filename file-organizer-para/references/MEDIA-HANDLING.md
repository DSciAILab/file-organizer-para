# Media Handling Reference

## Detection (Step 2.2)

### Image extensions
.jpg, .jpeg, .png, .heic, .heif, .webp, .gif, .bmp, .tiff, .tif,
.raw, .cr2, .nef, .arw, .dng, .svg

### Video extensions
.mov, .mp4, .avi, .mkv, .wmv, .flv, .webm, .m4v, .mpg, .mpeg,
.mts, .m2ts, .ts, .3gp

### Audio extensions
.mp3, .flac, .wav, .aac, .ogg, .oga, .opus, .m4a, .wma, .aif,
.aiff, .alac, .ape

### Screenshot detection
Filenames matching patterns: "Screenshot*", "Screen Shot*",
"Captura*", "Bildschirmfoto*", "screen-*".
Or metadata indicating screen capture.


## Metadata extraction

### EXIF (images)
Tools: exiftool, mdls (macOS), identify (ImageMagick).
Fields: DateTimeOriginal, GPSLatitude/Longitude, Make, Model,
ImageWidth, ImageHeight.
Tool unavailable = NOT_CHECKED. Group by file modification date
with warning.

### Audio/Video metadata
Tools: exiftool, ffprobe, mediainfo.
Fields: title, artist, album, year, track, duration, codec.
If available, suggest rename: YYYY_Artist_Album_Title.ext.

### Metadata availability
- Tool present and metadata found: use it.
- Tool present, no metadata: fall back to file modification date.
- Tool absent: NOT_CHECKED. Use file modification date with warning.


## Classification rules

### Photos associated with a project
1. Photo inside a recognized project folder: move with project.
   Confidence: High.
2. Keyword analysis matches photo name to a project/area:
   suggest association. Confidence: High.
3. EXIF date falls within active period of a known project:
   suggest association. Confidence: Medium. Ask user.
4. No association found: apply default media handling below.

### Default media handling presentation

```
MEDIA DETECTED: 847 images, 23 videos, 12 audio

Screenshots: 156 files
  -> Suggestion: 3-Recursos/Screenshots/YYYY-MM/
  -> [accept] [other destination] [ignore]

Photos with EXIF date: 614 files
  Periods: 2023-06 to 2026-02
  -> Suggestion: 5-Fotos/YYYY/YYYY-MM_Event-Name/
     (standalone library, separate from PARA categories)
  -> [accept] [other destination] [keep flat] [ignore]
  Note: you will be asked to name each event group, or
  accept auto-generated names from EXIF date clusters.

Photos without date: 77 files
  -> Suggestion: Legado-pre-organizacao/Fotos-sem-data/
  -> [accept] [other destination] [ignore]

Videos: 23 files (14.2 GB)
  -> Suggestion: follow same structure as photos
  -> [accept] [separate from photos] [ignore]

Audio: 12 files
  -> Suggestion: 3-Recursos/Music/ or rename by metadata
  -> [accept] [rename by metadata] [other] [ignore]
```


## User preference: standalone vs PARA vs external app

On first run with media detected, ask:
"Do you already have a system for photos (Apple Photos, Google
Photos, a dedicated folder)? If yes, should I ignore photos and
focus on other files?"

Options:
(a) Organize photos in standalone library (recommended).
    Creates: <root>/5-Fotos/YYYY/YYYY-MM_Event-Name/
    Photos live outside PARA categories (not in Archive).
(b) Organize photos within PARA structure (advanced).
    Uses: 4-Arquivo/Fotos/YYYY/YYYY-MM_Event-Name/
    Only recommended if user wants all files under one PARA tree.
(c) Ignore photos entirely. Focus on documents only.

Store preference in .para-config.json under "mediaPolicy".
Default: option (a).

### Why option (a) is the default

Photos are a memory library, not archived inactive documents.
Placing them in 4-Arquivo implies they are inactive, which
conflicts with how users access and browse their photo collection.
A standalone 5-Fotos/ tree with chronological structure gives
photos their own logical space while keeping PARA categories clean.


## Photo library structure (option a - recommended)

```
5-Fotos/
  2024/
    2024-01_Viagem-Argentina/
    2024-03_Casamento-Joao-Ana/
    2024-07_Cotidiano/
    2024-12_Natal-Familia/
  2025/
    2025-02_Conferencia-Lisboa/
    2025-07_Ferias-Nordeste/
    2025-08_Cotidiano/
  2026/
    2026-01_Viagem-Japao/
    2026-03_Cotidiano/
```

Folder naming: YYYY-MM_Event-Name (hyphen-separated, ASCII safe).
Files inside: renamed by EXIF date when user approves.
See RENAMING.md for photo rename convention and batch flow.

### Event name assignment

When grouping photos by EXIF date clusters, agent suggests
event names based on:
1. GPS location data (if available and user opts in).
2. Keyword analysis of existing filenames in the cluster.
3. Auto-generated name from date: YYYY-MM_Fotos or YYYY-MM_Photos.

Agent always asks user to confirm or edit event names before
creating folders. Never creates folders with auto-generated names
silently.


## Photos inside projects

Photos associated with a project (by folder, keyword, or EXIF date)
stay inside the project folder, not in 5-Fotos/:

```
1-Projetos/
  2026-03_Lancamento-Produto-X/
    03_Assets/
      fotos-produto/        <- project photos stay here
        2026-03-05_001.jpg
```

When a project is archived to 4-Arquivo/, project photos move
with the project. No special handling needed.


## Deduplication notes for media

Photos are the primary use case for dedup. Common scenarios:
- Same photo downloaded from WhatsApp/Telegram multiple times.
- Screenshot of screenshot.
- Export from iCloud/Google Photos creating copies.

Hash-based dedup (SHA-256) catches exact byte-for-byte copies.

### Perceptual hash (opt-in, for recompressed photos)

Recompressed photos (WhatsApp, Telegram, cloud export) have
different SHA-256 despite appearing identical. Hash-based dedup
will NOT catch these.

Tools required: ImageMagick (identify) or Python with imagehash.
If tools unavailable: NOT_CHECKED, skip perceptual hash with warning.

When tools available, offer during media triage:
"Enable perceptual hash for image dedup?
 Detects visually similar photos even if recompressed.
 Slower but catches WhatsApp/cloud export duplicates.
 -> [yes] [no, hash only]"

Perceptual hash threshold: Hamming distance <= 10 (configurable).
Results reported as PROBABLE_VISUAL_DUPLICATE, never auto-archived.
User must confirm each pair before any action.


## Audio rename suggestion format

When audio metadata is available:

```
Audio files with metadata: 8 files
  01-unknown.mp3 -> 2024_Artist-Name_Album-Title_Track-Name.mp3
  track02.flac -> 2023_Band-Name_Live-Album_Song-Title.flac

  -> [accept all renames] [review individually] [skip renames]
```

Rename is always optional. Never auto-rename without approval.
See RENAMING.md for full rename conventions.
