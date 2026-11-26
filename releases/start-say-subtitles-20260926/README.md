# Asynchronous Scripted Speech Subtitle Test (2026-09-26)

> [!NOTE]
> Historical milestone record from the 2026-09-26 `dbStartSay` subtitle pass. For the current release and build instructions, see the root [README.md](../../README.md) and [BUILDING.md](../../BUILDING.md).

## Overview

This test build inserts `dbBoxedDisplay(key)` immediately before **210** existing `dbStartSay(key)` commands across **28** `COMMON.POD` SCB scripts:

- **83** Rayne speech occurrences
- **127** other scripted speech occurrences
- **11** `dbStartSay` calls already had boxed captions
- **3** calls were intentionally skipped (2 have no caption text in the Chinese resource; 1 has no matching English audio)

All added captions reuse the existing `LANGUAGE.POD` translations and font rendering path from the preceding scripted-speech patch. No voice or font archives are modified.

## Scope Limitation

This build covers SCB-scripted asynchronous speech only. `engine_cue_inventory.tsv` separately lists **72** engine-selected Rayne voice categories (such as `Rayne_Taunt`, `Rayne_Ambient*`, and `Rayne_Kill`), which are not triggered via SCB `dbStartSay` and require a separate runtime cue-to-caption hook.

## Installation & Rollback

Close the game before installing or rolling back:

- **Install**: Run `install.ps1` to verify the previous `COMMON.POD` hash, back it up, and install the payload.
- **Rollback**: Run `ROLLBACK.sh` (which invokes `rollback.ps1`) to restore the prior version.

The game was not launched during this build; in-game visual timing still needs manual playtesting.
