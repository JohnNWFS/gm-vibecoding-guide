# Prompt For GameMaker Vibecoding Projects

Paste this into a new Codex, Claude Code, or ChatGPT coding session before
asking it to modify a GameMaker project.

If the task is to recover an ancient pre-Studio project such as `.gmk`,
`.gm81`, `.gmd`, or `.gb1`, use `PROMPT_FOR_LEGACY_GMK_RECOVERY.md` instead.

```text
You are working on a GameMaker Studio / GML project.

Before editing anything, read the shared GameMaker LLM guide at:
https://raw.githubusercontent.com/JohnNWFS/gm-vibecoding-guide/main/BEST_PRACTICES_FOR_LLM_VIBECODING_FOR_GML.md

Then read these shared guides when verifying or running from the shell:
- https://raw.githubusercontent.com/JohnNWFS/gm-vibecoding-guide/main/GM_CLI_HEADLESS_VERIFY.md
- https://raw.githubusercontent.com/JohnNWFS/gm-vibecoding-guide/main/LLM_VERIFICATION_HARNESS_PATTERN.md

Then inspect this project for any local LLM/project docs, such as:
- README.md
- PROJECT_STATUS.md
- docs/LLM_PROJECT_BRIEF.md
- docs/AUTOTEST_WORKFLOW.md
- docs/*BEST*PRACTICE*.md
- docs/*LLM*.md
- tools/agent-test.ps1 (if present)

Follow these rules:
- Treat GameMaker `.yy` metadata as code.
- Inspect room instanceCreationOrder, object eventList, persistence flags, and project `.yyp` entries when relevant.
- Do not guess through black screens or missing objects. First prove the current room, active instance counts, relevant globals, and which event is running.
- For web builds, verify the actual target used by the project script. Do not assume `html5`, `operagx`, `os_browser`, or `os_type` behavior without checking.
- Prefer one small, testable change over broad rewrites.
- Every string drawn for a person must fit its panel. Measure with string_width and string_height_ext at draw time, after the font is set. A character cap is not a width. The built-in font is proportional and differs by platform. A screenshot that cuts a word off at the panel edge is a failed layout.
- A repeated event that is stored for a person to read needs a bank of distinct human sentences, not one clinical template. Draft the bank with the project's local model, check that every line stays true to the event, and make each line fit the panel. When you add an encounter type or a new logged event, add lines for that type in the same change. The game does not call a model during play.
- When a turn shows an event that is not stored in the permanent record, print one debug row for that turn: the decision, the full on-screen note, and a mark that the note is absent from the biography. A reviewer who only reads the save cannot see that event otherwise.
- Do not revert user changes unless explicitly asked.
- Run verification yourself when the project documents gm-cli or an autotest workflow; do not only tell the user to test.
- After a run, read the project's transcript or snapshot file if one exists. Do not treat full Igor stdout or exit code alone as pass/fail when the Runner waits for keypress.
- New scripts need `.gml`, `.yy`, `.yyp`, and resource_order entries when applicable.
- On Windows PowerShell, use `git commit -m "title" -m "body"` — bash heredocs fail.
- If a new GameMaker/GML lesson is broadly useful across projects, propose an update to the shared guide. Keep project-specific lessons in the local project docs.

For this task, first summarize:
1. What local docs you found.
2. Which GameMaker metadata files are relevant.
3. The single most likely hypothesis.
4. The exact verification you will run.

Only then modify code.
```
