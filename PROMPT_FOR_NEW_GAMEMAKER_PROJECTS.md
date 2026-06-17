# Prompt For GameMaker Vibecoding Projects

Paste this into a new Codex, Claude Code, or ChatGPT coding session before
asking it to modify a GameMaker project.

```text
You are working on a GameMaker Studio / GML project.

Before editing anything, read the shared GameMaker LLM guide:

C:\Users\hoffe\GameMakerProjects\_GM_VibeCoding_Guide\BEST_PRACTICES_FOR_LLM_VIBECODING_FOR_GML.md

Then inspect this project for any local LLM/project docs, such as:
- README.md
- PROJECT_STATUS.md
- docs/LLM_PROJECT_BRIEF.md
- docs/AUTOTEST_WORKFLOW.md
- docs/*BEST*PRACTICE*.md
- docs/*LLM*.md

Follow these rules:
- Treat GameMaker `.yy` metadata as code.
- Inspect room instanceCreationOrder, object eventList, persistence flags, and project `.yyp` entries when relevant.
- Do not guess through black screens or missing objects. First prove the current room, active instance counts, relevant globals, and which event is running.
- For web builds, verify the actual target used by the project script. Do not assume `html5`, `operagx`, `os_browser`, or `os_type` behavior without checking.
- Prefer one small, testable change over broad rewrites.
- Do not revert user changes unless explicitly asked.
- If a new GameMaker/GML lesson is broadly useful across projects, propose an update to the shared guide. Keep project-specific lessons in the local project docs.

For this task, first summarize:
1. What local docs you found.
2. Which GameMaker metadata files are relevant.
3. The single most likely hypothesis.
4. The exact verification you will run.

Only then modify code.
```

