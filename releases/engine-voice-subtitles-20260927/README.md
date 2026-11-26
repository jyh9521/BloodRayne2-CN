# Engine Voice Subtitles (2026-09-27)

> [!NOTE]
> Historical milestone record for the 2026-09-27 engine voice subtitle hook. For the current release and reproducible build guide, see the root [README.md](../../README.md) and [BUILDING.md](../../BUILDING.md).

## Overview

The existing `COMMON.POD` already contains the 127 Chinese non-Rayne scripted voice lines and is left unchanged in this release. Across the **72** engine-selected Rayne voice cue categories, **253** distinct English WAV files exist in this installation (some categories have no WAV).

- `voice_subtitles_zh.tsv` stores the per-WAV Simplified Chinese captions; `translations.py` regenerates it from the source inventory.
- Ambiguous clips were cross-checked with `faster-whisper` (`medium`) as an assistive pass rather than an authoritative transcription; nonverbal clips use short action captions.

## Technical Implementation

- **Proxy hook (`dinput8.dll`, x86)**: Retains all previous patches and hooks only the engine voice-selector call at RVA `0x151F72` in the verified `rayne2.exe`. It plays the original audio, respects the game's subtitle toggle, calls the boxed-caption function, and avoids overwriting active scripted captions.
- **Font atlas (`LANGUAGE.POD`)**: Updates only `ART\GOTHICTITLE_RU.TEX` to add **50** new glyphs. `COMMON.POD`, `W32ENSND.POD`, and `W32ART.POD` are unchanged.

## Static Caption Bounds

Each added caption adheres to the following static limits (verified offline; actual in-game stability still requires manual playtesting):

- **Encoded length**: at most 52 bytes (build ceiling: 96 bytes; engine object buffer: 256 bytes)
- **Distinct Chinese glyphs**: at most 26 (build ceiling: 80)
- **Duration**: at most 10 seconds

## Rebuild, Install & Rollback

Rebuild in place (with the original two installed files present):

```powershell
python translations.py
python build.py
powershell.exe -NoProfile -ExecutionPolicy Bypass -File build_proxy.ps1
```

- **Install**: `install.ps1` verifies source and payload SHA-256 hashes and backs up both replaced files.
- **Rollback**: `ROLLBACK.sh` or `rollback.ps1` restores both backed-up files.

The game was not launched during this build. This repository tracks build recipes and translation files rather than redistributed proprietary game archives or compiled binaries.
