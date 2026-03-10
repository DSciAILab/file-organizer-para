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
  -> Suggestion: 3-Recursos/Screenshots/ (organized by month)
  -> [accept] [other destination] [ignore]

Photos with date (EXIF): 614 files
  Periods: 2023-06 to 2026-02
  -> Suggestion: 4-Arquivo/Fotos/YYYY/YYYY-MM/
  -> [accept] [other destination] [keep flat] [ignore]

Photos without date: 77 files
  -> Suggestion: Legacy-pre-organization/Fotos-sem-data/
  -> [accept] [other destination] [ignore]

Videos: 23 files (14.2 GB)
  -> Suggestion: follow same structure as photos
  -> [accept] [separate from photos] [ignore]

Audio: 12 files
  -> Suggestion: 3-Recursos/Music/ or rename by metadata
  -> [accept] [rename by metadata] [other] [ignore]
```

### User preference: standalone vs PARA
On first run with media detected, ask:
"Do you already have a system for photos (Apple Photos, Google
Photos, dedicated folder)? If yes, should I ignore photos and
focus on other files?"

Options:
(a) Organize photos within PARA structure (default).
(b) Organize photos in standalone structure (~/Photos/YYYY/MM/).
(c) Ignore photos entirely.

Store preference in .para-config.json under "mediaPolicy".

## Photos and project archival

When a project is archived, photos inside the project folder
move to Archive with the rest of the project. No special handling
needed since they are part of the atomic project folder.

## Deduplication notes for media

Photos are the primary use case for dedup. Common scenarios:
- Same photo downloaded from WhatsApp/Telegram multiple times.
- Screenshot of screenshot.
- Export from iCloud/Google Photos creating copies.

Warning: recompressed photos may have different hash despite
appearing identical. The skill's hash-based dedup will NOT catch
these. Report this limitation in the dedup report:
"Note: visually similar but recompressed photos are not detected
as duplicates by hash comparison."

## Audio rename suggestion format

When audio metadata is available:

```
Audio files with metadata: 8 files
  01-unknown.mp3 -> 2024_Artist-Name_Album-Title_Track-Name.mp3
  track02.flac -> 2023_Band-Name_Live-Album_Song-Title.flac

  -> [accept all renames] [review individually] [skip renames]
```

Rename is always optional. Never auto-rename without approval.