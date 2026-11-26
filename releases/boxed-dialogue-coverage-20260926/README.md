# Scripted-Dialogue Subtitle Coverage Test (2026-09-26)

> [!NOTE]
> Historical milestone record from the 2026-09-26 subtitle coverage pass. For the current all-in-one release and build instructions, see the root [README.md](../../README.md) and [BUILDING.md](../../BUILDING.md).

## Overview

This test build inserts a `dbBoxedDisplay` call immediately before each qualifying unboxed `dbSay` call, adding **417** caption calls across **32** `COMMON.POD` SCB scripts. `LANGUAGE.POD` also receives matching Chinese text for four previously unmatched audio keys and one reused debug-scene RU text resource.

- English audio is preserved; `W32ART.POD` and `W32ENSND.POD` are untouched.
- The earlier Club Strages fix remains intact.

## Static Audit Summary

| Category | Count |
| --- | ---: |
| Total scripted speech calls | 502 |
| Already boxed | 82 |
| Newly boxed in this build | 417 |
| Intentionally unboxed (2 have no Chinese/source dialogue; 1 has no matching audio) | 3 |

Each per-call decision is logged in `audit.tsv`. Note that this is a static script audit and does not mean every scene has been visually playtested in-game.

## Installation & Rollback

The archive's `payload/COMMON.POD` and `payload/LANGUAGE.POD` contain the modified files. Close the game before installing or rolling back:

- **Install**: Run `install.ps1` to verify both original hashes, back up the existing files, and install the payloads.
- **Rollback**: Run `ROLLBACK.sh` (which invokes `rollback.ps1`) to restore both original files from the backup.
