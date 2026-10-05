---
name: project_save_folder_rename_plan
description: "PLANNED 2026-10-04: SixthSenseOriginal's save moves from a SixthSense folder to SixthSenseOriginal, renaming the old folder by itself on the first start, so it sits beside SixthSenseReborn's own folder."
metadata:
  type: project
---

**Status: planned 2026-10-04, agreed with the dev; not started.** Mark it finished only once the dev says it works ([[feedback_record_plans_first]]).

**Why:** Sixth Sense is now two games in two repositories ([[project_two_repos]]). SixthSenseReborn keeps its save in `SixthSenseReborn`; the dev asked that this game's folder be named after this repository too: "The SixthSense folder in appdata should be renamed to SixthSenseOriginal."

## The dev's decisions (2026-10-04)
- The save lives in `%APPDATA%\SixthSenseOriginal`, `~/.local/share/SixthSenseOriginal` on Linux, `~/Library/Application Support/SixthSenseOriginal` on macOS.
- The game **renames** the old `SixthSense` folder to `SixthSenseOriginal` by itself, on its first start after the update; players keep their progress and no stale copy is left.

## What changes
- **`sixthsense/paths.py`:** `SAVE_FOLDER = 'SixthSenseOriginal'`, `OLD_SAVE_FOLDER = 'SixthSense'`, and `rename_old_save(base)`, used by `user_dir()`: when `SixthSenseOriginal` does not exist and `SixthSense` does, rename it (one `os.rename`, the whole folder with the choosers' subfolders). If the rename fails (a file held open, no permission), log it and use the old folder for that run, so nothing is lost; the next start tries again. The tests' `SIXTHSENSE_USER_DIR` override never renames anything.
- **The choosers** (`tests/interact/`): use `SixthSenseOriginal`, renaming the real folder and their own old one first, so a chooser run before the game still finds your keys and keeps its own save.
- **Comments** naming the folder: `paths.py`, `platform/defaults.py`, `platform/keymap.py`, `tests/case/_scratch_save.py`, README.md, `docks/readme.txt`, CLAUDE.md.
- **Tests** in `tests/case/paths.py`: the three folder tests expect `SixthSenseOriginal`; new ones check the rename moves everything, leaves an existing `SixthSenseOriginal` alone, does nothing with no old folder, and falls back to the old folder when the rename fails.
- **Changelog:** one line, the save folder's new name and that it moves by itself.

## The other side
SixthSenseReborn's first start copies its save from `SixthSenseOriginal`, or from `SixthSense` for a player who has not run the updated game yet (its `project_reborn_identity_plan.md`).
