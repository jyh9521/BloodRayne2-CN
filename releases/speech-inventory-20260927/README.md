# Speech Inventory for Review (2026-09-27)

> [!NOTE]
> This is an **extraction and review pack only**. It does not install subtitles, modify POD archives, or launch the game.

## Output Files

- **`scripted_other_speech.tsv`**: The **127** newly boxed `dbStartSay` occurrences not classified as Rayne (**119** unique audio keys). Each row includes the script/line, actor token, original English subtitle, and current Chinese caption decoded from the installed `LANGUAGE.POD` carrier encoding. Repeated keys are kept separate because they appear in different scripts.
- **`engine_rayne_categories.tsv`**: All **72** engine voice-table IDs with their prefix, general meaning, WAV count, and filenames. Three prefixes are duplicated (`Rayne_XerxBDie`, `Rayne_XerxBGetHurt`, `Rayne_XerxBTaunt`), yielding **69** distinct prefixes. **24** IDs have no matching English WAV in the current archive.
- **`engine_rayne_samples.tsv`**: All **253** distinct matching WAVs (accounting for **262** sample references across the 72 IDs because the XerxB samples are referenced twice). Draft text was generated via a small-model English ASR pass and should be treated as a rough aid only—7 clips yielded no speech, and grunts/screams may produce hallucinated text. Use category, duration, and filename when reviewing. Only one WAV key appears in the shipped English text resources; the rest are audio-only and have no official script text in `LANGUAGE.POD`.

## Verification & Cleanup

- `source_hashes.json` records the read-only input hashes.
- `verify.py` rechecks output counts and confirms input hashes are unchanged.
- `ROLLBACK.sh` deletes only these generated reports from a specified absolute directory and does not touch game files.

## Scope Note

The 72 engine categories are **not** covered by the `dbStartSay` patch. This inventory was compiled to decide which engine-selected cues warrant a dedicated runtime caption hook—especially `ambient`, `hint`, `alert`, `attack`, `kill`, and `taunt` lines (which contain full spoken dialogue), as opposed to nonverbal effort or pain sounds.
