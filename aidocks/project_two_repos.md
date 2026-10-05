---
name: project_two_repos
description: "Sixth Sense is two repositories worked on in tandem: SixthSenseOriginal (the faithful port, main only) and SixthSenseReborn (the old custom branch, where fidelity is not the goal). Changes that touch both, such as the save folders, are planned and built in both."
metadata:
  type: project
---

Since 2026-10-04 the game is two repositories, side by side in the dev's `games` folder, and the dev works on them **in tandem**, often in one session:
- **SixthSenseOriginal** (`github.com/tsatria03/SixthSenseOriginal`, formerly SixthSense-Windows): the faithful port, `main` only.
- **SixthSenseReborn** (`github.com/tsatria03/SixthSenseReborn`): the old `custom` branch, moved there with its full history on 2026-10-04 and deleted from SixthSenseOriginal. Fidelity is not the goal; its name for players is Sixth Sense Reborn.

**Why:** The dev said on 2026-10-04 "We are working on 2 repos in tandem", while one change (the save folders) needed code in both.

**How to apply:**
- Each repository keeps its own CLAUDE.md, aidocks and rules; read the one for the repository being changed. Commit and push each separately, with its own messages.
- A change that spans both gets a plan note in each repository, each saying what the other side does.
- The save folders (2026-10-04): SixthSenseOriginal keeps its save in `SixthSenseOriginal` (renaming an old `SixthSense` folder on its first start), and Reborn in `SixthSenseReborn` (copying SixthSenseOriginal's save once, or an old `SixthSense` one).
- **Names (2026-10-04):** SixthSenseOriginal keeps the plain names (`SixthSense.py`, `SixthSense.exe`), so they mark the original port; Reborn's carry its own name (`SixthSenseReborn.py`, `SixthSenseReborn.exe`). The dev: "the first repo should keep the old SixthSense.py so I know that this one is the original port". Only the save folders are named after both repositories.
- Bugs found in shared code are usually in both; the second evaluation's eight went into both todo lists.
