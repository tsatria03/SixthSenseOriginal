---
name: project_evaluation_2026_10
description: "Second full evaluation (2026-10-04, at 6d964dc on main, custom 6 commits ahead): new restart/pause bugs, the macOS PR's releaser gap, stale doc facts, and what matters for moving custom to its own repo."
metadata:
  type: project
---

Read-only evaluation on 2026-10-04 at `6d964dc` (main, after the macOS PR #1, `425bbf9`, by mzanm), done by three parallel reviewers (platform and tooling, game logic, docs and branches). Nothing was edited or run except the test suite. Follows [[project_evaluation_2026_09]], whose items are all settled. [V] = Claude checked it in the code; others are a reviewer's reading. Line numbers drift; re-locate by symbol.

**Tests:** the full suite passed 409 of 409 across 26 files in 242 s on main ([[project_safe_test_run]]).

## Branches
- **Update, 2026-10-04: `custom` moved to its own repository, `github.com/tsatria03/SixthSenseReborn`**, with its full history (pushed as Reborn's `main`, 237 commits, no tags), and was then deleted from this repository, on GitHub and locally. This repository now has only `main`, the faithful port; the old `seventh-sense` redirect is gone with the branch. The notes below describe the state before the move.
- `main` is the faithful port; `custom` is 6 commits ahead, branched at `6d964dc`, nothing behind. Custom's code diff is comments only (references to the deleted DIVERGENCES.md and PORTING_STATUS.md). See `project_custom_branch.md` on custom.
- The dev plans to move `custom` to its own repository (to be discussed after this evaluation). Things that matter for that move:
  - It carries all of main's history and the `V26.09.xx` tags; don't push tags to the new repo.
  - The same save folder (`paths.py` user_dir, `%APPDATA%\SixthSense`) and the same chooser saves, so the two games overwrite each other.
  - `releaser.py` makes the same tag `V<version>`, title "SixthSense V<version>" and asset names, and `VERSION` (26.09.28-2) plus main's released changelog sections; `gh` publishes to whatever `origin` is.
  - `compiler.py` build names are the same as main's.
  - Custom's notes still link `[[project_evaluation_2026_09]]` and name DIVERGENCES/PORTING_STATUS, which custom deleted (compiler, dev tasks, levels endless, safe test run, tests layout, binary analysis notes, sound organization, screen reader mode, test range, and most plan notes).
  - The first custom commit's message (`c99d25f`) still says "seventh-sense"; the rename was undone in `b0aa6ed`.
  - The repository was renamed from `SixthSense-Windows` to `SixthSenseOriginal`; on main the README title and links and the notes were updated to the new name on 2026-10-04. The build folder `dist\SixthSense-Windows` is a build name, not the repo's, and was left alone.

## Game bugs, new since the September evaluation
1. [V] **High: Restart after the first game breaks it.** After a first Start, the tutorial counts into the real game on the same `Stage_Tutorial` object (`tutorialEndGameStart_`), so the panel's Restart (`Stage_1_E.gameReplayAction_` -> `self.MapInitInBundle()`) runs the tutorial's override: beat One again with `finished` set, so `CheckTutorial` and P do nothing and only Escape leaves (coin spent). The original runs the first game in a plain `Stage_1_E` (0x33bec), so this is a port bug. Untested.
2. **High: Restart doesn't cancel pending performs.** `gameReplayAction_` cancels only `ChangeLevel_`; a pause in the 1.3 s before `playerDie_` lets it fire into the new run, and `missionCompletSounding` is never reset (`_reset_run_flags`).
3. **High: the test range can't die from a double hit.** `stage_1_test.py` checks `HP == 0`; two zombies on one tick at 1 heart give -1.
4. Medium: `StopElseSpeak` on the panel doesn't cancel `ReadTopScore` (row 10 reads 2 s late, over the next row).
5. Medium: `ESCAPE_LEAVES` stays True after the tutorial becomes the real game, so Escape quits instead of pausing, and the debug keys refuse.
6. Medium: a grab (`shakeMonsterTimer`, `NonShaking`) runs on through a pause and a restart; restart leaves `shakeFlag`/`heldMonster`.
7. Low: the test range's row 6 runs `continueAction_` and kills Restart for that panel; `oal_playback.stopSound_` reads `alGetError` twice (the check never fails); "has completed" vs "has been completed" wording; an always-true `m.HP <= 0` in `_level_transition`.
8. Suspected: `readStop` marks all ten digit sounds playing, which may exhaust the 122 sources in a long session.

Suggested fix: one `_cancel_run_timers()` used by pause, restart and teardown (bugs 2 and 6), and the first game leaving tutorial mode fully (clear `ESCAPE_LEAVES` per instance, restart through `Stage_1_E.MapInitInBundle`) for 1 and 5.

## Platform, macOS and tooling
- [V] **Medium: `releaser.py` can't package a macOS build**: the app keeps `VERSION` inside the bundle, so `built_version()` finds none. `--embed` is ignored on macOS but still checked. The app build skips `release_warnings()`. No test covers `move_app_build` or finding data inside a built app.
- Medium: `openal.py` declares the one-byte `ALCboolean` returns as `c_int` (open dev task).
- Low: OpenAL never closed (`tearDownAudio` has no callers); `volume.py` uses `isdigit` (accepts "²", then `int()` fails; use `isdecimal`); `keys.json` has no .bak and a non-object is overwritten; a bad `--game` falls back silently; stale "Windows" comments in `SixthSense.py`.
- `tests/case/release.py` folder-build test now passes `console=True`, so the windowed Windows command is no longer checked.
- [V] **The blooper clip ships in builds**: it is at `game/sounds/unused/bloopers/`, and the compiler copies all of `unused/`. CLAUDE.md says `game/sounds/bloopers/`, left out.

## Docs drift (main)
- Sound counts: the tree has 198 in `used/` and 99 in `unused/`; CLAUDE.md says 195/101, `project_sound_organization` 195/101, MEMORY.md and README.md 236/126, `project_compiler_py` 507 files.
- macOS not carried into CLAUDE.md: "A Windows port", "a Windows voice", vendor dylib and `vendor/openal/` path, the macOS dist and save path, requirements. README title and `docks/readme.txt:4` still say Windows.
- `project_dev_tasks.md` still cites 309 tests.

## Code health
- About 7,800 lines; `stage_1_e.py` is 1,980 and would split into tables, the pause/result panel (~500) and the core loop.
- Subclassing `Stage_Tutorial` and `Stage_1_TEST` from `Stage_1_E` mirrors the original but inherits behaviour they shouldn't; that is the root of bugs 1, 3 and 5.
- Dead code mirrors original methods (`oal_playback` note helpers, `moving_accelerometer`, several `stage_1_e` fields). Duplication: the NSString number parser (twice), the coin clock read (three times), `_say` (three times), weapon page helpers, the two `gameReplayAction_`, and the `MotionSamplingTimer` stop block (six times).
