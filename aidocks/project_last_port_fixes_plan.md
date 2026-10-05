---
name: project_last_port_fixes_plan
description: "FINISHED 2026-10-05, confirmed by the dev: the last fixes in SixthSenseOriginal, the four port bugs from the second evaluation (the first game after the tutorial restarting into it, Escape quitting that game, a superscript digit in settings.json, the purchase line's wording). The other four of that evaluation's bugs are the original's own and stay, recorded as reproduced."
metadata:
  type: project
---

**Status: FINISHED 2026-10-05, confirmed by the dev** ("All 4 bugs past"). The covering tests pass: tutorial.py 20, volume.py 15, store.py 20, input.py 27, menu.py 34. Mark it finished only once the dev says it works ([[feedback_record_plans_first]]).

**Why:** The dev wants this repository left as faithful as possible, and these are the last changes they plan to make here; new work goes to SixthSenseReborn ([[project_two_repos]]). The second evaluation ([[project_evaluation_2026_10]]) listed eight bugs a player can notice. A read-only check of the binary on 2026-10-05 found four of them are the 2013 game's own; the dev chose: "Please fix only bugs 1, 4, 7 and 8."

## The four port bugs, fixed
1. **Restarting after the first game puts you back in the tutorial.** After a first Start, the tutorial's countdown starts the real game on the same `Stage_Tutorial` object (`tutorialEndGameStart_`), so the panel's Restart (`gameReplayAction_` -> `self.MapInitInBundle()`) runs the tutorial's override: beat One again, with `finished` set. In the original the first game is a plain `Stage_1_E` running the tutorial inline (`-[Stage_1_E tutorialEnd:]` 0x33bec, `tutorialEndGameStart:` 0x33d40), so its restart is an ordinary one. **Fix:** `tutorialEndGameStart_` marks the tutorial over (`in_game`), and `Stage_Tutorial.MapInitInBundle` then does only `Stage_1_E`'s, with `isTutorial` left at 1. Every other tutorial method already does nothing once `finished` is set, and `Stage_1_E` stops calling the tutorial's hooks once `isTutorial` is 1.
4. **Escape in that first game goes back to the menu.** `ESCAPE_LEAVES` is set on the class, so it stays true after the countdown, and the debug keys refuse too. Escape is the port's own key. **Fix:** `tutorialEndGameStart_` sets `ESCAPE_LEAVES = False` on the object, so Escape pauses, as in any game.
7. **A volume in settings.json with an unusual digit stops the game from starting.** `volume.py`'s `_whole` used `str.isdigit()`, which accepts characters such as "²" that `int()` rejects. settings.json is the port's own file. **Fix:** `isdecimal()`, which accepts only what `int()` reads, so such a value counts as unusable (100), like any other.
8. **The spoken purchase line is the wrong one.** The binary has both strings: "Purchase has completed." (0xb7d86) is what the weapon buy shows (`-[DetailStoreController buyAction:]` `setText:` 0x1bfc4), and "Purchase has been completed." (0xb6f3e) belongs only to the in-app purchase alerts. The port's spoken text for recording 260 (`blind_screen.MESSAGE_TEXT`), a port addition used only for the weapon buy (0x1bf9c), took the alerts' wording. **Fix:** `MESSAGE_TEXT[260] = 'Purchase has completed.'`

## The original's own, kept (checked in the binary 2026-10-05)
- **The game over after pausing and restarting.** `playerDie:` is scheduled 1.3 s ahead (`performSelector:withObject:afterDelay:` 0x3b5d8, delay 0x3ff4cccc_c0000000); neither `StopPlayAction:` nor `gameReplayAction:` sends `cancelPreviousPerformRequestsWithTarget:`. The evaluation's claim that `missionCompletSounding` is never reset was wrong: `missionFailTell:` clears it (0x32788), and the port does too.
- **The test range at minus one heart.** `-[Stage_1_TEST MonsterAttPlayer]` takes a heart per zombie in range with no floor (0x4e6b0..0x4e6c8) and checks HP == 0 once, after the loop (0x4e838).
- **A grab through a pause and a restart.** `StopPlayAction:` touches only `tutorialTimer`, `checkTutorialTimer` and `MotionSamplingTimer`; `gameReplayAction:` resets `isShake` and `noAtt` but not `shakeFlag` or `shakeMonsterTimer`.
- **The top score read over the next row.** `-[Stage_1_E StopElseSpeak]` (0x30018..0x30168) cancels only ReadNumberOfZombies, ReadNumberOfHeadshot, ReadScore and ReadObtainedGold.
- They leave the todo list (they are not bugs to fix here) and go into DIVERGENCES.md as original behaviour the port reproduces, with these addresses.

## Tests
- `tutorial.py`: after the countdown ending, Restart from the panel starts an ordinary run (the walk timer runs, no tutorial beat, `isTutorial` 1), and Escape pauses rather than leaving.
- `volume.py`: "²" and other non-decimal digits count as unusable; ordinary digits still read.
- `store.py` or `menu.py`: the spoken purchase line matches the window's.
- Run only the files covering the changed code ([[project_safe_test_run]]).

## Also
- The changelog gets a line for each fix a player notices (1, 4, 7, 8).
- The todo list: the four fixed bugs go to Finished once the dev confirms them; the four original ones come out.
- Not in scope, noted by the binary check: the comment at `stage_1_e.py` (row 10, about 1689) calling `ReadTopScore` "past the original's bound" may misdescribe the original, which schedules it from `selectTapPointSoundStart` (0x30e48, 0x3102a). Left for a later look.
